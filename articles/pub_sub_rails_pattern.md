## Overview:
In this article we'll provide a comprehensive guide to understanding and implementing the Pub/Sub pattern. We will explore the evolution of this pattern from a primitive implementation to three product-ready solutions: `ActiveSupport::Notifications`, `Wisper`, and `Dry-Events`. We will begin by discussing the core concept of the Pub/Sub pattern. Then:
  - We will provide a step-by-step guide on how to implement the Pub/Sub pattern using the basic Ruby language constructs.
  - We will introduce the `ActiveSupport::Notifications` library.
  - We will delve into the `Wisper` gem.
  - Finally, we will examine the `Dry-Events` gem.

By the end of this article, we will have a solid understanding of the Pub/Sub pattern and the different options available for implementing it in their Ruby on Rails applications.

## Definition

In simple terms, the Pub/Sub (Publish/Subscribe) pattern is a way of decoupling components in a system by providing an alternative method of communication between them. Rather than sending messages directly to specific components, a publisher broadcasts messages to any subscribers that are interested in receiving them. In other words, the Pub/Sub pattern enables communication between different components of a system without these components having any knowledge of each other.

**Why do we need it and what problems can this pattern solve?** 🤷🏻‍♂️

Let's take a look at the following piece of code from the rails controller.

```rb
def create
  animal = Animal.create(animal_params)
  # send_email
  # send_mobile_notification
  # save_logs

  render json: animal
end
```

**What are we doing here?**

- We want to create an Animal record and show this record in the response
- PLUS send an email about it
- PLUS send a mobile notification about it
- PLUS save this action to the logs

**Do we have any problems with this approach?** 🤔

Yes, we have. This code can lead to several problems, including:

- **Reduced maintainability**: With multiple unrelated tasks in the same controller action, it can be difficult to maintain or modify the code in the future.
- **Reduced testability**: Testing the create action becomes more complicated due to the additional responsibilities it has. It also makes it more challenging to test these other responsibilities in isolation.
- **Reduced scalability**: If additional functionality needs to be added, it would likely be added to the create action, which can make the controller even more complicated and challenging to maintain.
- **Reduced readability**: Mixing unrelated functionality in the same controller action can make it difficult to understand the overall purpose of the action.

## How can we solve this problem? 🤔

Some developers solve this problem by using callbacks on the model to handle it. So, they might use something like this::

```rb
# app/models/animal.rb

# frozen_string_literal: true

class Animal < ApplicationRecord
  after_create :send_email, :send_mobile_notification, :save_logs

  private

  def send_email
    puts 'Email will be sent'
  end

  def send_mobile_notification
    puts 'Mobile Notification will be sent'
  end

  def save_logs
    puts 'Logs will be saved'
  end
end
```

But callbacks can potentially lead to significant issues, so it is better to avoid them.

