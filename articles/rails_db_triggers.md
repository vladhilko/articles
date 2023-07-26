## Overview

In this article, we are going to discuss the usage of database triggers in Ruby on Rails applications. We will cover what they are, why they are essential, the problems they solve, their pros and cons, when they are allowed to be used, and when they are not. Additionally, we will provide three simple examples of how to add these triggers in a Rails application. By the end of this article, you will have a better understanding and a broader perspective on using DB triggers in Rails applications.

## Introduction

In simple terms, a database trigger is a function that is automatically invoked when an INSERT, UPDATE, or DELETE operation is performed on a table. You can think of it as an ActiveRecord callback that is executed at the database level. Sometimes, they are even referred to as SQL callbacks.

### Use cases

We usually use triggers to solve the following problems:

- Auditing and Logging
- Database schema refactoring (denormalization, renaming columns, splitting a column into two columns, etc.)
- Populating Summary Tables

### Pros/Cons

**Pros:**

- Data Accuracy and Consistency
- Reduced Complexity in Application Code

**Cons:**

- When you use triggers, ActiveRecord has no way of knowing when your records have changed. This means ActiveRecord objects will retain outdated data until they are reloaded.
- Don't overdo it. Delegating too much business logic to triggers can create problems in the future (Debugging Difficulties, Testing Challenges, Performance Impact, etc.)

While triggers can be useful for certain functionalities, it's essential to use them judiciously and consider alternative approaches for implementing business logic within the application code when possible.

## Example 1: Adding a Trigger Inside Rails Console

In this first example, we will create a DB trigger directly in the Rails console and observe how it works. Let's consider two database tables, `animals` and `removed_animals`, each containing only two columns - `id` and `name`. Our goal is to add a trigger that duplicates an animal record into the `removed_animals` table every time we delete a record from the main `animals` table.

Let's see the current state:

```rb
# rails c

Animal.create(name: 'Animal')
Animal.last.delete

RemovedAnimal.all
# => []
```

Currently, the `removed_animals` table is empty and doesn't contain any removed records.

Now, let's execute a manual insertion into `removed_animals` to see what we expect to run in the trigger:

```rb
# rails c

ActiveRecord::Base.connection.execute("INSERT INTO `removed_animals` (`id`, `name`) VALUES (1, 'animal_name')")

RemovedAnimal.all
# =>  [#<RemovedAnimal:0x000000010bc4be98 id: 1, name: "animal_name">]
```

As seen above, we manually inserted a record into `removed_animals`.

Now, let's implement the trigger and check how it works:

```
# rails c

ActiveRecord::Base.connection.execute(
"
CREATE TRIGGER save_removed_animal_trigger
AFTER DELETE ON animals
FOR EACH ROW
INSERT INTO `removed_animals` (`id`, `name`) VALUES (OLD.`id`, OLD.`name`)
"
)
```

With the trigger in place, let's test it further:

```rb
Animal.create(name: 'Animal')
Animal.create(name: 'Another Animal')

Animal.all
# => [#<Animal:0x000000010dc41f18 id: 10, name: "Animal">, #<Animal:0x000000010dc41e50 id: 11, name: "Another Animal">]

RemovedAnimal.all
# => []

Animal.destroy_all

Animal.all
# => []

RemovedAnimal.all
# => [#<RemovedAnimal:0x000000010de6b550 id: 10, name: "Animal">, #<RemovedAnimal:0x000000010de6b488 id: 11, name: "Another Animal">]
```

As shown above, all our destroyed animal records have been saved to the `removed_animals` table.

Now, let's drop this trigger and recreate the same logic using the correct Rails approach in Example 2:

```rb
# rails c

ActiveRecord::Base.connection.execute("DROP TRIGGER save_removed_animal_trigger")
# => nil
```

## Example 2: Adding Triggers Using the hair_trigger Gem

