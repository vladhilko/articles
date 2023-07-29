## Overview

In this article, we will discuss the usage of database transactions in a Rails application, with a particular focus on one of the **ACID** principles - **Isolation** - and its four levels. We will explore what these isolation levels are, why they are necessary, and the problems they aim to solve. Additionally, we will provide a straightforward working example of each isolation level using the Rails console. By the end of the article, you will have a decent understanding of isolation levels, and you will be able to verify their behavior on your own to ensure how they work.

## Definition

In simple terms, a transaction is a mechanism that allows you to execute a group of operations in such a way that either they all execute (commit), or the system state will be as if they have not started to execute at all (rollback). In Rails, to use a transaction, you need to write your operations into the following block:

```rb
ActiveRecord::Base.transaction { # ... }
```

## Introduction

To better understand transactions, we will explain what the **ACID** principles are and the kind of problems they aim to solve. Afterward, we will delve deeply into one of these principles called Isolation, and we will explain why we need it and the problem-solving strategies that isolation suggests. Additionally, we will provide Rails examples to better grasp this concept.

## ACID

**ACID** is an acronym that stands for **Atomicity**, **Consistency**, **Isolation**, and **Durability**. These four principles highlight the potential problems that may occur and what we should be aware of. For two of them (**Consistency** and **Durability**), we don't have any control, and we should rely on the database implementation and trust it. For the other two (**Atomicity** and **Isolation**), we can have more control, and we will be focusing on them more.

### Atomicity

The first principle is **Atomicity**. To remember this, we can think about the atom. You can't split the atom. The same goes for an atomicity transaction; you can't just split it. It should be all or nothing. Let's provide an example of breaking this principle:

```rb
# rails c

animal = Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: nil, status: nil>

def check_transaction_atomicity(animal)
  animal.update!(name: 'Cat')
  raise 'Error'
  animal.update!(status: 'created')
end

check_transaction_atomicity(animal)
# RuntimeError: Error

animal.reload
# #<Animal:0x00000001112ad408 id: 1, name: "Cat", status: nil>
```

As you can see, our data is now saved in an inconsistent state. The name was saved, but the status wasn't. This is why we violate the atomicity principle because we must commit all or nothing.

### Consistency

The second principle is **Consistency**. Consistency means that the database must always move from one valid state to another, ensuring that data meets all defined rules and constraints after a transaction is completed. Consistency ensures that any illegal states resulting from the transaction are automatically rolled back to the previous valid state.

If Consistency doesn't exist during the database transaction, then the following code would successfully work even if a strict `NOT NULL` constraint was present on the id column.

```rb
animal = Animal.first

def check_transaction_consistency(animal)
  animal.update(id: nil)
end

check_transaction_consistency(animal)

# => ActiveRecord::NotNullViolation: Mysql2::Error: Column 'id' cannot be null

animal.reload.id
# => 1
```

We can't control this principle; that's why we should fully rely on the database.

### Isolation

The isolation principle doesn't allow interference with any data inside the transaction until the transaction is released. There are three potential problems that may occur and break isolation, all of them are connected with reading data during transaction execution, here they are:

- **Dirty Read**
- **Non-Repeatable Read**
- **Phantom Read**

To remember these problems, you can think about the CRUD operations (create, read, update, and delete):

- **Dirty Read** may appear when we `READ` data
- **Non-Repeatable Read** may appear when we `UPDATE` data
- **Phantom Read**. may appear when we `CREATE` data 

Let's discuss some abstract examples to gain a rough understanding of the problems. In the next chapter, we will provide a real example to illustrate these concepts more concretely.

#### Dirty Read

When a transaction reads data from another uncommitted transaction, we call this violation a **Dirty Read**. This problem has such a name because it allows us to read 'Dirty' data that is unfinished and may not be accurate. An abstract example to demonstrate this violation would look as follows:

```rb
# rails c

Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: "Cat", status: nil>

def dirty_read_transaction
  Animal.first.update!(status: 'dirty_read')
  
  sleep 5
  
  raise 'Unexpected Error'
end

def try_to_read_data_inside_transaction_during_execution
  sleep 1

  puts Animal.first.status
end

Thread.new { dirty_read_transaction }
Thread.new { try_to_read_data_inside_transaction_during_execution }

# => 'dirty_read'
```

As you can see, the problem here is that we have access to the updated data even if the data is not fully committed and may be rolled back later.

#### Non-Repeatable Read

A non-repeatable read occurs when, during the course of a transaction, a row is retrieved twice, and the values within the row differ between reads. This problem has such a name because the same data inside the transaction may not be equal ("not repeated") by the end of the transaction execution. To demonstrate it, we can use the following snippet:

```rb
# rails c

Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: "Cat", status: 'repeatable_read'>

def non_repeatable_read_transaction
  puts "Current value is: #{Animal.first.status}"
  
  sleep 5
  
  puts "Current value is: #{Animal.first.status}"
end

def try_to_update_data_inside_transaction_during_execution
  sleep 1
  
  Animal.last.update(status: 'non_repeatable_read')
end

Thread.new { non_repeatable_read_transaction }
Thread.new { try_to_update_data_inside_transaction_during_execution }

# => 'Current value is: repeatable_read'
# => 'Current value is: non_repeatable_read'
```

