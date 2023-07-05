## Overview:

In this article, we will discuss the usage of Platform Settings in a Rails application. We will explore why we need them and the specific problems they aim to solve. Additionally, we will consider four different available options, examining the pros and cons of each. Furthermore, we will fully implement the most suitable approach and explore how to ensure the quality of platform settings through contracts and tests. By the end of the article, we will have a production-ready solution to this problem.

## Definition:

In simple terms, platform settings in a Rails application allow us to configure different behaviors for various platforms within the same application. To provide clarity, let's consider three different examples.

- **Example 1: McDonald's Franchise**

Let's imagine that you're creating an application for a McDonald's franchise. After the initial version is ready, they approach you with a request for adjustments specific to another country. These adjustments may include a different menu, enabling specific promotional actions, implementing logic for alcohol sales, and so on. You successfully implement these changes. However, they approach you again, this time requesting adjustments for a third customer who has purchased their own franchise. This scenario prompts you to seek a more generalized solution, as potentially every franchise could have hundreds of differences, and the number of franchise customers can grow rapidly.

- **Example 2: Betting Sites**

Let's consider the example of a betting site. You have created the initial fully functional version of the site. However, after some time, a customer approaches you with a request for changes because they intend to run the site on a different platform in another market with distinct regulatory rules. These changes may include implementing rules that prohibit registration for individuals under 18, sending information about profits to the government, and enforcing restrictions for addicted gamblers, among other requirements. With numerous new markets and diverse regulations, you realize the necessity of developing a robust solution to handle these various issues and simplify your workflow.

- **Example 3: Digital Banking Service**

Let's explore the example of digital banking services. You have developed a bank application that functions effectively within your local market. However, you recognize the opportunity to offer this banking service to other customers, enabling them to create their own banks using your core platform. Consequently, you realize the need to introduce logic that allows for flexibility in accommodating different customers. This realization prompts you to implement platform settings within your application, ensuring the adaptability and customization required to support various banking institutions.

___

## Exploring Implementation Options: Choosing the Right Approach

In this chapter, we will explore four different possible approaches to solving the problems discussed in the previous examples. Let's examine each of them in detail:

- **Option 1. Create a Separate Branch to Implement the New Logic**
- **Option 2. Applying Conditional Operators**
- **Option 3. Setting Up a Database Configuration Table**
- **Option 4. Adding Platform Settings YAML File Configurations**

### Option 1. Create a Separate Branch to Implement the New Logic

One possible solution that might initially come to mind is to create a separate branch for each new platform and make all the necessary changes there.

**Advantages:**

- No need to implement additional logic to support platform differentiation.
- Code remains clear and does not include unnecessary conditional operators.

**Problems:**

- Synchronization with the core platform:
  - Resolving conflicts becomes a complex task when changes overlap in the same location.
  - There is a high risk of introducing errors or disrupting functionality.
  - Maintenance becomes more challenging.

### Option 2. Applying Conditional Operators

The second solution that you can consider is to create multiple `if/else` conditions directly in the code. For example, something like this:

```rb
if platform == "Platform A"
  # Platform A specific code
elsif platform == "Platform B"
  # Platform B specific code
elsif platform == "Platform C"
  # Platform C specific code
# Add more conditions for additional platforms as needed
else
  # Default code for unsupported platforms
end
```

**Advantages:**
- Straightforward implementation without the need for separate branches.

**Problems:**
- Code complexity and readability:
  - The code can become cluttered and harder to understand as the number of platforms and conditions grows.
- Maintenance challenges:
  - Modifying or adding platform-specific logic requires modifying the main codebase, which can introduce errors and make maintenance more difficult.
- Lack of flexibility:
  - Adding or removing platforms requires modifying the code, potentially leading to more significant changes and potential regressions.

### Option 3. Setting Up a Database Configuration Table

The third option is to create a separate table to store all configurations. For example, if we have platform settings for the admin page, it might look like this:

```rb
platform_settings = PlatformSettings.create(
  admin_enabled: true, 
  admin_show_page_1: true,
  admin_show_page_2: true,
  admin_available_pages: ['system', 'companies', 'configurations']
)

platform_settings.admin_enabled
# => true
platform_settings.admin_show_page_1
# => true
platform_settings.admin_show_page_2
# => true
platform_settings.admin_available_pages
# => ['system', 'companies', 'configurations']
```

