## Overview:

In this article, we will discuss the usage of `ActiveSupport::Concern`. We will delve into what it is, why we need it, and the problems it can solve. Additionally, we will explore the pros and cons associated with its usage. To illustrate its practical implementation, we will provide three examples.

## Definition:

In simple terms, `Concern` allows you to extract common logic from different models into a reusable module. You may ask, "Why can't we just use a module? Why do we need to define something else?" Here are the main differences: Essentially, `Concern` is an ordinary module, but it allows you to include any `ActiveRecord` behaviors that you can define on the model. Therefore, we can consider it as a module for encapsulating and reusing `ActiveRecord` methods.

**Advantages:**

- **Code Reusability:** ActiveSupport Concern allows us to follow the DRY principle by extracting common logic into reusable modules.

**Problems:**

- **Complex Dependencies:** Overusing concerns can result in a complex web of dependencies, making the code harder to understand and maintain.
- **Difficulty in Code Search:** Finding the definition of a specific behavior or functionality can be more challenging due to the dispersed nature of concerns.

In essence, concerns are well-suited for storing technical components, such as:
- Adding file helpers to facilitate handling files with a clear interface.
- Creating wrappers for existing attributes and values to enhance interface clarity.
- Implementing technical features like change history tracking, logging, and more.
- etc...

However, concerns are not well suitable for storing business logic.

## Implementation

In the following section, we will discuss the key elements of concerns. To understand concerns, we only need to know what a module is and the `included` and `class_methods` methods from `ActiveSupport::Concern`.

### Module

A module in Ruby is a container that groups together related methods, constants, and other module or class definitions. It provides a way to organize and reuse code in a modular manner. And Сoncern is just a module, so we need to know how to create it.

```rb
# app/models/concerns/concern_sample.rb

# frozen_string_literal: true

module ConcernSample
  def hello_world
    puts 'Hello World!'
  end
end
```

### ActiveSupport::Concern methods

So, what is the main difference between a simple Ruby module and a concern? The main differences are that when you include `ActiveSupport::Concern` in a module, two new methods are added. These methods are:

- `included`

This method enables us to add Active Record logic directly to the module and reuse it at the model level. For example:

```rb
# app/models/concerns/concern_sample.rb

# frozen_string_literal: true

module ConcernSample
  extend ActiveSupport::Concern

  included do
    # Define class methods here
    def self.my_class_method
      # Class method logic
    end

    # Define instance methods here
    def my_instance_method
      # Instance method logic
    end

    # Define associations here
    has_many :comments
    belongs_to :category

    # Define validations here
    validates_presence_of :name
    validates_uniqueness_of :email

    # Define callbacks here
    before_save :do_something
    after_create :do_something_else

    # Define scopes here
    scope :active, -> { where(active: true) }
    scope :recent, -> { order(created_at: :desc) }
  end
end

```

- `class_methods`

This method allows us to add methods that can be called from the class itself:

```rb
# app/models/concerns/concern_sample.rb

# frozen_string_literal: true

module ConcernSample
  extend ActiveSupport::Concern

  class_methods do
    def my_class_method
      # Class method logic
    end
  end
end

```

Let's consider three examples to gain a better understanding of this concept.

## Example 1: FullDescription.

In the first example, we will consider a very basic scenario to gain an understanding of how this actually works. For instance, let's assume we have the following model:

```rb
# app/models/animal.rb

class Animal < ApplicationRecord
end
```

Then we decided to add a method to the model that would provide the full description of this animal.

```rb
# app/models/animal.rb

class Animal < ApplicationRecord
  def full_description
    "Name: #{name}. Kind: #{kind}. Status: #{status}."
  end
end
```

Here's how it works:

```rb
animal = Animal.first
animal.full_description
# => "Name: Simba. Kind: Cat. Status: Adopted"

animal = Animal.second
animal.full_description
# => "Name: Rocky. Kind: Rabbit. Status: Available"
```

What do we need to do if we want to move this logic to the concern? First of all, we need to create a new concern:

```rb
# app/models/concerns/full_description.rb

# frozen_string_literal: true

module FullDescription
  def full_description
    "Name: #{name}. Kind: #{kind}. Status: #{status}"
  end
end

```

And include this concern at the model level:

```rb
# app/models/animal.rb

class Animal < ApplicationRecord
  include FullDescription
end
```

So the same interface is still available:

```rb
animal = Animal.first
animal.full_description
# => "Name: Simba. Kind: Cat. Status: Adopted"
```