As you may notice, the value between the two reads is different ("not repeated").

#### Phantom Read

A phantom read occurs when, in the course of a transaction, new rows are added by another transaction to the records being read. It has such a name because the new record appears in the middle of transaction execution out of nowhere, like a 'phantom', because there were no such records when we just started the transaction execution.

Let's provide an example of breaking this principle:

```rb

Animal.all
# [
#  #<Animal:0x000000010b7ca748 id: 1, name: "Cat", status: nil>,
#  #<Animal:0x000000010b7ca680 id: 2, name: "Dog", status: nil>
# ]

def phantom_read_transaction
  puts "We have the following animals #{Animal.ids}"

  sleep 5

  puts "We have the following animals #{Animal.ids}"
end

def try_to_create_data_inside_transaction_during_execution
  sleep 1
  
  Animal.create(name: 'Wolf')
end

Thread.new { phantom_read_transaction }
Thread.new { try_to_create_data_inside_transaction_during_execution }

# => We have the following animals [1, 2]
# => We have the following animals [1, 2, 3]
```

That's it.

### Durability

The final aspect of the ACID approach to database management is durability.

Durability ensures that changes made to the database (transactions) that are successfully committed will survive permanently, even in the case of system failures. This ensures that the data within the database will not be corrupted by:

- Service outages
- Crashes
- Other cases of failure

Let's take a look at the following example:

```rb
# rails c

animal = Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: nil, status: nil>

def check_transaction_durability(animal)
  animal.update!(name: 'Cat')
end

check_transaction_durability(animal)

# power outage

animal.reload.name
# => 'Cat'
```

If we didn't apply the Durability principle here, then after some unexpected crashes, we would lose the data that was already committed. We can't control this; that's why we should fully rely on the database.

## Transaction Isolation Levels 

To solve the problems that were mentioned above in the **ACID** isolation chapter, the Database provides 4 isolation levels:

- **Read Uncommitted**
- **Read Committed**
- **Repeatable Read**
- **Serializable**

Each of these levels is stronger than the previous one and solves all previous problems by default.

### Read Uncommitted

At this level, we can just rollback the transaction; everyone can update and read data inside during execution. At this level, there's no isolation at all. And as the name suggests, this isolation level is allowed to read the data from the transaction that hasn't been committed. Let's take a look at the example:

```rb
# rails c

Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: "Cat", status: nil>

def read_uncommitted_transaction
  Animal.first.update!(status: 'read_uncommitted')
  
  sleep 5
  
  raise 'Unexpected Error'
end

def try_to_read_data_inside_transaction_during_execution
  sleep 1

  puts "We have access to the uncommitted value inside the transaction, and the status value is: #{Animal.first.status}"
end

Thread.new { ActiveRecord::Base.transaction(isolation: :read_uncommitted) { read_uncommitted_transaction } }
Thread.new { ActiveRecord::Base.transaction(isolation: :read_uncommitted) { try_to_read_data_inside_transaction_during_execution } }

# => We have access to the uncommitted value inside the transaction, and the status value is: read_uncommitted
# => Error (RuntimeError)

Animal.first.status
# => 'nil'
```

As you can see, we had access to the uncommitted status inside the transaction, but after rollback, the final status value is still nil.

### Read Committed

The second isolation level allows us to solve the **Dirty Read** problem but doesn't solve the other two (**Non-Repeatable Read** and **Phantom Read**). As the name of this isolation level suggests, now we can't read uncommitted data, only the committed one. First of all, let's try to run the same example but we'll change the isolation level from `:read_uncommitted` to `:read_committed`:

```rb
# rails c

Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: "Cat", status: nil>

def read_committed_transaction
  Animal.first.update!(status: 'read_committed')
  
  sleep 5
  
  raise 'Unexpected Error'
end

def try_to_read_data_inside_transaction_during_execution
  sleep 1

  puts "We don't have access to uncommitted value inside the transaction, and the status value is still: #{Animal.first.status}"
end

Thread.new { ActiveRecord::Base.transaction(isolation: :read_committed) { read_committed_transaction } }
Thread.new { ActiveRecord::Base.transaction(isolation: :read_committed) { try_to_read_data_inside_transaction_during_execution } }

# => We don't have access to uncommitted value inside the transaction, and the status value is still:
# => Error (RuntimeError)

Animal.first.status
# => 'nil'
```

As you can see here, we don't have access to the value that has been updated inside the other transaction because the second transaction can't read the data from the first one. Do we have any other problems with this isolation level? Yes, as I said, this transaction can solve only the **Dirty Read**  problem, and **Non-Repeatable Read** Read and **Phantom Read** still exist. Let's take a look:

```rb
# rails c

Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: "Cat", status: 'initial'>

def read_committed_transaction
  puts "Current value is: #{Animal.first.status}"
  
  sleep 5
  
  puts "Current value is: #{Animal.first.status}"
end

def try_to_update_data_inside_transaction_during_execution
  sleep 1
  
  Animal.first.update!(status: 'non_repeatable_read')
end

Thread.new { ActiveRecord::Base.transaction(isolation: :read_committed) { read_committed_transaction } }
Thread.new { ActiveRecord::Base.transaction(isolation: :read_committed) { try_to_update_data_inside_transaction_during_execution } }

# => 'Current value is: initial'
# => 'Current value is: non_repeatable_read'
```

As you can see, the second transaction is changing the value inside the first one, and we receive non-repeatable values.

### Repeatable Read

The third isolation level allows us to solve the  **Dirty Read** and  **Non-Repeatable Read** problems, but still can't handle the **Phantom Read**. Let's run the code from the previous example to make sure that **Non Repeatable Read** won't be present here:

```rb
# rails c

Animal.first
# => #<Animal:0x00000001112ad408 id: 1, name: "Cat", status: 'initial'>

def repeatable_read_transaction
  puts "Current value is: #{Animal.first.status}"
  
  sleep 5
  
  puts "Current value is: #{Animal.first.status}"
end

def try_to_update_data_inside_transaction_during_execution
  sleep 1
  
  Animal.first.update!(status: 'non_repeatable_read')
end

Thread.new { ActiveRecord::Base.transaction(isolation: :repeatable_read) { repeatable_read_transaction } }
Thread.new { ActiveRecord::Base.transaction(isolation: :repeatable_read) { try_to_update_data_inside_transaction_during_execution } }

# => 'Current value is: initial'
# => 'Current value is: initial'
```

As you can see, the previous problem disappeared here, and we have "repeatable" values.

Now, let's take a look if we really have the  **Phantom Read** problem here:

```rb
# rails c 

Animal.all
# [
#  #<Animal:0x000000010b7ca748 id: 1, name: "Cat", status: nil>,
#  #<Animal:0x000000010b7ca680 id: 2, name: "Dog", status: nil>
# ]

def repeatable_read_transaction
  puts "Inside the current transaction, we have the following animals: #{Animal.ids}"

  sleep 5

  puts "After the time gap, we still have the following animals: #{Animal.ids}"
  Animal.update_all(status: 'phantom_read_triggered')
  puts "After we triggered the update all operation, we have the following animals: #{Animal.ids}"
end

def try_to_create_data_inside_transaction_during_execution
  sleep 1

  Animal.create(name: 'Wolf')
end

Thread.new { ActiveRecord::Base.transaction(isolation: :repeatable_read) { repeatable_read_transaction } }
Thread.new { ActiveRecord::Base.transaction(isolation: :repeatable_read) { try_to_create_data_inside_transaction_during_execution } }

# => Inside the current transaction, we have the following animals [1, 2]
# => After the time gap, we still have the following animals [1, 2]
# => After we triggered the update all operation, we have the following animals [1, 2, 3]
```

As you can see, a new ID has been added, and it was not expected. Let's try to fix it using the latest isolation level.

### Serializable

The **Serializable** isolation level is the strongest among all the isolation levels. It provides the highest level of isolation and ensures that the transactions are executed in a way that is equivalent to running them sequentially, one after the other. This means that no concurrent execution of transactions can result in anomalies like **Dirty Read**, **Non-Repeatable Read**, or **Phantom Read**.

Let's use the **Serializable** isolation level to fix the **Phantom Read** problem in the previous example:

```rb
# rails c

Animal.all
# [
#  #<Animal:0x000000010b7ca748 id: 1, name: "Cat", status: nil>,
#  #<Animal:0x000000010b7ca680 id: 2, name: "Dog", status: nil>
# ]

def serializable_transaction
  puts "Inside the current transaction, we have the following animals: #{Animal.ids}"

  sleep 5

  puts "After the time gap, we still have the following animals: #{Animal.ids}"
  Animal.update_all(status: 'phantom_read_fixed')
  puts "After we triggered the update all operation, we have the following animals: #{Animal.ids}"
end

def try_to_create_data_inside_transaction_during_execution
  sleep 1

  Animal.create(name: 'Wolf')
end

Thread.new { ActiveRecord::Base.transaction(isolation: :serializable) { serializable_transaction } }
Thread.new { ActiveRecord::Base.transaction(isolation: :serializable) { try_to_create_data_inside_transaction_during_execution } }

# => Inside the current transaction, we have the following animals [1, 2]
# => After the time gap, we still have the following animals [1, 2]
# => After we triggered the update all operation, we have the following animals [1, 2]
```

As you can see there's no phantom records and all data were properly isolated inside the transaction.

## Conclusion

Transaction isolation levels are essential in managing data consistency and concurrency in databases. We discussed four isolation levels: **Read Uncommitted**, **Read Committed**, **Repeatable Read**, and **Serializable**. Each level offers different degrees of isolation and addresses specific problems related to concurrent transactions. We demonstrated how each isolation level affects transactions through practical examples. By carefully choosing the appropriate isolation level, developers can strike a balance between data integrity and performance, ensuring that transactions behave predictably and maintain data consistency even in a multi-user environment.
