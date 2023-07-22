## Overview

In this article, we're going to discuss the possible strategies for migrating, generating, and backfilling data in a Rails application. We'll implement them, improve them, consider their pros and cons, and discuss which ones are better to use in different scenarios. By the end of the article, we will have a full picture of the different ways to solve data migration problems.

## Introduction

In simple terms, data migration is the process of adding, updating, or transferring some data inside your application. The most popular cases for data migration are as follows:

- *Backfilling column data*
- *Moving column data from one table to another*
- *Generating new database records*
- *Updating corrupted or invalid data with correct values*
- *Removing unused data*

We'll consider 3 different ways to do it:

- **Direct Data Manipulation**
- **Rake Task**
- **Data Migration Gem**

## Direct Data Manipulation

The first option is the simplest one: we'll just add missing data via `rails c` or through a direct database connection in production.

**Advantages:**

- Easy
- No need to implement anything new
- It's fast because data migration can be done in minutes.

**Problems:**

- Too risky; changes may not end up as intended
- Possible access and security problems
- There are no tests and code reviews, so we can't be sure of the quality
- Lack of control; you don't know who ran the migration or why they ran it.

## Rake Task

The second option is the rake task. In this chapter, we will try to understand how to properly add rake tasks, ensure that they work correctly, learn their pros and cons, and explore how they can be used for data migration. We will start by adding the simplest rake task, and then we will proceed to improve its structure, cover the logic with tests, and consider using best practices for writing data migration using rake tasks.

Let's imagine that we have an Animal model with the following fields:

- `id`
- `kind`
- `status`
- `created_at`
- `updated_at`

And we need to change the status value from `nil` to `reserved` for all animals that we created before today. How can we do it? Let's start by adding a simple rake task template.

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    puts 'Updating animal status...'
  end
end
```

And check that it works:

```zsh
rake animals:backfill_statuses
# => Updating animal status...
```

The task has been executed and works as expected. Now, let's add the actual code with the database update. It will look like the following:

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    Animal.where(status: nil).where('created_at < ?', Time.zone.today).update_all(status: 'reserved')
  end
end
```

Now, let's check if it works:

```zsh
rake animals:backfill_statuses
```

The rake task has been executed, and the database values are updated accordingly. That's it. Our main scenario works as expected, but there's still some room for improvements. Let's take a look at what we can do to make our task more reliable.

### Improvements

There are 5 areas that we can potentially improve:

- **Display results in the console for visibility**
- **Ensure data consistency with transactions**
- **Optimize DB requests**
- **Isolate the rake task code**
- **Add tests**

### Display results in the console for visibility

As you may have noticed, the rake task above hasn't shown any output. It can be a real problem because you don't know whether it was run successfully or not, and you will spend much time trying to check it by yourself on production data. Let's fix this problem:

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    puts "Before running the rake task, there were #{Animal.where(status: 'reserved').count} animals in the 'reserved' state."

    Animal.where(status: nil).where('created_at < ?', Time.zone.today).update_all(status: 'reserved')
    
    puts "After running the rake task, there are now #{Animal.where(status: 'reserved').count} animals in the 'reserved' state."
  end
end
```

Now, let's run the rake task:

```zsh
rake animals:backfill_statuses

# => Before running the rake task, there were 0 animals in the 'reserved' state.
# => After running the rake task, there are now 101 animals in the 'reserved' state.
```

With the updated code, we display the number of animals in the 'reserved' state both before and after running the rake task, providing better visibility and ensuring that the task is executed successfully.

### Ensure data consistency with transactions

What happens if some unexpected errors appear in the middle of data migration? Right now, we don't handle it. Even if it's not critical for the example that we've provided, in general, we should not forget to wrap such kind of data manipulation into a transaction to keep a consistent data state.

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    ActiveRecord::Base.transaction do
      Animal.where(status: nil).where('created_at < ?', Time.zone.today).update_all(status: 'reserved')
    end
  end
end
```

By adding the `ActiveRecord::Base.transaction` block, we ensure that all the updates are executed as a single atomic operation. If any error occurs during the data migration, the transaction will be rolled back, and the data will remain unchanged, maintaining data consistency and integrity.

### Optimize DB requests

