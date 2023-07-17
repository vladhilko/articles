## Overview

In this article, we are going to discuss the adapter pattern. We will explore what the adapter pattern is, why we need it, and how it can be implemented in a Rails application. We will also examine the benefits and drawbacks it provides. Additionally, we will provide three different examples to help better understand the concept and the reasons behind it.

## Definition

In simple terms, the Adapter pattern allows you to transform the interface of a class into a different interface that clients expect. By doing so, it enables classes with incompatible interfaces to collaborate and work together seamlessly. We usually use the Adapter pattern for the following purposes:

- **Integrating incompatible components** 

When you need to integrate two components that have incompatible interfaces, Adapter can bridge the gap by providing a common interface that allows them to work together seamlessly.

- **Implementing backward compatibility**

If you need to make changes to an existing class or component but want to maintain compatibility with code that relies on the old interface, Adapter can be used to translate between the old and new interfaces.

- **Working with external libraries, services, or legacy systems**

When integrating with external libraries or services that have their own specific interfaces, Adapter can be used to adapt their interface to fit the interface expected by your application. Adapters are also particularly useful when working with legacy systems or third-party components that have outdated or incompatible interfaces.

The Adapter pattern has the following pros and cons:

### Advantages:

- **Enhanced flexibility**

The Adapter pattern makes it easier to switch between implementations or integrate new components without modifying existing code.

- **Enhanced testability**

The Adapter pattern enhances testability by isolating components, facilitating the use of mocking and dependency injection, and providing a clear interface for the adapted components.

- **Encapsulation of complexity**

The Adapter pattern encapsulates complexity by simplifying complex logic, hiding implementation details, and separating concerns.

- **Independence from external solutions**

The Adapter pattern enables you to switch or replace the external solution without affecting the rest of your codebase. This flexibility allows you to choose the most suitable external solution for your needs without the need for extensive modifications throughout your codebase.

### Problems:

- **Potential increase in code complexity:**

Introducing adapters can add complexity to the codebase, especially when dealing with a large number of adapters or complex adaptation logic.

- **Higher learning curve for developers:**

Developers who are not familiar with the Adapter pattern may require additional time and effort to understand the purpose, design, and usage of adapters in the codebase. This learning curve can slow down development and onboarding processes for new team members.

## Implementation

Let's start by implementing our first adapter. We'll begin with a simple abstract example to help us grasp the concept. Afterward, we'll provide two additional examples and demonstrate how to integrate them into a Rails application.

## Example 1: Cat and Dog

In this example, we will discuss how two different objects with different interfaces can collaborate. Let's imagine that we have the following two classes:

```rb
class Cat
  def self.meow!
    p 'Meow!'
  end
end

class Dog 
  def self.woof!
    p 'Woof!'
  end
end

Cat.meow!
# => 'Meow!'
Dog.woof!
# => 'Woof!'
```

For example, let's say we would like to use one of them based on a certain condition, like the one shown below:

```rb

if Rails.env.test? 
  Cat.meow!
else
  Dog.woof!
end
```

Having many such conditions scattered throughout the code makes it difficult to maintain and less manageable. Therefore, we need to find a solution to address this issue. The main goal of the Adapter Pattern is to solve this problem by creating a common interface without altering the initial implementation. Let's explore how we can achieve it. 

Firstly, we need to create a new Adapter class for each class that we intend to adopt with a new interface.

```rb
class CatAdapter
  def self.speak!
    Cat.meow!
  end
end

class DogAdapter
  def self.speak!
    Dog.woof!
  end
end

CatAdapter.speak!
# => 'Meow!'
DogAdapter.speak!
# => 'Woof!'
```

We have created a common interface for the two objects, which allows us to choose which one to use in a single location, without the need for additional changes throughout the application. For instance, we can implement the following code at the Rails configuration level:

```rb
AnimalAdapter = Rails.env.test? ? CatAdapter : DogAdapter
```

Now we can utilize `AnimalAdapter` throughout the entire application, and this adapter will encapsulate the logic of Cat under the hood

