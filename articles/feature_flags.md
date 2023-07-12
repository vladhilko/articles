## Overview

In this article, we are going to delve into the area of Feature Flags. We will explain what a feature flag is, why we need it, and how we can use it. We will discuss how to create feature flags and explore the advantages and disadvantages they offer. Additionally, we will explore different feature release strategies that can be safely employed in your Rails application. Finally, we will consider various improvements that can be applied to enhance the reliability of feature flags.

## Definition

In simple terms, a feature flag is a technique used in software development to enable or disable specific features at runtime. Feature flags act as switches that control the availability and visibility of certain features within an application without the need for redeploying or modifying the code. Using feature flags has its pros and cons, let's consider them:


### Advantages:
- **Risk Mitigation**
> Easily disable features in case of unexpected problems or errors.
- **A/B Testing and Experimentation**
> Conduct experiments and compare different feature variations to make informed decisions.
- **Controlled Rollouts**
> Allows testing the feature on different platforms (staging/sandbox) before enabling it in production.
- **Speed of merging**
> Developers can work on small pull requests that are merged frequently into the main code branch.

### Problems:
- **Increased Complexity**
> Increases code loading and cognitive load due to the need for additional conditional checks in the code.
- **Technical Debt**
> If feature flags are not appropriately managed and cleaned up, they can become deprecated or unnecessary, resulting in increased code complexity and maintenance overhead.
- **Testing Overhead**
> The presence of multiple feature variations requires thorough testing of each flag's behavior and interactions, potentially increasing the testing effort and complexity.

It's important to remember that a feature flag is a temporary solution that should be removed after the feature is fully implemented, tested, delivered, and approved by clients.

## Implementation

We're going to use the [flipper](https://github.com/jnunemaker/flipper) gem. Let's take a look at how to use it properly.

### Installation 

First of all, we need to install this gem.

```rb
# Gemfile

gem 'flipper'
```

And then execute:

```zsh
bundle
```

The gem provides interfaces for the following most important features:

- **Enabling/Disabling the feature**
- **Checking if the feature is enabled or not**
- **Showing the list of all features**