We didn't include `ActiveSupport::Concern` here just to highlight that for simple methods without `ActiveRecord` logic, it will still work fine. However, it will also work with `ActiveSupport::Concern`:

```rb
# app/models/concerns/full_description.rb

# frozen_string_literal: true

module FullDescription
  extend ActiveSupport::Concern

  included do
    def full_description
      "Name: #{name}. Kind: #{kind}. Status: #{status}."
    end
  end

end
```

Like this:

```rb
animal = Animal.first
animal.full_description
# => "Name: Simba. Kind: Cat. Status: Adopted."
```

And if you want to add class methods, we can do the following:

```rb
# app/models/concerns/full_description.rb

# frozen_string_literal: true

module FullDescription
  extend ActiveSupport::Concern

  class_methods do
    def full_description
      all.map { _1.full_description }.join(' ')
    end
  end

end
```

Now the `full_description` method is available on the `Animal` class:

```rb
Animal.full_description
# => "Name: Simba. Kind: Cat. Status: Adopted. Name: Rocky. Kind: Rabbit. Status: Available."
```

## Example 2: UUID

In the second example, we will create a concern that generates and saves a unique UUID value before creating a new database record. To achieve this, we will define a new concern as follows:

```rb
# app/models/concerns/uuid.rb

# frozen_string_literal: true

module UUID
  extend ActiveSupport::Concern

  included do
    before_create :set_uuid

    private

    def set_uuid
      self.uuid ||= SecureRandom.uuid
    end
  end
end
```

And we will include it at the model level:

```rb
# app/models/animal.rb

class Animal < ApplicationRecord
  include UUID
end
```

Then, every time we create a new Animal record in the database, the UUID will be automatically generated for us:

```rb
animal = Animal.create
animal.uuid
# => '7f934646-bc67-446c-920d-ca5415d28b94'
```

P.S. To use the UUID acronym, you need to add the following code to `config/initializers/inflections.rb`:

```rb
# config/initializers/inflections.rb

ActiveSupport::Inflector.inflections do |inflect|
  inflect.acronym 'UUID'
end
```

## Example 3: PredicateMethods

In the last example, let's consider adding a concern that allows us to simplify the interface for attributes with a fixed number of available values. For instance, if an `animal` record has a `status` attribute with the following available values: `['available', 'adopted', 'pending']`, then we would like to have the following interface:

```rb
animal = Animal.create(status: 'available')

animal.available?
# => true
animal.adopted?
# => false
animal.pending?
# => false 

# or with prexix

animal.status_available?
# => true
animal.status_adopted? 
# => flase

# etc ..
```

What do we need to do to implement it? Let's create a new concern called `PredicateMethods` and add its method to the model:

```rb
# app/models/animal.rb

class Animal < ApplicationRecord
  include PredicateMethods

  define_predicate_methods attribute: :status,
                           available_values: ['available', 'adopted', 'pending'],
                           prefix: false
end
```

So how is this concern implemented? It's quite simple. We just need to define a new method called `define_predicate_methods`, and this method will generate new methods on the model for every available value that you pass as an argument. Inside this method, we will check if the current status value is equal to the given value or not. Here's the implementation:

```rb
# app/models/concerns/predicate_methods.rb

# frozen_string_literal: true

module PredicateMethods
  extend ActiveSupport::Concern

  MethodAlreadyDefined = Class.new(ArgumentError)

  class_methods do
    def define_predicate_methods(attribute:, available_values:, prefix: false)
      available_values.each do |value|
        method_name = [(prefix ? "#{attribute}_" : ''), value, '?'].join.to_sym
        raise MethodAlreadyDefined, "#{method_name} already defined" if instance_methods.include?(method_name)

        define_method(method_name) do
          public_send(attribute) == value
        end
      end
    end
  end
end
```

Now our interface works as desired:

```rb
animal = Animal.last

animal.available?
# => true
animal.adopted?
# => false
```

If you'd like to add the `status` prefix, you just need to change the value of the `prefix` attribute from `false` to `true`:

```rb
animal = Animal.last

animal.status_available?
# => true

animal.status_adopted?
# => false
``` 

That's it! Now we can add this concern to any of our models and make their interfaces more readable and simplified.

## Conclusion

In conclusion, in this article we delved into the usage of `ActiveSupport::Concern` in Rails. We explored what it was, why we needed it, and the problems it could solve. We examined the pros and cons of utilizing this approach, considering different perspectives. Furthermore, to enhance our understanding of the concept, we provided three examples that illustrated its practical implementation.