```rb
AnimalAdapter.speak!
# => 'Meow!'
```

## Example 2. Temporary Data Storage

At the second example we are goint to implement a temperory data storage which should do the following:
- Add the given data by key
- Get the date by key
- Remove the data by key

Such temperary data store can usefull when we need to save some intermidate data for example for the following:
- Receive async job stasus

For example if we have a button that send many emails to someone we can block this button to avoid double send untill job will be finished and all emails will be send.

- Save data processing intermediate results

In data-intensive applications, temporary storage is often used to store intermediate results during data processing pipelines. This allows for efficient data transformation, aggregation, or analysis before the final output is generated.

- Anything else that you want to temperaty save 

And we will create 3 diffirent imlementation for every rails enviroments:
- For Test Environment we will create Memory Based Temporary Data Storage
- For Development Enviromnet we will create ActiveRecord Based Temporary Data Storage
- For Production Environment we will create Redis Based Temporary Data Storage

Let's implement each of them to undertand the main reason behind adapter. 


---

In this example, we are going to implement a temporary data storage that should perform the following functions:

- **Add the given data by key**
- **Retrieve the data by key**
- **Remove the data by key**

Such temporary data storage can be useful in various scenarios, including:

- **Tracking the status of asynchronous jobs**

For example, if we have a button that sends multiple emails, we can use temporary data storage to block the button and prevent duplicate sends until the job is finished and all emails are sent.

- **Storing intermediate results during data processing**

In data-intensive applications, temporary storage is often used to store intermediate results during data processing pipelines. This enables efficient data transformation, aggregation, or analysis before generating the final output.

- **Temporary storage for any other purpose**

Temporary data storage can be employed for various other use cases where we need to temporarily store data.


To demonstrate this adapter pattern, we will create three different implementations for each Rails environment:

- For the Test environment, we will create a **Memory-Based Temporary Data Storage**.
- For the Development environment, we will create an **ActiveRecord-Based Temporary Data Storage**.
- For the Production environment, we will create a **Redis-Based Temporary Data Storage**.

Let's proceed with implementing each of them to gain a better understanding of the main purpose behind the adapter pattern.

### Memory-Based Temporary Data Storage

The first solution will be the simplest one - we will create a Memory-Based Temporary Data Storage using a Ruby Hash. We will only use this solution for testing purposes, so we don't need to be concerned about data storage reliability.

First, let's add a new adapter file and define the common interface:

```rb
# app/adapters/temporary_data_store_adapter/memory.rb

# frozen_string_literal: true

module TemporaryDataStoreAdapter
  class Memory

    def set(key, value)
    end

    def get(key)
    end

    def delete(key)
    end

  end
end

```

For our data storage, we have defined three required methods:

- `set`
- `get`
- `delete`

Now, let's implement these methods:

```rb
# app/adapters/temporary_data_store_adapter/memory.rb

# frozen_string_literal: true

module TemporaryDataStoreAdapter
  class Memory

    def initialize
      @store = {}
    end

    def set(key, value)
      @store[key.to_s] = value.to_json
      'OK'
    end

    def get(key)
      return nil unless (value = @store[key.to_s])

      JSON.parse(value)
    end

    def delete(key)
      return nil unless (value = @store[key.to_s])

      @store.delete key.to_s
      JSON.parse(value)
    end

  end
end
```

Let's check how it actually works in the Rails console:

```rb
# rails c

adapter = TemporaryDataStoreAdapter::Memory.new
# =>  #<TemporaryDataStoreAdapter::Memory:0x00000001166b1b80 @store={}>

adapter.set('key', {example: 'example'})
# => "OK"
adapter.get('key')
# => {"example"=>"example"}
adapter.delete('key')
# => {"example"=>"example"}
adapter.get('key')
# => nil
```

That's it!

P.S. It also makes sense to add a clear method to remove all data from the Hash. You can run this command after every test that uses this adapter to avoid any global value problems in the specs:

```rb
def clear
  @store.clear
end
```

### ActiveRecord-Based Temporary Data Storage

