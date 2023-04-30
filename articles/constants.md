## Overview:

In this article, we will discuss the usage of constants in a Rails application. We'll take a deep dive into the existing approaches of defining constants in Rails, including the use of configuration files, initializers, and model files. We'll also examine the pros and cons of each approach.

We'll then provide a better approach to defining constants in Rails. We'll discuss the benefits, including how they can help prevent errors and make it easier to change values throughout the application. We'll also consider best practices for using constants in Rails, such as using meaningful names and limiting the scope of constants.

Finally, we'll implement a production-ready solution that solves most of the problems associated with the standard approach. By the end of the article, we will have a clear understanding of how to use constants in our Rails application and we'will be able to implement a more efficient and effective approach to defining constants.

## Definition

In simple terms, a constant is a special kind of value that never changes while a program is running.

**What standard options do we have for defining constants in Rails?** 🤷🏻‍♂️

When it comes to defining constants in Rails, we have a few standard options to choose from. Let's take a closer look at three of them:
- **Option 1.** Don't define constants at all and just use simple primitive data types such as strings, numbers, or symbols.
- **Option 2.** Define constants within the classes or modules where they are used.
- **Option 3.** Define global constants either in initializers or at the Rails configuration level.

### Option 1. Don't define constants at all and just use simple primitive data types such as strings, numbers, or symbols.

As an example, let's consider the following scenario:

```rb
def update
  animal = Animal.update(type: 'cat')

  render json: animal
end
```

In this example, we're just using the string 'cat' instead of defining a new constant. The main advantage of this approach is its simplicity, but there are many potential problems that may appear in the future. Let's discuss them more closely:

**Problems:**

- **Typos and errors.** When using strings instead of constants, it's easy to make typos or use slightly different string literals, which can lead to errors that are difficult to debug.
- **Maintenance and refactoring.** If you use the same string literal in multiple places throughout your code, it can be difficult to change or update the value later on. This can make your code harder to maintain and refactor.
- **Inconsistency.** If different parts of your code use different string literals to represent the same value, it can lead to inconsistencies and make your code harder to understand.
- **Lack of systematization.** It's not clear to which domain or model this string belongs, nor what other options are available. As a result, everything is scattered throughout the app, leading to disorganization and difficulties in maintenance and modification.

### Option 2. Define constants within the classes or modules where they are used.

The second option suggests defining a new constant somewhere on the business logic level, for example, in the model, service, or module. Let's take a look at this code:

```rb
# frozen_string_literal: true

class Animal < ApplicationRecord
  TYPES = ['cat', 'dog', 'pig']
  COLORS = ['green', 'red', 'white', 'blue']
end
```

We've defined a constant array on the model level. It solves some problems that were introduced in `Option 1` such as **Typos and errors**, **Inconsistency**, and **Lack of systematization**. However, we still have the following problems:

**Problems:**

- The interface is not user-friendly. You cannot access a `green` color, for example, by simply doing `Animal::COLORS.green`. Therefore, you have to define a new constant (for example, `COLOR_GREEN = 'green'`, `COLOR_WHITE = 'white'`) for each color or use a hash.
- Having a large number of constants in your model increases the model's complexity and reduces its readability, ultimately making it harder to maintain.
- You cannot define and use something that is not related to the business logic in the model. For instance, default system data such as `limit`, `offset`, and `pagination` cannot be defined in the model. This increases cognitive load and you have to always think about the best place to keep such data.
- When you define a constant within a class or module in your Rails model, it will only be accessible within that class or module. This can be problematic if you need to use the constant elsewhere in your code, as you may have to duplicate it or use an unattractive model prefix to access it.

### Option 3. Define global constants either in initializers or at the Rails configuration level.

In this option, we will consider defining constants at the global level. There are two ways to do it:
- **Define constants in the initializer file**
- **Define constants in the configuration file**

#### Define constants in the initializer file

We can define the constant in the initializer file, for example in `config/initializers/animal.rb`:

```rb
# config/initializers/animal.rb

TYPES = ['cat', 'dog', 'pig']
COLORS = ['green', 'red', 'white', 'blue']

```

This way, the constants will be loaded with the Rails application and can be accessed globally throughout the application. The main problems of this approach are:
- **Non-user-friendly interface**
- **Lack of namespaces**
- **Difficulty in maintenance and scalability**