**Advantages:**

- Provides a user-friendly interface for managing platform settings.
- Offers ease of modification and updates to platform settings configurations.

**Problems:**

- Direct dependency on the database:
  - Relying solely on the database for configuration settings may not be the most efficient approach, especially if the settings only need to be set once during the platform's launch.
  - Overreliance on the database can potentially impact performance.
  - In some cases, it may be necessary to check settings before establishing a database connection.
  - Increases complexity in writing tests.

### Option 4. Adding Platform Settings YAML File Configurations

The last option, which we are going to implement, is a configuration YAML file. We will create multiple `.yml` files, similar to this example:

```yml
first_platform:
  enabled: true
second_platform:
  enabled: true
third_platform:
  enabled: false
```

And transform them into an appropriate interface, like this:

```rb
platform.enabled?
```

Advantages:

- Provides a clear and accessible interface for working with platform-specific configurations.
- Separates configurations from the business logic, consolidating them in one central location.
- Offers flexibility for extending and modifying configurations as needed.

Problems:

- Requires additional logic to load and parse the YAML files.
- Potential for inconsistencies if the YAML files and the codebase are not synchronized.
- Platform setting changes can only be performed by developers, restricting customer access to configuration modifications.

Let's examine this option more precisely.

___

## Adding Platform Settings YAML File Configurations. Implementation.

In this chapter, we will explore the implementation of platform settings using a YAML configuration file. As always, we will start with the simplest solution and gradually enhance it. We will begin by implementing the initial solution and then progress to make it more flexible and convenient. Additionally, we will incorporate logic to ensure the validity of our configuration files and explore options for writing specifications for these settings.

Here is our plan:

- **Creating the basic solution**
- **Improving the basic solution**
- **Adding contracts to guarantee integrity and synchronization**
- **Integrating the solution with tests**

## Creating the basic solution

Let's explore the basic solution and outline the steps required for its implementation:

- **Step 1: Add a new environment variable that indicates the current platform.**
- **Step 2: Add YAML files containing configurations for each platform.**
- **Step 3: Develop the necessary code to handle the YAML files and create a suitable interface.**

Now, let's begin the implementation process.

#### Step 1: Add a new environment variable that indicates the current platform.

We need to add `CURRENT_PLATFORM` to the list of environment variables.

```
# .env

CURRENT_PLATFORM='first_platform'
```

#### Step 2: Add YAML files containing configurations for each platform.

Let's examine the platform settings for the admin, as mentioned earlier, across the three available platforms: `first_platform`, `second_platform`, and `third_platform`. To accomplish this, we need to create a dedicated file named `config/settings/platform/admin.yml`. In this file, we will define the configuration options.

```yml
# config/settings/platform/admin.yml

first_platform:
  enabled: true
  show_page_1: true
  show_page_2: true
  available_pages:
    - system
    - companies
    - configurations
second_platform:
  enabled: true
  show_page_1: false
  show_page_2: false
  available_pages:
    - system
third_platform:
  enabled: false
  show_page_1: false
  show_page_2: false
  available_pages: []
```

Additionally, let's create a second file for the sake of clarity:

```yml
# config/settings/platform/animal.yml

first_platform:
  enabled: true
second_platform:
  enabled: true
third_platform:
  enabled: false
```

#### Step 3: Develop the necessary code to handle the YAML files and create a suitable interface.

My objective is to load the content from the YAML files and retrieve data exclusively for the current platform (`first_platform`). The expected interface should resemble the following:

```rb
Settings::Platform::Repository.admin
# => {"enabled"=>true, "show_page_1"=>true, "show_page_2"=>true, "available_pages"=>["system", "companies", "configurations"]}
Settings::Platform::Repository.animal
# => {"enabled"=>true}
```

Let's explore the necessary steps for implementation:

- **Step 1:** Retrieve a list of all files from the `config/settings/platform` directory.
- **Step 2:** Iterate through the obtained list and define a class method for each file within the `Settings::Platform::Repository`, such as `Settings::Platform::Repository.admin`, `Settings::Platform::Repository.animal`, and so on.
- **Step 3:** Ensure that the method created in `Step 2` returns the YAML file's hash content exclusively for the current platform.
- **Step 4:** Execute these steps within the Rails initializer.

Below is the code implementation:

```rb
# frozen_string_literal: true

module Settings
  module Platform
    class Repository

      CONFIG_DIRECTORY = 'config/settings/platform'
      CONFIG_FILE_EXTENSION = '.yml'
      DEFAULT_PLATFORM = 'first_platform'

      class << self

        def load!
          all_platform_settings_config_files.each do |file_path|
            define_platform_setting_method(
              method_name: File.basename(file_path, CONFIG_FILE_EXTENSION),
              content: YAML.load_file(file_path)
            )
          end
        end

        def current_platform
          ENV.fetch('CURRENT_PLATFORM', DEFAULT_PLATFORM)
        end

        private

        def all_platform_settings_config_files
          Dir.glob(File.join(CONFIG_DIRECTORY, "*#{CONFIG_FILE_EXTENSION}"))
        end

        def define_platform_setting_method(method_name:, content:)
          deep_freeze(content)
          define_singleton_method(method_name) { content[current_platform] }
        end

        def deep_freeze(enum)
          enum.freeze.each { |item| item.respond_to?(:each) ? deep_freeze(item) : item.freeze }
        end

      end

    end
  end
end

```

I have added a `load!` method to facilitate the initialization process, and I have also included a `deep_freeze` method to prevent modifications to the hash after initialization.

Let's verify if it works as intended. We will execute the `load!` method within the Rails configuration `before_initialize` block, as illustrated below:

```rb
# config/initializers/platform_settings.rb

# frozen_string_literal: true

require 'settings/platform/repository'

Rails.application.config.before_initialize do
  Settings::Platform::Repository.load!
end
```

Now, let's take a look at the updated interface:

```rb
Settings::Platform::Repository.admin.fetch('enabled')
# => true
Settings::Platform::Repository.admin.fetch('show_page_1')
# => true
Settings::Platform::Repository.animal.fetch('enabled')
# => true 
```

___

#### What can we improve in the interface described above?

___

There are two main problems with the current implementation:

- **Problem 1: Requiring the full name every time**
- **Problem 2: Unoptimized interface with hash usage**

Let's describe them more precisely.

**Problem 1: Requiring the full name every time**

In the given example, we have to use Settings::Platform::Repository every time we want to access the platform configuration. This can become cumbersome and repetitive, requiring us to create helper methods each time, such as:

```rb
def platform_admin
  @platform_admin ||= Settings::Platform::Repository.admin
end

platform_admin.fetch('enabled')
# => true
platform_admin.fetch('show_page_1')
# => true
```

However, it can be quite cumbersome and tedious to define these helper methods every time, particularly when dealing with numerous platform settings. It would be ideal if we could find a solution that eliminates the need for creating such helpers altogether.

**Problem 2:Unoptimized interface with hash usage**

The current implementation utilizes a hash-based interface for accessing the platform settings, which can make the code less readable and more error-prone.

So, what can we do to address this issue? What kind of interface would be more desirable? Ideally, we would like to have a simplified and intuitive interface that allows us to access the platform settings without the need for explicit helper methods. Here's an example of the desired interface:

```rb
platform_admin.enabled?
# => true
platform_admin.show_page_1?
# => true 
platform_admin.available_pages
# => ['system', 'companies', 'configurations']
```

However, I prefer not to have a global variable accessible throughout the entire application. Instead, I would like to include these settings only in the specific context where I actually intend to use them, similar to how we utilize modules or mixins. Therefore, my desired interface should look like this

```rb
include Settings::Platform[:admin]

platform_admin.enabled? # => true
```

Let's delve into the implementation of this approach.


___

## Improving the basic solution

___

How can we achieve the desired interface above? What do we need? Let's take a closer look. We need to make three improvements:

- **Improvement 1: Including the module with all platform settings helpers inside.**
- **Improvement 2: Returning an object with the appropriate interface instead of a hash.**
- **Improvement 3: Combining Improvements 1 and 2 together.**