Our second implementation of Temporary Data Storage utilizes ActiveRecord. I have mainly added this implementation to enhance our understanding of the Adapter concept, rather than for real-life usage.

Firstly, to use ActiveRecord, we need to create a new model and generate a migration to set up the data storage. Let's create the following model:

```rb
# app/models/temporary_data_entry.rb

# frozen_string_literal: true

class TemporaryDataEntry < ApplicationRecord
end
```

And the corresponding migration:

```rb
# db/migrate/20230714142730_create_temporary_data_entries.rb

class CreateTemporaryDataEntries < ActiveRecord::Migration[7.0]
  def change
    create_table :temporary_data_entries do |t|
      t.string :key, null: false
      t.json :data

      t.timestamps
      t.index :key, unique: true
    end
  end
end
```

Afterward, execute the following command:

```
rails db:migrate
```

Now, everything is set up, and we can proceed to create a new adapter. Let's take a look at how this adapter will be implemented:

```rb
# app/adapters/temporary_data_store_adapter/active_record.rb

# frozen_string_literal: true

module TemporaryDataStoreAdapter
  class ActiveRecord

    def set(key, data)
      temp_data_entry = TemporaryDataEntry.find_by(key: key)

      if temp_data_entry.present?
        temp_data_entry.update(data: data)
      else
        TemporaryDataEntry.create(key: key, data: data)
      end
      'OK'
    end

    def get(key)
      TemporaryDataEntry.find_by(key: key)&.data
    end

    def delete(key)
      TemporaryDataEntry.find_by(key: key)&.delete&.data
    end

  end
end
```

Let's test these methods in the console:

```rb
# rails c


adapter = TemporaryDataStoreAdapter::ActiveRecord.new
# => #<TemporaryDataStoreAdapter::ActiveRecord:0x000000010ec64800>
adapter.set('key', {example: 'example'})
# => "OK"
adapter.get('key')
# => {"example"=>"example"}
adapter.delete('key')
# => {"example"=>"example"}
adapter.get('key')
# => nil
```

That's it.

### Redis-Based Temporary Data Storage

Our final implementation will be based on Redis. First of all, let's install the Redis gem and ensure that we are connected to the server. Add the following gem to your Gemfile:

```rb
# Gemfile 

gem 'redis'
```

Then execute:

```
bundle install
```

Next, start the Redis server and verify if the connection is successful:

```rb
# REDIS_URL='redis://127.0.0.1:6379'
# rails c 

Redis.new(url: ENV.fetch('REDIS_URL')).info
# => {"redis_version"=>"7.0.5" ... }
```

As we can see, the Redis connection is successful. Now, let's proceed with adding a new Redis-based temporary data storage adapter:

```rb
# app/adapters/temporary_data_store_adapter/redis.rb

# frozen_string_literal: true

module TemporaryDataStoreAdapter
  class Redis

    def set(key, value)
      redis.set key, value.to_json
    end

    def get(key)
      return nil unless (value = redis.get(key))

      JSON.parse(value)
    end

    def delete(key)
      return nil unless (value = redis.getdel(key))

      JSON.parse(value)
    end

    private

    def redis
      @redis ||= ::Redis.new(url: ENV.fetch('REDIS_URL'))
    end

  end
end
```

Let's check if it works:

```rb

adapter = TemporaryDataStoreAdapter::Redis.new
# =>  #<TemporaryDataStoreAdapter::Redis:0x0000000112017c60>

adapter.set('key', {example: 'example'})
# => "OK"
adapter.get('key')
# => {"example"=>"example"}
adapter.delete('key')
# => {"example"=>"example"}
adapter.get('key')
# => nil
```

That's it. All adapters have been successfully added. Now, let's take a look at how we can choose the one that is suitable for our needs.

### Initialize Adapter

We're going to use different adapters depending on the Rails environment:

- For the `TEST` environment, we'll use the Memory-Based Temporary Data Storage Adapter.
- For the `DEVELOPMENT` environment, we'll use the ActiveRecord-Based Temporary Data Storage Adapter.
- For the `PRODUCTION` environment, we'll use the Redis-Based Temporary Data Storage Adapter.