#### Define constants in the configuration file

To overcome the disadvantages mentioned above, we can try using the following approach: creating separate YAML files containing all of our data for each model. Here's an example:

```yml
# config/animal.yml

types:
  - cat
  - dog
  - pig
colors:
  - green
  - red
  - white
  - blue
```

Then we will load this data in the `config/application.rb`:

```rb

# config/application.rb

# ...
config.animal = YAML.load_file("#{Rails.root}/config/animal.yml")
```

After that all data will be available by using the following interface:

```rb
Rails.configuration.animal

# {"types"=>["cat", "dog", "pig"], "colors"=>["green", "red", "white", "blue"]}
```

Although it solved some of the problems, it still looks strange. Therefore, we are going to implement a new approach that will gather all the advantages from every approach we discussed here.

---

## A new approach for defining constants

In this section, we'll consider creating a custom approach for defining constants to solve the problems that were mentioned above. We will start by **gathering requirements**, then we will define **the interface**. After that, we will **implement three approaches** to show the evaluation from the simplest to the production-ready approach.

### Requirements

Let's describe what we want to achieve in the end. Our custom constant should meet the following 5 requirements:

- The constant should be available globally.
- The constant should have a clear and straightforward interface.
- The constant should be easily extendable and maintainable.
- The constant should be frozen and not modifiable after initialization.
- The constant should be namespaced to avoid overwriting.

### Interface

What interface do we want for our custom implementation? Personally, I would like to have something that looks like `Rails.configuration.animal` from `Option 3` that we discussed earlier, but with 4 additional features:

- **Direct access to any array element:**

```rb
Rails.configuration.animal.types.cat
# => cat
Rails.configuration.animal.types.dog
# => dog
```
- **Ability to return all values from the defined array:**

```rb
Rails.configuration.animal.types.values
# => ['cat', 'dog', 'pig']
Rails.configuration.animal.colors.values
# => ['green', 'red', 'white', 'blue']
```

- **Using a name that is related to constant instead of `Rails.configuration`:**

```rb
Constants.animal.types.cat
# => cat 
```

- **Keep constants namespaced for each separated model:**

```rb
Constants.animal.types.cat
# => cat 
Constants.car.types.truck
# => truck
```

Putting it all together, we get the following interface:

```rb
Constants.animal.types.cat
# => cat 
Constants.animal.colors.green
# => green
Constants.car.types.truck
# => truck
Constants.animal.colors.values
# => ['green', 'red', 'white', 'blue']
Constants.car.types.values
# => ['sedan', 'minivan', 'truck']
```

### Implementation

We'll look at three different options for addressing the issue, beginning with the most straightforward and ending with the most comprehensive solution.

### Option 1. One class/module that contains all the methods.

In this option, we will initiate a new module called `Constants` and add methods for each model that will return us `OpenStruct` objects with all required data. So, we would have something like this:

```rb
# lib/constants.rb

# frozen_string_literal: true

module Constants
  def self.animal
    types = {
      cat: 'cat',
      dog: 'dog',
      pig: 'pig'
    }

    colors = {
      green: 'green',
      red: 'red',
      white: 'white',
      blue: 'blue'
    }

    OpenStruct.new(
      types: OpenStruct.new(values: types.values, **types).freeze,
      colors: OpenStruct.new(values: colors.values, **colors).freeze
    ).freeze
  end

  def self.car
    types = {
      sedan: 'sedan',
      minivan: 'minivan',
      truck: 'truck'
    }

    OpenStruct.new(
      types: OpenStruct.new(values: types.values, **types).freeze
    ).freeze
  end
end
```

And our interface would look like this:

```rb
Constants.animal.types.values
# => ["cat", "dog", "pig"]
Constants.car.types.values
# => ["sedan", "minivan", "truck"]
Constants.car.types.truck
# => "truck"
```

The interface looks good, but we still have many open issues that don't align with our requirements, like these ones:

- It is hard to maintain and extend because everything is in one place.
- There is a high risk of breaking something because we have to create every method from scratch.
- There is a high risk of invalid methods because no error is raised if a non-existent constant value is requested.
```rb
Constants.car.types.not_exist
# => nil
```

That's why we need to build a more scalable and reliable solution. Let's consider **Option 2**.