### Improvement 1: Including the module with all platform settings helpers inside.

Let's consider the following code. In this code, we create a module with our helpers inside so that we can include it instead of defining new helpers every time.

```rb
module PlatformAdmin
  def platform_admin
    @platform_admin ||= Settings::Platform::Repository.admin
  end
end

include PlatformAdmin 
```

But this approach is not flexible; we would have to create too many modules for every platform setting file. I'd like these modules to be created automatically just by the platform setting name. How can we do it? Actually, it's quite simple. We need to define a method that will create a new module instance based on the platform name and dynamically define methods inside. Here's how it may look:

```rb
# lib/settings/platform/mixin.rb

# frozen_string_literal: true

module Settings
  module Platform
    module Mixin

      def self.platform_module(name)
        Module.new do
          define_method "platform_#{name}".to_sym do
            Settings::Platform::Repository.public_send(name)
          end
        end
      end

    end
  end
end
```

Now we can do the following:

```rb
include Settings::Platform::Mixin.platform_module(:admin)

platform_admin
# => {"show_profit_calculation_page"=>true}
```

So instead of creating a helper method every time, we can include them via the module.


### Improvement 2: Returning an object with the appropriate interface instead of a hash.

Returning the raw hash from the configuration doesn't seem convenient. So let's change it and return a new object that transforms the hash into an object with the appropriate interface. To achieve this, we need to do the following:

- Create a separate class that accepts the platform settings name and performs the following actions:
  - Loads the platform setting hash by name
  - Iterates through every hash key
  - Defines suitable methods instead of fetch for every hash key

Here's the implementation:

```rb
class PlatformStruct < SimpleDelegator

  def initialize(name)
    super(Settings::Platform::Repository.public_send(name))

    generate_methods
  end

  private

  def generate_methods
    keys.each do |key|
      define_singleton_method(key) { self[key] }
      define_singleton_method("#{key}?") { self[key] }
    end
  end

end
```

And now we can do the following:

```rb
platform_struct = PlatformStruct.new('admin')

platform_struct.enabled
# => true 
platform_struct.enabled?
# => true
platform_struct.available_pages
# => ["system", "companies", "configurations"]
```

### Improvement 3: Combining Improvements 1 and 2 together.

The last step is to combine Improvements 1 and 2 together. We will also change the name from `platform_module` to `[]` to make it more readable. Instead of using the `Settings::Platform::Repository` class, we will use the `PlatformStruct` class from `Improvement 2`. Here's the updated code:

```rb
# lib/settings/platform/mixin.rb

# frozen_string_literal: true

module Settings
  module Platform
    module Mixin
      def self.[](name)
        Module.new do
          define_method "platform_#{name}".to_sym do
            PlatformStruct.new(name)
          end
        end
      end

      class PlatformStruct < SimpleDelegator

        def initialize(name)
          super(Settings::Platform::Repository.public_send(name))

          generate_methods
        end

        private

        def generate_methods
          keys.each do |key|
            define_singleton_method(key) { self[key] }
            define_singleton_method("#{key}?") { self[key] }
          end
        end

      end
    end
  end
end

```

And here's our new interface:

```rb
include Settings::Platform::Mixin[:admin]
include Settings::Platform::Mixin[:animal]

platform_admin.enabled?
# => true
platform_animal.enabled?
# => true
```

## Adding contracts to guarantee integrity and synchronization

There's still one problem with our approach. What happens if someone makes a mistake and adds incorrect configuration? For example, if the configuration has a different structure or incorrect data types:

```yml
first_platform:
  enabled: true
  show_page_1: true
  show_page_2: true
  available_pages:
    - system
    - companies
    - configurations
second_platform:
  enabled: true
  show_page_1: 'yes' # Incorrect data type
  show_page_2: 'no'  # Incorrect data type
third_platform:
  show_page_1: 'enabled' # Incorrect data type
  available_pages: 'Text' # Incorrect data type
```

That's not appropriate, and we need to ensure that the structure is the same for every platform and that all attributes are mirrored and have valid types. To achieve this, we can use the [dry-validation](https://github.com/dry-rb/dry-validation) gem, which provides a powerful validation library for Ruby.