We have already handled this problem in our rake task, but it's important to mention that we should use the optimal database solution if possible. For example, someone could write our task like this:

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    Animal.where(status: nil).where('created_at < ?', Time.zone.today).each do |animal|
      animal.update(status: 'reserved')
    end
  end
end
```

The following code will trigger an SQL update request for every animal from the list, making it non-optimal:

```zsh
D, [2023-07-21T09:50:58.346040 #67787] DEBUG -- :   Animal Load (1.6ms)  SELECT `animals`.* FROM `animals` WHERE `animals`.`status` IS NULL AND (created_at < '2023-07-21')
D, [2023-07-21T09:50:58.346735 #67787] DEBUG -- :   ↳ lib/tasks/animals/backfill_statuses.rake:10:in `block (2 levels) in <main>'
D, [2023-07-21T09:50:58.371908 #67787] DEBUG -- :   TRANSACTION (2.2ms)  BEGIN
D, [2023-07-21T09:50:58.372737 #67787] DEBUG -- :   ↳ lib/tasks/animals/backfill_statuses.rake:11:in `block (3 levels) in <main>'
D, [2023-07-21T09:50:58.375091 #67787] DEBUG -- :   Animal Update (2.2ms)  UPDATE `animals` SET `animals`.`status` = 'reserved', `animals`.`updated_at` = '2023-07-21 07:50:58.368697' WHERE `animals`.`id` = 1
D, [2023-07-21T09:50:58.375713 #67787] DEBUG -- :   ↳ lib/tasks/animals/backfill_statuses.rake:11:in `block (3 levels) in <main>'
D, [2023-07-21T09:50:58.381169 #67787] DEBUG -- :   TRANSACTION (5.0ms)  COMMIT
D, [2023-07-21T09:50:58.381524 #67787] DEBUG -- :   ↳ lib/tasks/animals/backfill_statuses.rake:11:in `block (3 levels) in <main>'
D, [2023-07-21T09:50:58.383624 #67787] DEBUG -- :   TRANSACTION (1.3ms)  BEGIN
D, [2023-07-21T09:50:58.384250 #67787] DEBUG -- :   ↳ lib/tasks/animals/backfill_statuses.rake:11:in `block (3 levels) in <main>'
D, [2023-07-21T09:50:58.385792 #67787] DEBUG -- :   Animal Update (1.4ms)  UPDATE `animals` SET `animals`.`status` = 'reserved', `animals`.`updated_at` = '2023-07-21 07:50:58.381901' WHERE `animals`.`id` = 2

...
```

That's why it always makes sense to try to find a way to do it in a single DB operation, as we did with the following code:

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    Animal.where(status: nil).where('created_at < ?', Time.zone.today).update_all(status: 'reserved')
  end
end
```

This code will trigger only one DB request:

```zsh
Animal Update All (6.8ms)  UPDATE `animals` SET `animals`.`status` = 'reserved' WHERE `animals`.`status` IS NULL AND (created_at < '2023-07-21')
```

If there's no way to update something in a single DB request, at least you should consider using [batches](https://apidock.com/rails/ActiveRecord/Batches/find_each) as a good practice:

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    Animal.where(status: nil).where('created_at < ?', Time.zone.today).find_each do |animal|
      animal.update(status: 'reserved')
    end
  end
end
```

P.S. To see SQL logs from the rake task, you can add the following code inside:

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

namespace :animals do
  desc "Update animal status to 'reserved' for animals created before today"
  task backfill_statuses: [:environment] do
    ActiveRecord::Base.logger = Logger.new(STDOUT)

    # ...
  end
end
```

### Isolate the rake task code

One not so obvious problem that can occur when you have many rake tasks is a lack of encapsulation. Let's take a look at the following two rake tasks and try to guess what can be wrong here:

- `rake animals:task1`

```rb
# frozen_string_literal: true

# lib/tasks/animals/task1.rake

namespace :animals do
  task task1: [:environment] do
    puts message
  end

  def message
    'Hello world from Task 1!'
  end
end
```

- `rake animals:task2`

```rb
# frozen_string_literal: true

# lib/tasks/animals/task2.rake

namespace :animals do
  task task2: [:environment] do
    puts message
  end

  def message
    'Hello world from Task 2!'
  end
end
```

Now let's run both of them:

```zsh
rake animals:task1
# => Hello world from Task 2!
rake animals:task2
# => Hello world from Task 2!
```

Have you noticed? That's not what we expected! The second rake task overrode the method value from the first one! And that's quite dangerous and unexpected if you were to use something like this:

- `rake animals:task1`

```rb
# frozen_string_literal: true

# lib/tasks/animals/task1.rake

namespace :animals do
  task task1: [:environment] do
    query.destroy_all
  end

  def query
    Animal.where(status: nil)
  end
end
```

- `lib/tasks/animals/task2.rake`

```rb
# frozen_string_literal: true

# lib/tasks/animals/task2.rake

namespace :animals do
  task task2: [:environment] do
    query.destroy_all
  end

  def query
    Animal.all
  end
end
```

And run:

```zsh
rake animals:task1
```

You would remove all your records instead of the desired subset!

How can we fix it?

We need to wrap our rake tasks into the `Rake::DSL` class like this:

- ` rake animals:task1`

```rb
# frozen_string_literal: true

# lib/tasks/animals/task1.rake

module Tasks
  module Animals
    class Task1

      include Rake::DSL

      def initialize
        namespace :animals do
          task task1: [:environment] do
            puts message
          end
        end
      end

      private

      def message
        'Hello world from Task 1!'
      end
    end
  end
end

Tasks::Animals::Task1.new
```

- `rake animals:task2`

```rb
# frozen_string_literal: true

# lib/tasks/animals/task2.rake

module Tasks
  module Animals
    class Task2

      include Rake::DSL

      def initialize
        namespace :animals do
          task task2: [:environment] do
            puts message
          end
        end
      end

      private

      def message
        'Hello world from Task 2!'
      end
    end
  end
end

Tasks::Animals::Task2.new
```

And let's execute:

```zsh
rake animals:task1
# => Hello world from Task 1!
rake animals:task2
# => Hello world from Task 2!
```

Now everything works as expected. Let's apply the same isolation for our `backfill_statuses` rake task.

```rb
# frozen_string_literal: true

# lib/tasks/animals/backfill_statuses.rake

module Tasks
  module Animals
    class BackfillStatuses

      include Rake::DSL

      def initialize
        namespace :animals do
          desc "Update animal status to 'reserved' for animals created before today"
          task backfill_statuses: [:environment] do
            Animal.where(status: nil).where('created_at < ?', Time.zone.today).update_all(status: 'reserved')
          end
        end
      end

    end
  end
end

Tasks::Animals::BackfillStatuses.new
```

That's it.

### Add tests

The last thing that we're going to do to ensure quality is to add tests. Let's see how we can test the rake tasks.

First of all, we need to define some code to load our tasks:

```rb
# spec/support/tasks.rb

# frozen_string_literal: true

RSpec.configure do |_config|
  Rails.application.load_tasks
end
```

And include it in the `spec/rails_helper.rb`:

```rb
# spec/rails_helper.rb

require 'support/tasks'
```

Then let's add our test:

```rb
# spec/tasks/animals/backfill_statuses_spec.rb

# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'rake animals:backfill_statuses', type: :task do
  subject { Rake::Task['animals:backfill_statuses'].execute }

  let(:expected_output) do
    <<~TEXT
      Before running the rake task, there were 1 animals in the 'reserved' state.
      After running the rake task, there are now 2 animals in the 'reserved' state.
    TEXT
  end

  let(:animal_1) { create(:animal, created_at: 10.days.ago, status: nil) }
  let(:animal_2) { create(:animal, created_at: 10.days.ago, status: 'reserved') }
  let(:animal_3) { create(:animal, created_at: 10.days.ago, status: 'another_status') }
  let(:animal_4) { create(:animal, created_at: 10.days.from_now, status: nil) }

  before do
    animal_1
    animal_2
    animal_3
    animal_4
  end

  it "update animal status to 'reserved' for animals created before today" do
    expect { subject }.to change { animal_1.reload.status }.from(nil).to('reserved')
      .and output(expected_output).to_stdout

    expect(animal_2.reload.status).to eq('reserved')
    expect(animal_3.reload.status).to eq('another_status')
    expect(animal_4.reload.status).to be_nil
  end
end
```

That's it.

## Data Migration Gem

The third option is to use the [data-migrate](https://github.com/ilyakatz/data-migrate) gem.

Let's add this gem to our project:

```rb
# Gemfile

gem 'data_migrate'
```

And execute:

```
bundle install
```

Now you can generate a data migration as you would generate a schema migration:

```
rails g data_migration backfill_animal_statuses

# => db/data/20230721111716_backfill_animal_statuses.rb
```

Let's add some code to the generated file to check if it actually works:

```rb
# frozen_string_literal: true

class BackfillAnimalStatuses < ActiveRecord::Migration[7.0]
  def up
    puts 'Test Data Migration'
  end

  def down
    # do nothing
  end
end
```

To run the migration, we need to use the following command:

```
rake data:migrate
# or
rake db:migrate:with_data
```

And we get the following output:

```
== 20230721111716 BackfillAnimalStatuses: migrating ===========================
Test Data Migration
== 20230721111716 BackfillAnimalStatuses: migrated (0.0000s) ==================
```

This migration can be run only once. So let's remove it and generate another one and add real code inside:

```
rails g data_migration backfill_animal_statuses

# => db/data/20230721112534_backfill_animal_statuses.rbb
```

Here's what we get after adding our business logic code:

```rb
# db/data/20230721112534_backfill_animal_statuses.rb

# frozen_string_literal: true

class BackfillAnimalStatuses < ActiveRecord::Migration[7.0]
  def up
    Animal.where(status: nil).where('created_at < ?', Time.zone.today).update_all(status: 'reserved')
  end

  def down
    # do nothing
  end
end
```

And let's run:

```
rake data:migrate
```

Here's what we get:

```
== Data =======================================================================
== 20230721112534 BackfillAnimalStatuses: migrating ===========================
== 20230721112534 BackfillAnimalStatuses: migrated (0.0221s) ==================
```

Basically, data migration works by the same logic as scheme migration, but instead of saving the last running migration version into the `schema_migrations` table, data migration saves the version into another table called `data_migrations`.

It's important to mention that data migration should be irreversible in most cases, but we don't want to raise an explicit error as it would prevent rollback for the scheme structure changes. Instead, we just leave the `down` method empty. For this reason, it would be better to design the migration in an idempotent way, to be able to run it several times if possible.

Data Migration doesn't provide any additional advantages except for what we discussed above. Therefore, we still need to think about the problems that we solved for the rake task, such as displaying output, adding transactions, optimizing DB requests, etc.

## Comparison of Rake Task and Data Migration Gem

Let's compare these two solutions and decide which one we should use and under what conditions:

### When does a rake task fit better?

- When you want to have the ability to select the exact time and day when you want to run it.
- When you want to have the ability to select the platform where you want to run it (for example, staging or production).
- When you want to run the same rake task several times.

### When does the data migration gem fit better?

- When you want to make sure that data will be added automatically and no one forgets to run something.
- When schema migration order is important for you (for example, schema migration adds a new column and data migration backfills this column).
- When you want to run it on all environments without additional effort.
- When you need to run the migration only once.


So, in general, the rake task is much more flexible and testable and can cover the same tasks as the data migration but may require more effort. Data migrations are much more strict but provide some automations and strict execution order that are connected with schema changes.

For example, the data migration gem suits very well if you need to support some database schema restructurization and you don't want to miss the data during these changes. For instance, if you need to rename a column and you want to copy the values from the old one to a new one, then set `NULL=false` constraint to the new one, and then completely remove the old one, the data migration gem will make this process much easier compared to using a rake task.

On the other hand, if the task is unrelated to database schema changes, a rake task might be a more suitable choice. It offers greater flexibility and testability, making it easier to manage tasks that are not directly tied to schema alterations.

## Conclusion

Throughout this article, we have explored various strategies for data migration, generation, and backfilling in a Rails application. We have implemented and improved these strategies, carefully considering their advantages and disadvantages. Additionally, we have compared two primary solutions, the rake task and data migration gem, and examined their suitability in different scenarios. 