In this second example, we will achieve the same functionality using a clearer approach by utilizing the [hair_trigger](https://github.com/jenseng/hair_trigger) gem. Let's start by installing it:

```rb
# Gemfile

gem 'hairtrigger'
```

And run:

```
bundle install
```

After installing the gem, we can declare triggers in our models and use a rake task to auto-generate the appropriate migration:

```rb
# app/models/animal.rb

class Animal < ApplicationRecord
  trigger.after(:delete) do
    "INSERT INTO `removed_animals` (`id`, `name`) VALUES (OLD.`id`, OLD.`name`)"
  end
end
```

To generate the migration, run the following command:

```zsh
rake db:generate_trigger_migration
```

This task will generate the migration file for us:

```rb
# db/migrate/20230725122552_create_trigger_animals_delete.rb

# This migration was auto-generated via `rake db:generate_trigger_migration'.
# While you can edit this file, any changes you make to the definitions here
# will be undone by the next auto-generated trigger migration.

class CreateTriggerAnimalsDelete < ActiveRecord::Migration[7.0]
  def up
    create_trigger("animals_after_delete_row_tr", :generated => true, :compatibility => 1).
        on("animals").
        after(:delete) do
      "INSERT INTO `removed_animals` (`id`, `name`) VALUES (OLD.`id`, OLD.`name`);"
    end
  end

  def down
    drop_trigger("animals_after_delete_row_tr", "animals", :generated => true)
  end
end
```

Now, execute the migration:

```zsh
rails db:migrate
```

And test if the trigger works as expected, similar to what we did in Example 1:

```rb
# rails c

Animal.create(name: 'Animal')
Animal.create(name: 'Another Animal')

Animal.all
# => [#<Animal:0x000000010bbb1730 id: 12, name: "Animal">, #<Animal:0x000000010bbb15c8 id: 13, name: "Another Animal">]

RemovedAnimal.all
# => []

Animal.destroy_all

Animal.all
# => []

RemovedAnimal.all
# => [#<RemovedAnimal:0x000000010bd81498 id: 12, name: "Animal">, #<RemovedAnimal:0x000000010bd813d0 id: 13, name: "Another Animal">]
```

As you can see, everything works as expected.

## Example 3: Adding a Trigger to Support Column Renaming

In this third example, we will add a trigger that allows us to rename a column with zero downtime. To achieve this, we'll follow these steps:

1. Add a new column.
2. **Add a trigger to dual-write to both columns.**
3. Backfill the new column with a copy of the old column’s values.
4. Start using the new column throughout the whole application.
5. Drop the old column.

For now, we're focused on step 2 - creating a new trigger to write to both columns simultaneously. Let's examine what we want to achieve. Suppose we have an `animals` table with two columns -> `id` and `name`, and we decide to rename the `name` column to `full_name`, for example. So, we need to add this new column and observe the current behavior:

```rb
# rails c

Animal.create(name: 'Animal')
Animal.last

# => #<Animal:0x000000010e823980 id: 14, name: "Animal", full_name: nil>

Animal.last.update(name: 'Updated Animal')
Animal.last

# => #<Animal:0x000000010e9f9d90 id: 14, name: "Updated Animal", full_name: nil>
```

As seen above, when we create or update an animal's name, the `full_name` column remains blank. However, if we replace the `name` column with `full_name`, all values should synchronize. This is where a trigger can be immensely helpful:

```rb
# app/models/animal.rb

class Animal < ApplicationRecord
  trigger.before(:insert) do
    "SET NEW.full_name = NEW.name;"
  end

  trigger.before(:update).of(:name) do
    "SET NEW.full_name = NEW.name;"
  end
end
```

After adding the triggers to the model, we need to run the following command to generate the migration:

```zsh
rake db:generate_trigger_migration
```

This rake task generates the migration file for us:

```rb
# db/migrate/20230725154353_create_triggers_animals_insert_or_animals_update.rb

# This migration was auto-generated via `rake db:generate_trigger_migration'.
# While you can edit this file, any changes you make to the definitions here
# will be undone by the next auto-generated trigger migration.

class CreateTriggersAnimalsInsertOrAnimalsUpdate < ActiveRecord::Migration[7.0]
  def up
    create_trigger("animals_before_insert_row_tr", :generated => true, :compatibility => 1).
        on("animals").
        before(:insert) do
      "SET NEW.full_name = NEW.name;"
    end

    create_trigger("animals_before_update_of_name_row_tr", :generated => true, :compatibility => 1).
        on("animals").
        before(:update).
        of(:name) do
      "SET NEW.full_name = NEW.name;"
    end
  end

  def down
    drop_trigger("animals_before_insert_row_tr", "animals", :generated => true)

    drop_trigger("animals_before_update_of_name_row_tr", "animals", :generated => true)
  end
end

```

Let's run this migration:

```zsh
rails db:migrate
```


And check if it works:

```rb
# rails c

animal = Animal.create(name: 'Animal')
# => #<Animal:0x000000010e5723f8 id: 21, name: "Animal", full_name: nil>
animal.reload
# => #<Animal:0x000000010e5723f8 id: 21, name: "Animal", full_name: "Animal">

animal.update(name: 'New Name')
animal
# => #<Animal:0x000000010e5723f8 id: 21, name: "New Name", full_name: "Animal">
```

As you can see, everything works as expected, and all name values are duplicated into the `full_name` column.


## Conclusion

In this article, we explored three examples of adding triggers to a Rails application. From creating simple triggers in the Rails console to using the `hair_trigger` gem for a clearer approach. Triggers offer data accuracy and automation but should be used judiciously to avoid potential challenges.