Potentially, we could create [a separate service object](https://dev.to/vladhilko/how-to-implement-service-object-pattern-in-ruby-on-rails-2mh8) and place all this logic inside it. For example:

```rb
# app/units/create_animal_service.rb

class CreateAnimalService
  def self.call
    # do something
    send_email
    send_mobile_notification
    save_logs
  end
  
  private
  
  def send_email
    puts 'Email will be sent'
  end

  def send_mobile_notification
    puts 'Mobile Notification will be sent'
  end

  def save_logs
    puts 'Logs will be saved'
  end
end
```

While this approach is an improvement over using model callbacks, there is still a significant amount of cohesion between the core business logic and non-critical secondary logic. As a result, developers must understand the secondary logic, which is closely tied to the core logic, leading to increased cognitive load and a shift in focus from the primary part to technical details.

To address these issues, we can use the Pub/Sub pattern to separate the core business logic from non-critical secondary logic. Let's discuss this further:

## How pub/sub actually works?

There are three key elements that must be implemented to make Pub/Sub work for you:

- Send/Publish/Broadcast an event.
- Define the logic that can potentially react to this event.
- Bind the event with the logic ("subscribe to the event").

And we would like to have something like this as our new interface:

```rb
def create
  animal = Animal.create(animal_params).tap { send_event('animal_created') }

  render json: animal
end
```

This interface would allow us to concentrate on the core business logic and delegate all 'secondary' logic to somewhere else.

With the key elements above in mind, let's try to build a primitive Pub/Sub logic using plain Ruby to understand the concept behind it.

---

## Primitive Ruby implementation

In this implementation, we'll use the same three main elements: 'Publishing an event', 'Defining logic' and 'Binding the event to the logic'.

### Publishing an Event

For example, let's consider an event named `animal_created`, which is represented as a string. Whenever this event is sent, we simply add this string to an array

```rb
events = []
events << 'animal_created'
```

### Defining logic to react to the event (subcriptions)

To react to the event, we need to define some actions that can potentially happen when the event occurs. These actions are called `subscriptions` or `listeners`. We can use a simple hash with two keys: `event_name` and `action`. `event_name` is a string that identifies the event, and `action` is a proc that is executed when the event is received.

```rb
subscriptions = []
subscriptions << { event_name: 'animal_created', action: -> { puts 'hello animal!' } }
subscriptions << { event_name: 'animal_created', action: -> { puts 'hello animal 2!' } }
```

### Binding events and subscriptions

We have defined two main elements, but there is a problem - events and subscriptions are not connected and do not know about each other. To address this, we need to introduce some logic that will iterate over new events and match them with the appropriate action defined in the subscriptions.

```rb
events.each do |event_name|
  subscriptions.select { |subscription| subscription[:event_name] == event_name }.each { |subscription| subscription[:action].call }
end
events.clear
```

So, that is the basic idea. Let's try to improve our implementation by encapsulating some logic in a class.

---

## Primitive Ruby implementation using class

We will create a class called `Notification`, which will have 3 methods: `publish`, `subscribe` and `initialize`. In this implementation, we will immediately react to the published event as soon as it is sent.

```rb
class Notification
  def initialize
    # We are initializing a Hash with empty arrays as default values for any new key
    @notifications = Hash.new { |hash, key| hash[key] = [] }
  end

  def subscribe(event_name, action:)
    notifications[event_name] << action
  end

  def publish(event_name)
    notifications[event_name].each { |action| action.call }
  end

  private

  attr_reader :notifications
end
```

Here is a usage example:


```rb
# Initialize the class
notification = Notification.new

# Subscribe to the event "animal_created" with two actions.
notification.subscribe('animal_created', action: -> { puts 'hello animal!' })
notification.subscribe('animal_created', action: -> { puts 'hello animal 2!' })

# Publish the 'animal_created' event, which will trigger both actions.
notification.publish('animal_created')
```

Out three key elements have also been used in this implementation.

---

This is a very simple example of how pub/sub can work. However, there are many ready-made solutions that we can use in our application. Let's consider three of them:

- `ActiveSupport::Notification`
- `Wisper`
- `Dry-events`

They're pretty much the same and have very similar interfaces. Let's look at each of these options one by one to better understand the concept.

---

## ActiveSupport::Notification

[ActiveSupport::Notification ](https://api.rubyonrails.org/classes/ActiveSupport/Notifications.html "ActiveSupport::Notification") has the same three key elements that we discussed earlier

- Send/Publish/Broadcast an event.
- Define the logic that can potentially react to this event.
- Bind the event with the logic ("subscribe to the event").

Let's consider three ways of implementing the pub/sub pattern using `ActiveSupport::Notification`, starting from the simplest and progressing to the most complex, in order to better understand the concept:

- `ActiveSupport::Notification` with blocks
- `ActiveSupport::Notification` with classes
- `ActiveSupport::Notification` in a Rails application

Let's start with the most basic one 

### `ActiveSupport::Notification` with blocks

```rb
# Define the logic
ActiveSupport::Notifications.subscribe('animal_created') do |*args|
  puts "New animal created 1"
end

ActiveSupport::Notifications.subscribe('animal_created') do |*args|
  puts "New animal created 2"
end

# Send the event
ActiveSupport::Notifications.instrument('animal_created')

# => New animal created 1
# => New animal created 2
```

So, here everything looks obvious. Basically, we have the same interface as we described in the primitive Ruby implementation. Let's go further.

### `ActiveSupport::Notification` with classes

Instead of a block, we can also send a class that has a `call` method. This can be useful when your logic becomes too complex, and using classes can greatly simplify testing.

```rb
# Define the logic
class Subscription1
  def call(name, started, finished, unique_id, payload)
    puts "New animal created 1"
  end
end

class Subscription2
  def call(name, started, finished, unique_id, payload)
    puts "New animal created 2"
  end
end

# Bind the event with the logic
ActiveSupport::Notifications.subscribe('animal_created', Subscription1.new)
ActiveSupport::Notifications.subscribe('animal_created', Subscription2.new)

# Send the event
ActiveSupport::Notifications.instrument('animal_created')

# => New animal created 1
# => New animal created 2
```
Here's how `ActiveSupport::Notification` basically works. Let's try to add it to a real Rails application.

### `ActiveSupport::Notification` in a Rails application

For example, we have a service object that performs some actions and sends the `'animal_created'` event. 

#### Publishing an Event

```rb
# app/units/service.rb

class Service
  def self.call
    # perform some actions
    ActiveSupport::Notifications.instrument('animal_created', payload: {})
  end
end
```

#### Defining logic to react to the event

Additionally, we need to create subscriptions to react to these events. We want to send an email, send a mobile notification, and log this action somewhere. To accomplish this, we'll create three subscription classes, each of which will handle one of these tasks.

- `EmailSubscription`

```rb
# app/subscriptions/email_subscription.rb

# frozen_string_literal: true

class EmailSubscription < Subscription

  def animal_created(payload)
    puts "Email will be sent #{payload}"
  end

end
```
- `MobileNotificationSubscription`

```rb
# app/subscriptions/mobile_notification_subscription.rb

# frozen_string_literal: true

class MobileNotificationSubscription < Subscription

  def animal_created(payload)
    puts "Mobile Notification will be sent #{payload}"
  end

end
```

- `LoggerSubscription`

```rb
# app/subscriptions/logger_subscription.rb

# frozen_string_literal: true

class LoggerSubscription < Subscription

  def animal_created(payload)
    puts "Logs will be saved #{payload}"
  end

end
```

Every subscription class should include the desired reaction to the event and a method call to actually run this reaction. That's why we've created a parent class `Subscription`.

-  `Subscription`

```rb
# lib/subscription.rb

# frozen_string_literal: true

class Subscription

  def call(event_name, _, _, _, payload)
    send(event_name, payload)
  end

end

```

#### Binding events and logic

We've created an event that can be sent and classes with the appropriate reaction to the event. However, these classes aren't currently connected to the event, meaning they won't react to it. To establish the connection, we'll use the following logic after Rails initialization.

```rb
# config/initializers/subscriptions.rb

# frozen_string_literal: true

Rails.application.config.after_initialize do
  subscriptions = {
    EmailSubscription: ['animal_created'],
    LoggerSubscription: ['animal_created'],
    MobileNotificationSubscription: ['animal_created']
  }

  # We iterate over each subscription class and bind its logic with the event name, similar to what we did in the example with blocks.
  subscriptions.each do |subscription_class, events|
    events.each do |event|
      ActiveSupport::Notifications.subscribe(event, subscription_class.to_s.constantize.new)
    end
  end
end
```

So now we can call `Service.call` and this code will send the 'animal_created' event, triggering all subscriptions:

```
Email will be sent {:payload=>{}}
Logs will be saved {:payload=>{}}
Mobile Notification will be sent {:payload=>{}}
```

#### Adding a new event

To add a new event, you need to update the subscription classes with the desired logic and then bind them to the new event. This ensures that the subscription logic will be executed whenever the event is triggered.

For example, if you would like to add the `car_created` event to the EmailSubscription, then you should do the following:

```rb
# app/subscriptions/email_subscription.rb

# frozen_string_literal: true

class EmailSubscription < Subscription

  def animal_created(payload)
    puts "Email will be sent #{payload}"
  end

  def car_created(payload)
    puts "Car email will be sent #{payload}"
  end

end
```

It's important not to forget to subscribe to this event in the initializer after adding the logic to the `EmailSubscription` class for the `car_created` event.

```rb
# config/initializers/subscriptions.rb

# frozen_string_literal: true

Rails.application.config.after_initialize do
  subscriptions = {
    EmailSubscription: ['animal_created', 'car_created'],
    LoggerSubscription: ['animal_created'],
    MobileNotificationSubscription: ['animal_created']
  }
  # ...
end
```

When we call `car_created`, the subscribed logic in `EmailSubscription` will be triggered as expected.

```rb
ActiveSupport::Notifications.instrument('car_created', payload: {})

# => Car email will be sent {:payload=>{}}
```

---

## Wisper gem

The second option is to use [`Wisper`](https://github.com/krisleech/wisper) gem.

First of all, we need to add the `wisper` gem to the `Gemfile` and then run `bundle install`:

```rb
gem 'wisper'
```

This gem has the same three key elements:

- Send/Publish/Broadcast an event.
- Define the logic that can potentially react to this event.
- Bind the event with the logic ("subscribe to the event").

### Send an event.

To send an event we need to do the following:

```rb
# app/units/create_animal_service.rb

# frozen_string_literal: true

class CreateAnimalService
  include Wisper::Publisher

  def call
    # do something
    broadcast(:animal_created, payload: {})
  end
end

CreateAnimalService.new.call
```

### Define the logic

To add logic for our events, we need to do the following:

- `EmailSubscription`

```rb
# app/subscriptions/email_subscription.rb

# frozen_string_literal: true

class EmailSubscription

  def animal_created(payload)
    puts "Email will be sent #{payload}"
  end

end
```

- `LoggerSubscription`

```rb
# app/subscriptions/logger_subscription.rb

# frozen_string_literal: true

class LoggerSubscription

  def animal_created(payload)
    puts "Logs will be saved #{payload}"
  end

end
```

- `MobileNotificationSubscription`

```rb
# app/subscriptions/mobile_notification_subscription.rb

# frozen_string_literal: true

class MobileNotificationSubscription

  def animal_created(payload)
    puts "Mobile Notification will be sent #{payload}"
  end

end
```

### Bind the event with the logic

To bind the event and subscription, we need to do the following

```rb
# config/initializers/subscriptions.rb

# frozen_string_literal: true

Rails.application.config.after_initialize do
  Wisper.subscribe(EmailSubscription.new)
  Wisper.subscribe(LoggerSubscription.new)
  Wisper.subscribe(MobileNotificationSubscription.new)
end

```

As you can see, the interface is quite similar to what we had in `ActiveSupport::Notifications`.

---


## Dry-event

Our third option is to create a Pub/Sub system using the [`dry-rb`](https://dry-rb.org/gems/) gems. To be more precise, we are going to use the [`dry-events`](https://github.com/dry-rb/dry-events) gem. 

To get started, we need to add the `dry-events` gem to our `Gemfile` and run `bundle install`:

```rb
gem 'dry-events'
```

This gem has the same 3 key elements plus an additional one - we need to register an event. Let's take a look at the example below:

### Register an event 

```rb
# lib/events.rb

# frozen_string_literal: true

require 'dry/events/publisher'

class Events
  include Singleton
  include Dry::Events::Publisher[:my_publisher]

  register_event('animal.created')
end
```

We included `Singleton` here because `dry-event` requires us to work with an instance, but we don't want to create a new instance every time.

### Send an event

```rb
# app/units/create_animal_service.rb

# frozen_string_literal: true

class CreateAnimalService
  def self.call
    # do something
    Events.instance.publish('animal.created', payload: {})
  end
end
```

### Defining logic to react to the event (Subscriptions)

- `EmailSubscription`

```rb
# app/subscriptions/email_subscription.rb

# frozen_string_literal: true

class EmailSubscription

  def on_animal_created(event)
    puts "Email will be sent #{event[:payload]}"
  end

end

```

- `LoggerSubscription`

```rb
# app/subscriptions/logger_subscription.rb

# frozen_string_literal: true

class LoggerSubscription

  def on_animal_created(event)
    puts "Logs will be saved #{event[:payload]}"
  end

end

```

- `MobileNotificationSubscription`

```rb
# app/subscriptions/mobile_notification_subscription.rb

# frozen_string_literal: true

class MobileNotificationSubscription

  def on_animal_created(event)
    puts "Mobile Notification will be sent #{event[:payload]}"
  end

end

```

## Bind the event with the logic

```rb
# config/initializers/subscriptions.rb

# frozen_string_literal: true

Rails.application.config.after_initialize do
  events = Events.instance

  events.subscribe(EmailSubscription.new)
  events.subscribe(LoggerSubscription.new)
  events.subscribe(MobileNotificationSubscription.new)
end

```

---


## Conclusion

**The final solution may look like this or be placed under the Service object which sends an event inside:**

```rb
def create
  animal = Animal.create(animal_params).tap { ActiveSupport::Notifications.instrument('animal_created') }

  render json: animal
end
```
In conclusion, Pub/Sub pattern is a useful design pattern in Ruby on Rails applications for managing communication between different parts of the system. The advantages of using Pub/Sub are numerous, including the most important ones:

### Advantages
- **Decoupling**
> One of the main advantages of the pub/sub pattern is that it allows publishers and subscribers to be decoupled from each other. Publishers do not need to know about the subscribers, and subscribers do not need to know about the publishers. This makes it easy to add or remove components from the system without affecting the other components.

- **Scalability**
> The pub/sub pattern is highly scalable, since it allows multiple subscribers to receive the same message at the same time. This makes it easy to distribute messages to a large number of subscribers, without overloading the publishers or the subscribers.

- **Flexibility**
> The pub/sub pattern provides a flexible way to implement communication between different components of an application, as well as between different applications. Since publishers and subscribers do not need to know about each other, it is easy to add new features or change existing ones without having to modify existing code.

- **Asynchronous**
> The pub/sub pattern is asynchronous, which means that publishers and subscribers do not need to be active at the same time. Publishers can publish messages at any time, and subscribers can consume messages whenever they are ready. This makes it easy to implement real-time notifications and messaging systems.

### Disadvatages

- **Complexity**
> Implementing the pub/sub pattern can be complex, especially when dealing with a large number of publishers and subscribers. This pattern can add an additional layer of complexity to the system, which can make it harder to debug and maintain.

- **Reduction of application flow visibility**
> Another potential disadvantage of using the Pub/Sub pattern is that it can result in a reduction of application flow visibility. This means that the overall flow of the system may be harder to understand, as the primary action and its secondary effects may be implemented in different parts of the codebase, making it difficult for future readers to comprehend the system's operation.