```rb
# rails c 

Flipper.features
# #<Set: {}>
Flipper.enable :search
# #<Set: {#<Flipper::Feature:72040 name=:search, state=:on, enabled_gate_names=[:boolean], adapter=:memoizable>}>
Flipper.enabled? :search
# true
Flipper.disable :search
# #<Set: {#<Flipper::Feature:72040 name=:search, state=:off, enabled_gate_names=[], adapter=:memoizable>}>
Flipper.enabled? :search
# false
```
The problem with the current approach is that the data stored in memory will be lost after reloading the Rails console. Therefore, we need to determine where exactly we want to store this data. Flipper provides several available options for storing the value, called [adapters](https://www.flippercloud.io/docs/adapters). In this case, we're going to use [flipper-active_record](https://www.flippercloud.io/docs/adapters/active-record) as it is the most familiar one.

### Add ActiveRecord adapter

To use the ActiveRecord adapter, we need to install the gem:

```rb
# Gemfile

gem 'flipper-active_record'
```

And then execute:

```zsh
bundle
```

After the gem is installed, we need to generate a migration where our data will be stored:

```
rails g flipper:active_record
```

And then execute:

```zsh
rails db:migrate
```

Let's see how it works in the Rails console:

```rb
# rails c 

Flipper.features
# #<Set: {}>
Flipper.enable :search
# This command creates the following records in the database:

# [
#   #<Flipper::Adapters::ActiveRecord::Feature:0x000000011672da78
#   id: 1,
#   key: "search",
#   created_at: Sun, 09 Jul 2023 13:08:14.499774000 UTC +00:00,
#   updated_at: Sun, 09 Jul 2023 13:08:14.499774000 UTC +00:00>
# ]
# [
#   #<Flipper::Adapters::ActiveRecord::Gate:0x00000001166ddbb8
#   id: 1,
#   feature_key: "search",
#   key: "boolean",
#   value: "true",
#   created_at: Sun, 09 Jul 2023 13:08:14.527445000 UTC +00:00,
#   updated_at: Sun, 09 Jul 2023 13:08:14.527445000 UTC +00:00>
# ]

Flipper.enabled? :search
# true
```

That's it.

## Feature Release Strategies

Feature release strategies refer to the approaches and techniques used to release new features or updates in software development.

Here are some common feature release strategies:

- **Big Bang Release**
- **Incremental Release**
- **Feature Flags/Toggles**
- **Canary Release**
- **Phased Rollout**
- **Fault tolerance**

We will discuss these strategies using a simple abstract example of changing the background color of our website to red. Let's take a closer look at each of them and provide examples of their implementation in Ruby.

### Big Bang Release

The Big Bang Release strategy states that all changes need to be deployed at once as a single, comprehensive update. This strategy does not require the use of feature flags. For example, if you had the following code in your application:

```rb
# change_background_color.rb

ChangeBackgroundColor.call('blue')
```

The Big Bang Strategy will replace this code all at once wherever it is used, in a single move

```rb
# change_background_color.rb

ChangeBackgroundColor.call('red')
```

### Incremental Release

The Incremental Release strategy states that you should deliver your code piece by piece to catch possible errors as soon as possible. For example, if we have the following code:

```rb
# change_background_color_1.rb

ChangeBackgroundColor1.call('blue')
```

and 

```rb
# change_background_color_2.rb

ChangeBackgroundColor2.call('blue')
```

The Incremental Release Strategy suggests updating and deploying the first file first, and only after that, update and deploy the second one. For example, in the first deployment, we update the first file as follows:

```rb
# change_background_color_1.rb

ChangeBackgroundColor1.call('red')
```

Then, in the second deployment, we update the second file as follows:

```rb
# change_background_color_2.rb

ChangeBackgroundColor2.call('red')
```

So, when we have multiple changes, they should be split and gradually updated and deployed.

### Feature Flags/Toggles

This is the first strategy where Feature Flags fit perfectly. In this strategy, we choose the behavior depending on whether the feature flag is enabled or not:

```rb
# change_background_color.rb

if Flipper.enabled? :red_background_color
  ChangeBackgroundColor.call('red')
else
  ChangeBackgroundColor.call('blue')
end
```

It allows us to disable the feature if something goes wrong.

### Canary Release

The Canary Release Strategy allows us to enable a feature for specific percentages of users. This enables us to perform slow rollouts. For example, if we want to enable the feature only for one user, we can do the following:

```rb
user = User.first

Flipper.enabled?(:red_background_color)
# => false
Flipper.enabled?(:red_background_color, user)
# => false

Flipper.enable(:red_background_color, user)

Flipper.enabled?(:red_background_color)
# => false

Flipper.enabled?(:red_background_color, user)
# => true
```

But if we want to enable the `:red_background_color` feature for only 25% of users, we need to do the following:

```rb
Flipper.enable_percentage_of_actors(:red_background_color, 25)

user = User.first

Flipper.enabled?(:red_background_color, user)

# => true/false
```

Please note that you can read more about how the algorithm actually works [here](https://www.hackwithpassion.com/this-is-how-percentages-work-in-flipper/).

### Phased Rollout

The Phased Rollout Strategy allows us to enable a feature for specific groups, such as by country, role, or status etc. To enable this feature, we need to define the criteria or conditions for determining when the enabling should occur:

```rb
user = User.first 
# => <User id: 1, role: "admin">

Flipper.register(:admins) do |actor, context|
  actor.respond_to?(:role) && actor.role == 'admin'
end

Flipper.enable_group(:red_background_color, :admins)

Flipper.enabled?(:red_background_color, user)
# true
```

Let's consider another example where we want to enable the feature for all users from the USA. It would look like this:

```rb
user = User.first 
# => <User id: 1, country: "USA">

Flipper.register(:from_usa) do |actor, context|
  actor.respond_to?(:country) && actor.country == 'USA'
end

Flipper.enable_group(:red_background_color, :from_usa)

Flipper.enabled?(:red_background_color, user)
# => true
```

However, we have one problem. Where should we store `Flipper.register(:admins)`? Since we don't save the value of the block in the database, this block should be placed somewhere at the configuration level. Let's add it to the initializer:

```rb
# config/initializers/flipper.rb

# frozen_string_literal: true

require 'flipper'

Rails.application.reloader.to_prepare do
  Flipper.register(:admins) do |actor, context|
    actor.respond_to?(:role) && actor.role == 'admin'
  end
end
```

That's it.

### Fault Tolerance

In this strategy, we run the new version of the code, but if something unexpected happens, we fallback to the previous version. It can be implemented as follows:

```rb
begin
  ChangeBackgroundColor.call('red')
rescue UnexpectedError
  ChangeBackgroundColor.call('blue')
end
```

## Improvements

Our solution is not perfect; there is still some room for improvement. Let's explore how we can enhance the following aspects:

- **Removing the feature flag**
- **Optimizing requests using caching**
- **Providing a user interface (UI)**
- **Adding an API**
- **Adding validations for Feature Flags**
- **Including information about the Feature Flags**
- **Improving Spec Performance**

### Removing the feature flag

It's important to remember that every feature flag is a temporary solution for safe rollouts, and sooner or later they should be removed from the codebase. How can we do it? To remove a Feature Flag, we need to follow these steps:

```rb
Flipper.remove(:red_background_color)
# => true
```

### Optimizing requests using caching

To optimize the retrieval of feature flag information and avoid unnecessary database calls, Flipper provides the option to enable caching. To implement caching, you need to install the `flipper-active_support_cache_store` gem by adding it to your Gemfile:

```rb
# Gemfile

gem 'flipper-active_support_cache_store'

```

After adding the gem, you need to update the configuration file as follows:

```rb
# config/initializers/flipper.rb

# frozen_string_literal: true

require 'flipper/adapters/active_record'
require 'flipper/adapters/active_support_cache_store'

Rails.application.reloader.to_prepare do
  Flipper.configure do |config|
    config.adapter do
      Flipper::Adapters::ActiveSupportCacheStore.new(
        Flipper::Adapters::ActiveRecord.new,
        ActiveSupport::Cache::MemoryStore.new,
        expires_in: 5.minutes
      )
    end
  end
end
```

With this configuration in place, you can now check if a feature is enabled without triggering a database call for the next 5 minutes:

```rb
# rails console

Flipper.enabled?(:red_background_color)
# triggers a database call

Flipper.enabled?(:red_background_color)
# does not trigger a database call
```

### Providing a user interface (UI)

To add a user interface (UI) for managing feature flags, you can follow these steps:

1. Add the `flipper-ui` gem to your application's Gemfile:

```rb
# Gemfile

gem 'flipper-ui'
```

2. Execute the bundle command to install the gem:

```
bundle
```

3. Update your config/routes.rb file to mount the Flipper UI:


```rb
# config/routes.rb

YourRailsApp::Application.routes.draw do
  mount Flipper::UI.app(Flipper) => '/flipper'
end
```

After updating the routes, the new UI will be available at http://localhost:3000/flipper (assuming your application is running on localhost and port 3000). You can find more information about using the Flipper UI [here](https://www.flippercloud.io/docs/ui).


### Adding an API

What if you want to expose the list of feature flags as an API? This can be useful when your client and API are separated, making it easier to retrieve the data. To add the Flipper API to your project, follow these steps:

First, install the `flipper-api` gem by adding it to your Gemfile:

```rb
# Gemfile

gem 'flipper-api'
```

Next, execute the `bundle` command to install the gem:

```
bundle 
```

Update your `config/routes.rb` file to include the Flipper API:

```rb
# config/routes.rb

YourRailsApp::Application.routes.draw do
  mount Flipper::Api.app(Flipper) => '/flipper/api'
end

```

With this configuration, you can make a GET request to http://localhost:3000/flipper/api/features to retrieve the list of feature flags. The response will be in JSON format, as shown below:

```json
{
  "features":[
    {
      "key":"red_background_color",
      "state":"on",
      "gates":[
        {
          "key":"boolean",
          "value":"true",
          "name":"boolean"
        },
        {
          "key":"actors",
          "value":[
            
          ],
          "name":"actor"
        },
        {
          "key":"percentage_of_actors",
          "value":null,
          "name":"percentage_of_actors"
        },
        {
          "key":"percentage_of_time",
          "value":null,
          "name":"percentage_of_time"
        },
        {
          "key":"groups",
          "value":[
            
          ],
          "name":"group"
        }
      ]
    }
  ]
}
```

For more details on adding an API, refer to the Flipper documentation [here](https://www.flippercloud.io/docs/api)

### Adding validations for Feature Flags

As you may have noticed, our current implementation allows adding flags with any names and removing flags without restrictions. However, this approach is not reliable, as someone could make a typo and mistakenly enable/disable the wrong flag or accidentally remove a flag that is actively being used in production. Let's explore a possible solution to prevent such issues.

First, we can create a value object to keep track of all available feature flags. This will help us ensure that only valid flags are used. Let's create the FeatureFlag value object:

```rb
# app/value_objects/feature_flag.rb

# frozen_string_literal: true

class FeatureFlag

  class << self

    def all
      [
        'search',
        'red_background_color'
      ]
    end

    def supported?(flag_name)
      all.include?(flag_name.to_s)
    end

  end

end
```

This value object provides a list of all available feature flags and allows us to check if a specific flag is supported:

```rb
FeatureFlag.all
# => ['search', 'red_background_color']
FeatureFlag.supported?(:red_background_color)
# => true
```

Next, we can create a custom adapter that will enforce the availability of feature flags before enabling or adding them. The adapter can be implemented as follows:

```rb
# lib/feature_flags/adapters/active_record_based.rb

# frozen_string_literal: true

require 'flipper'

module FeatureFlags
  module Adapters
    class ActiveRecordBased < Flipper::Adapters::ActiveRecord

      def add(feature)
        return false unless supported_feature_flag?(feature.name)

        super
      end

      def enable(feature, gate, thing)
        return false unless supported_feature_flag?(feature.name)

        super
      end

      def remove(feature)
        return false if supported_feature_flag?(feature.name)

        super
      end

      private

      def supported_feature_flag?(feature)
        FeatureFlag.supported?(feature)
      end

    end
  end
end
```

In this adapter, we check if the given feature flag is supported before performing actions such as adding, enabling, or removing it.

To include this custom adapter in the Flipper configuration, we can update the initializer file:

```rb
# config/initializers/flipper.rb

# frozen_string_literal: true

require 'flipper/adapters/active_record'

Rails.application.reloader.to_prepare do
  Flipper.configure do |config|
    config.adapter { FeatureFlags::Adapters::ActiveRecordBased.new }
  end
end
```

Now, with these validations in place, the restrictions prevent performing actions with unsupported flags:

```rb
Flipper.enable(:not_supported)
# => false
Flipper.enable(:red_background_color)
# => true
Flipper.remove(:red_background_color)
# => false
```

### Including information about the Feature Flags

As we mentioned earlier, feature flags should eventually be removed. It would be beneficial to have more information about each flag beyond just the name. Let's add additional fields to the feature flag value object, such as:

- **Description**
- **Expected expiration date**
- **Owner**

To achieve this, we can update the FeatureFlag value object:

```rb
# frozen_string_literal: true

class FeatureFlag

  class << self

    def all
      [
        {
          name: 'search',
          description: 'Search description',
          expected_expiration_date: 2024-01-01,
          owner: 'Backend team'
        },
        {
          name: 'red_background_color',
          description: 'Red background color description',
          expected_expiration_date: 2024-01-02,
          owner: 'Frontend team'
        }
      ]
    end

    def supported?(flag_name)
      all.map { _1[:name] }.include?(flag_name.to_s)
    end

  end

end
```

Now, each feature flag includes additional information such as description, expected expiration date, and owner.

Note: You can also consider [defining constants in a YAML file](https://dev.to/vladhilko/say-goodbye-to-messy-constants-a-new-approach-to-moving-constants-away-from-your-model-58i1) to simplify the interface and keep the information organized.


### Improving Spec Performance

To improve the performance of your tests, it is recommended to avoid unnecessary database hits when working with feature flags. You can achieve this by replacing the Active Record adapter with the Memory Adapter for your test environment.

Here's what you need to do:

Update your `config/initializers/flipper.rb file` as follows:

```rb
# config/initializers/flipper.rb

# frozen_string_literal: true

require 'flipper/adapters/active_record'

Rails.application.reloader.to_prepare do
  Flipper.configure do |config|
    config.adapter do
      Rails.env.test? ? Flipper::Adapters::Memory.new : Flipper::Adapters::ActiveRecord.new
    end
  end
end

```
With this configuration, the Memory Adapter will be used for the test environment, while the Active Record Adapter will be used for other environments.

Now, when you run rails console in the test environment (`rails c -e test`), you can verify that the database is not being triggered when working with feature flags:

```rb
# rails c -e t

Flipper.enable(:red_background_color)
Flipper.enable?(:red_background_color)
```

This change will help improve the performance of your test suite by eliminating unnecessary database hits.

## Conclusion

In this article, we've been diving deep into the world of feature flags. We explored the benefits of using feature flags, including risk mitigation, A/B testing, controlled rollouts, and faster development cycles.

We discussed different feature release strategies, such as the Big Bang Release, Incremental Release, Feature Flags/Toggles, Canary Release, Phased Rollout, and Fault Tolerance. Each strategy offers a unique approach to releasing new features or updates, providing flexibility and control over the rollout process.

To enhance our feature flag implementation, we made several improvements. We added validations to ensure that only supported flags can be enabled or added, preventing potential errors. We integrated caching to optimize performance by reducing unnecessary database hits. Additionally, we explored the options of providing a user interface and adding an API to expose feature flag information to clients.