To solve this problem, we'll initialize each adapter in their respective environments in the `config/environments` directory:

- **Memory-Based Temporary Data Storage Adapter**

```rb
# config/environments/test.rb

# frozen_string_literal: true

Rails.application.configure do
  # ...
  config.after_initialize do
    config.temporary_data_store_adapter = TemporaryDataStoreAdapter::Memory.new
  end
end
```

- **ActiveRecord-Based Temporary Data Storage Adapter**

```rb
# config/environments/development.rb

# frozen_string_literal: true

Rails.application.configure do
  # ...
  config.after_initialize do
    config.temporary_data_store_adapter = TemporaryDataStoreAdapter::ActiveRecord.new
  end
end
```

- **Redis-Based Temporary Data Storage Adapter**

```rb
# config/environments/production.rb

# frozen_string_literal: true

Rails.application.configure do
  # ...
  config.after_initialize do
    config.temporary_data_store_adapter = TemporaryDataStoreAdapter::Redis.new
  end
end
```

Once initialized, we can call this adapter like this:

```rb
# rails c

Rails.application.config.temporary_data_store_adapter
# => #<TemporaryDataStoreAdapter::ActiveRecord:0x0000000114816ec8>
```

However, this doesn't look too convenient, so let's add a wrapper:

```rb
# app/adapters/adapter.rb

# frozen_string_literal: true

module Adapter
  class << self

    def method_missing(method, *args, &block)
      Rails.application.config.public_send("#{method}_adapter")
    rescue NameError
      super
    end

  end
end
```

Now, we can call the adapter like this:

```rb
# rails c -e development

Adapter.temporary_data_store
# => #<TemporaryDataStoreAdapter::ActiveRecord:0x0000000107634c98>

# rails c -e test

Adapter.temporary_data_store
# => #<TemporaryDataStoreAdapter::Memory:0x00000001101743a8 @store={}>

# rails c -e production

Adapter.temporary_data_store
# => #<TemporaryDataStoreAdapter::Redis:0x0000000112ef9698>
```

That's it. We now have an identical interface, and our previous incompatible solution becomes interchangeable.

## Example 3. Datadog.

In the last example, we will add an Abstract Monitoring Adapter to consolidate what we've already learned. Let's imagine that we have DataDog monitoring, and we want to disable it for the development and test environments. How would we do it? Let's create the following:

```rb
# app/adapters/monitoring_adapter/datadog.rb

# frozen_string_literal: true

module MonitoringAdapter
  class Datadog

    def call
      puts 'Send a real API request to DataDog'
    end

  end
end
```

And the Fake one:

```rb
# frozen_string_literal: true

module MonitoringAdapter
  class Fake

    def call
      puts 'Pretend that the request to DataDog has been sent'
    end

  end
end
```

Let's initialize them:

- **Fake**

```rb
# config/environments/test.rb

# frozen_string_literal: true

Rails.application.configure do
  # ...
  config.after_initialize do
    config.monitoring_adapter = MonitoringAdapter::Fake.new
  end
end
```

- **Datadog**

```rb
# config/environments/production.rb

# frozen_string_literal: true

Rails.application.configure do
  # ...
  config.after_initialize do
    config.monitoring_adapter = MonitoringAdapter::Datadog.new
  end
end
```

And let's check:

```rb
# rails c -e production

Adapter.monitoring.call
# => Send a real API request to DataDog

# rails c -e test

Adapter.monitoring.call
# => Pretend that the request to DataDog has been sent
```

That's it.

## Conclusion

In summary, the adapter pattern proves to be a valuable tool in Rails applications, enabling the integration of components with different interfaces without modifying existing code. By creating adapter classes for each component, we can achieve a common interface and easily switch between implementations. This approach enhances code maintainability, reduces complexity, and improves testability by isolating components and providing clear contracts. With adapters, developers can seamlessly integrate external libraries or services, adapt to legacy systems, and handle compatibility issues efficiently.