To install the `dry-validation` gem, you can add it to your Gemfile and run bundle install. Here's an example of how to install it:

```rb
# Gemfile

gem 'dry-validation'
```

### Contract implementation

Once the gem is installed, we can use it to define a schema for validating the platform settings configuration. The schema will specify the expected structure, attribute types, and any other validation rules.

Here's an example of how we can use `dry-validation` to define a schema and validate the platform settings configuration:

- **Admin**

```rb
# spec/lib/settings/platform/contracts/admin.rb

# frozen_string_literal: true

module Settings
  module Platform
    module Contracts
      class Admin < Dry::Validation::Contract

        schema do
          required(:enabled).filled(:bool)
          required(:show_page_1).filled(:bool)
          required(:show_page_2).filled(:bool)
          required(:available_pages).array(:string, included_in?: %w[system companies configurations])
        end

      end
    end
  end
end
```

- **Animal**

```rb
# spec/lib/settings/platform/contracts/animal.rb

# frozen_string_literal: true

module Settings
  module Platform
    module Contracts
      class Animal < Dry::Validation::Contract

        schema do
          required(:enabled).filled(:bool)
        end

      end
    end
  end
end
```

After we have defined our contracts, let's test how they actually work by validating some sample data against the schema. Here's an example

```rb
Settings::Platform::Contracts::Admin.new.call({show_page_1: 'enabled', available_pages: 'Text'}).errors.to_h
# => { :enabled=>["is missing"], :show_page_1=>["must be boolean"], :show_page_2=>["is missing"], :available_pages=>["must be an array"] }

Settings::Platform::Contracts::Animal.new.call({enabled: true}).errors.to_h
# => {} 
```

Basically, we pass a hash as an argument to the contract, and the contract checks the hash against its defined conditions. If any errors or inconsistencies are found, it will return a result object containing the specific error messages.

### Contract validations

To ensure that the configuration data adheres to the defined contracts, we can create a test spec that iterates over all platform settings files and performs the contract validation. Here's how the spec may look:

```rb
# spec/lib/settings/platform/contracts_spec.rb

# frozen_string_literal: true

require 'rails_helper'

Dir[Rails.root.join('spec', 'lib', 'settings', 'platform', 'contracts', '**', '*.rb')].sort.each(&method(:require))

RSpec.describe Settings::Platform::Contracts do
  Dir.glob('config/settings/platform/*.yml').each do |settings_file|
    ['first_platform', 'second_platform', 'third_platform'].each do |platform|
      context "when checking #{settings_file} for #{platform}" do
        subject { contract_class.new.call(platform_settings_content).errors.to_h }

        let(:contract_class) { "#{described_class}::#{File.basename(settings_file, '.yml').camelize}".constantize }
        let(:platform_settings_content) { YAML.load_file(settings_file).fetch(platform).deep_symbolize_keys }

        it { is_expected.to be_empty }
      end
    end
  end

  context 'when checking existing spec files' do
    let(:spec_contract_file_name_list) do
      Dir.glob(Rails.root.join('spec', 'lib', 'settings', 'platform', 'contracts', '*.rb'))
        .map { File.basename(_1, '.rb') }
    end

    let(:platform_settings_file_name_list) do
      Dir.glob(Rails.root.join('config', 'settings', 'platform', '*.yml'))
        .map { File.basename(_1, '.yml') }
    end

    it 'every contract is expected to have a matching platform settings file' do
      expect(spec_contract_file_name_list).to match_array(platform_settings_file_name_list)
    end
  end
end
```

That's it. Now we can be confident that the structure and content of our platform settings are valid. The last aspect I want to highlight is writing specs and manipulating these settings. We aim to make our tests as flexible and straightforward as possible.

## Integrating the solution with tests

There are two strategies that we can consider during testing to manipulate the platform settings and obtain different content:
- **Option 1. Stubbing Platform Settings with Custom Values**
- **Option 2: Overriding ENV Variable to Use Content from Another Platform**

### Option 1. Stubbing Platform Settings with Custom Values