---

## Option 2. Storing constants in YAML files 

In this option, we will improve scalability and move all our constant data to YAML files, as we did in the example with Rails configuration above. Here is our plan:

- **Step 1.** Create a new YAML file for each model.
- **Step 2.** Load all these YAML files and combine them into one big hash.
- **Step 3.** Transform this hash into a nested OpenStruct object.
- **Step 4.** Assign this new OpenStruct object to a single constant.

### **Step 1.** Create a new YAML file for each model.

In this example, we will create 2 files: `animal.yml` and `car.yml`

- `animal.yml`

```yml
# config/constants/animal.yml

animal:
  types:
    - cat
    - dog
    - pig
  colors:
    - green
    - red
    - white
    - blue
  ages:
    one_week: 1 week
    one_month: 1 month
    one_year: 1 year
```

- `car.yml`


```yml
# config/constants/car.yml

car:
  types:
    - sedan
    - minivan
    - truck
```

### **Step 2.**  Load all these YAML files and combine them into one big hash.

To achieve this, we can do the following:

```rb
constant_hash = Dir.glob(File.join('config/constants', '*.yml')).reduce({}) do |hash, file_path|
  hash.merge(YAML.load_file(file_path))
end
# => 
{
 "animal"=>
  { 
    "types"=>["cat", "dog", "pig"],
    "colors"=>["green", "red", "white", "blue"],
    "ages"=>{
      "one_week"=>"1 week",
      "one_month"=>"1 month", 
      "one_year"=>"1 year"
     }
  },
 "car"=>
 {
   "types"=>["sedan", "minivan", "truck"]
 }
}
```

### **Step 3.**  Transform this hash into a nested OpenStruct object.


To transform the hash into a nested OpenStruct object, we can define the following method in the Hash class:

```rb
class Hash
  def to_open_struct
    JSON.parse to_json, object_class: OpenStruct
  end
end
```

And then use this method on the hash:

```rb
constant_open_struct = constant_hash.to_open_struct

constant_open_struct.animal.types
# => ["cat", "dog", "pig"]
constant_open_struct.animal.ages.one_week
# => "1 week"
```

### **Step 4.**  Assign this new OpenStruct object to a single constant.

Finally, we need to create a new constant and assign this new OpenStruct to it in the Rails initializer:

```rb
# config/initializers/constants.rb

# frozen_string_literal: true

Rails.application.config.after_initialize do
  # ...

  Kernel.const_set('Constants', constant_hash.to_open_struct)
end

```

Now we have the following interface:

```rb
Constants.animal.ages.one_month
# => "1 month"
Constants.animal.types
# => ["cat", "dog", "pig"]
Constants.car.types
# => ["sedan", "minivan", "truck"]
```

This solution is much more flexible now, but there are still many open problems:

- Constant values are not frozen.
- Without the hash, it is not possible to access elements from an array. For example, the following constant `Constants.animal.types.cat` doesn't work.
- The OpenStruct performance is not good.

That's why we need to consider **Option 3** to fix these problems.

---

## Option 3. Production Ready Solution

In the final solution, we aim to preserve all the advantages and address the drawbacks of the previous solution. This solution will be nearly identical to the previous one, except for one difference: we will create a new custom class instead of using OpenStruct. To implement this, we need to follow these four steps:

- **Step 1.** Create a new YAML file for each model.
- **Step 2.** Load all these YAML files and combine them into one big hash.
- **Step 3.** Create a new custom class to store our core logic for the new constant object.
- **Step 4.** Transform the hash from **Step 2** into an instance of the class created in **Step 3**. Then, assign this object to a new constant.

### **Step 1.** Create a new YAML file for each model.

As a first step, we'll add YAML files just like we did in the previous option:

- `animal.yml`

```yml
# config/constants/animal.yml

animal:
  limit: 10
  types:
    - cat
    - dog
    - pig
  colors:
    - green
    - red
    - white
    - blue
  ages:
    one_week: 1 week
    one_month: 1 month
    one_year: 1 year

```

- `car.yml`

```yml
# config/constants/car.yml

car:
  types:
    - sedan
    - minivan
    - truck
```

### **Step 2.** Load all these YAML files and combine them into one big hash.

At the second step, we will load all YAML files and combine them into one big hash, as we did in the previous option. Additionally, we will wrap it in a separate class for our convenience:

```rb
# lib/constant/load.rb

# frozen_string_literal: true

module Constant
  class Load

    def initialize(path)
      @path = path
    end

    def call
      Dir.glob(File.join(path, '*.yml')).reduce({}) do |hash, file_path|
        hash.merge(YAML.load_file(file_path))
      end
    end

    private

    attr_reader :path

  end
end
```

### **Step 3.** Create a new custom class to store our core logic for the new constant object.

Let's take a look at the implementation, here's how our new class will look like. Don't worry if it seems complicated; we'll go over how this class works later:

```rb
# lib/constant/model.rb

# frozen_string_literal: true

module Constant
  class Model

    def initialize(constant_hash = {})
      @constant_hash = constant_hash.deep_symbolize_keys.freeze
    end

    def deep_transform
      tap { constant_hash.each { |key, value| define_singleton_method(key) { initialize_value(value) } } }
    end

    # provides missing hash methods ( for example '.values' etc. )
    delegate_missing_to :constant_hash

    private

    attr_reader :constant_hash

    def initialize_value(value)
      case value
      when Hash then Model.new(value).deep_transform
      when Array then Model.new(value.index_by(&:itself)).deep_transform
      else value.freeze
      end
    end

  end
end
```

The idea behind this class is as follows:

- We send a hash from **Step 2** as an argument.
- We iterate over each key of this hash and define a new method for each key, for example:

```rb
# hash = { animal: { limit: 10 } }

def animal
  # ...
end
```

- In this method, there are three options that we can return depending on the value of the key:

**1. Primitive type.** If the hash value for this key is a primitive type, for example, a string/symbol/integer, then we return this value.

```rb
# hash = { limit: 10 }

def limit
  10
end
```

**2. Hash.** If the value for this key is a Hash, then we return an instance of our new class with this value as an argument:

```rb
# hash = { animal: { limit: 10 } }

def animal
  Constant::Model.new({ limit: 10 }).deep_transform
end
```

**3. Array.** If the value for this key is an Array, we transform this array into a hash and return an instance of our new class with this value as an argument:

```rb
# hash_with_array = { types: ['cat', 'dog'] }
# hash_with_hash = { types: { cat: 'cat', dog: 'dog' } } 

def types
  Constant::Model.new({ cat: 'cat', dog: 'dog' }).deep_transform
end
```

And don't forget to freeze everything. That's it.

### **Step 4.** Transform the hash from **Step 2** into an instance of the class created in **Step 3**. Then, assign this object to a new constant.

 Our final step is to bind **Step 2** and **Step 3**, and save the result to a new constant. We will move this initialization into a separate class:

```rb
# frozen_string_literal: true

module Constant
  class Initialize

    def initialize(path:, constant_name:)
      @path = path
      @constant_name = constant_name
    end

    def call
      Kernel.const_set(constant_name, Model.new(load_hash_from_yml).deep_transform)
    end

    private

    attr_reader :path, :constant_name

    def load_hash_from_yml
      Constant::Load.new(path).call
    end

  end
end
```

And we will run this class in the initializer:

```rb
# config/initializers/constants.rb

# frozen_string_literal: true

Rails.application.config.after_initialize do
  Constant::Initialize.new(path: 'config/constants', constant_name: 'Constants').call
end
```

Alright, now we have our desired interface:

```rb
Constants.animal.types.cat
# => cat 
Constants.animal.colors.green
# => green
Constants.animal.limit
# => 10
Constants.animal.ages.values
# => ["1 week", "1 month", "1 year"]
Constants.animal.colors.values
# => ['green', 'red', 'white', 'blue']
Constants.car.types.truck
# => truck
Constants.car.types.values
# => ['sedan', 'minivan', 'truck']
```

## Conclusion

Here's the main benefit that we receive using this new approach:

- **Clear Interface.**
- **Maintenance and refactoring.**
- **DRY code.**
- **Data validity.** 
> You can prevent typos and invalid data from being saved.
- **Systematization.** 
> It's easy to understand which domain/model a value belongs to and what other options are available.
- **Immutability.** 
> Immutable values prevent accidental changes and ensure that the value remains consistent throughout your application.
- **Consistency.** 
> You can ensure that the same value is represented consistently throughout your code.