In this approach, we can use stubbing techniques to override the behavior of the platform settings and return custom values. This allows us to simulate different scenarios and test our code under various configurations. For example, we can stub a specific setting to return a different value than what is defined in the actual configuration file. To make this work, we are going to define a shared context with the following interface:

```rb
include_context 'when platform settings are', admin: {
  'enabled' => false,
  'show_page_1' => false,
  'available_pages' => ['system']
}
```

Here's how we can use it in the spec:

```rb
# spec/lib/settings/platform/sample_spec.rb

# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'Sample' do
  describe '#platform_admin' do
    subject(:platform_admin) { klass.new.platform_admin }

    let(:klass) do
      Class.new do
        include Settings::Platform::Mixin[:admin]
      end
    end

    include_context 'when platform settings are', admin: {
      'enabled' => false,
      'show_page_1' => false,
      'available_pages' => ['system']
    }

    it 'returns correct platform specific values' do
      expect(platform_admin.enabled).to be false
      expect(platform_admin.enabled?).to be false

      expect(platform_admin.show_page_1).to be false
      expect(platform_admin.show_page_1?).to be false

      expect(platform_admin.available_pages).to eq(['system'])
    end
  end
end

```

In the shared context definition, we stubbed `Settings::Platform::Repository` by replacing it with a custom hash value that is sent as an argument in the spec.

```rb
# spec/support/shared_contexts/platform_settings.rb

# frozen_string_literal: true

RSpec.shared_context 'when platform settings are' do |name_and_settings|
  before do
    name_and_settings.each do |name, settings_data|
      allow(Settings::Platform::Repository).to receive(name).and_return(settings_data)
    end
  end
end
```

Also, don't forget to include this shared context in `spec/rails_helper.rb`. This ensures that the shared context is available for all specs in your Rails application.

```rb
# spec/rails_helper.rb

require 'support/shared_contexts/platform_settings'
```

That's it. Now we can test more easily and not depend on the settings.

### Option 2: Overriding ENV Variable to Use Content from Another Platform

Another strategy is to override the ENV variable that determines the current platform. By changing the value of the ENV variable, we can force the code to load platform settings from a different platform's configuration file. This approach allows us to test how our code behaves with different platform settings without modifying the actual configuration files. We'll achieve this by utilizing the [Climate Controle](https://github.com/thoughtbot/climate_control) gem. It allows us to temporarily modify environment variables within the scope of our tests, ensuring that our code behaves correctly under different platform configurations. By using Climate Control, we can simulate different platform environments without affecting the actual system-wide environment variables.

To make it work, we will add metadata named `current_platform` with the desired platform value. This metadata will be associated with our test example or test suite, depending on how we want to configure it:

```rb
# spec/lib/settings/platform/sample_spec.rb

# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'Sample' do
  describe '#admin' do
    subject(:platform_admin) { klass.new.platform_admin }

    let(:klass) do
      Class.new do
        include Settings::Platform::Mixin[:admin]
      end
    end

    it 'returns correct platform specific values', current_platform: 'third_platform'  do
      expect(platform_admin.enabled).to be false
      expect(platform_admin.enabled?).to be false

      expect(platform_admin.show_page_1).to be false
      expect(platform_admin.show_page_1?).to be false

      expect(platform_admin.available_pages).to eq([])
    end
  end
end

```

Our `current_platform` RSpec metadata logic looks like this:

```rb
# spec/support/current_platform.rb

# frozen_string_literal: true

RSpec.configure do |config|
  config.around(:example, :current_platform) do |example|
    ClimateControl.modify CURRENT_PLATFORM: example.metadata[:current_platform] do
      example.run
    end
  end
end
```

Also, don't forget to include it in your `spec/rails_helper.rb` file. 

```rb
# spec/rails_helper.rb

require 'support/current_platform'
```

That's it!


## Conclusion

In this article, we have explored the concept of Platform Settings in a Rails application and delved into the reasons behind their necessity. We have identified the specific problems they aim to solve and discussed four different available options to implement them, weighing their respective advantages and disadvantages. Based on our analysis, we have chosen the most suitable approach and implemented it in detail, addressing the issues encountered along the way.
