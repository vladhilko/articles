## Overview:
> In this article, we will cover the use of form objects in Ruby on Rails applications. We will explore three different approaches to implementing form objects in Rails: `ActiveRecord` Form Objects, `Virtus` Form Objects and `DRY-rb` Form Objects. We will provide examples and discuss the pros and cons of each approach.

## Definition

In simple terms, `Form Object` is a design pattern that handles and validates data before saving it to the database.

**Why do we need it and what problems can this pattern solve?** 🤷🏻‍♂️

Let's take a look at the following example.

Imagine we have the model:

```rb
# app/models/animal.rb

class Animal < ApplicationRecord

  attribute :name, :string

  validates :name, presence: true

end
```

And controller:

```rb
def create
  animal_params = params.require(:animal).permit(:name)
  animal = Animal.create(animal_params)

  render json: animal
end
```

**What are we doing here?**

- We accept only allowed data ( `params.require(:animal).permit(:name)` )
- We convert data to the specific type  ( `attribute :name, :string` )
- We validate the data ( `validates :name, presence: true` )
- We save this data the database (`Animal.create(animal_params)`)
- We render the saved record (`render json: animal`)

**Do we have any problems with this approach?** 🤔

Yes, we have.

- We share data preparation responsibilities between `controller` and `model`
- We violate the Single Responsibility Principle because the model becomes responsible for data preparation and validation.
- We cannot use different validation rules for different cases within the same model.

>For example, sometimes we want to have different attributes and validations depending on where the code is run, such as different validation rules for a Rake task versus a CRM system or API.

- It would be nice to receive validation error as soon as possible to avoid potential unnecessary calculations. 

## How can we solve this problem? 🤔

Form objects can help address these issues by centralizing data validation in a single place. The following responsibilities from the code above actually belongs to the Form Object:

- <del> We accept only allowed data ( `params.require(:animal).permit(:name)` ) </del>
- <del> We convert data to the specific type  ( `attribute :name, :string` ) </del> 
- <del> We validate the data ( `validates :name, presence: true` ) </del>

 Let's discuss them more closely:

## Responsibilities

- **Enforcing strict attribute definitions** 

>Form objects may have strict attribute definitions, meaning that they only allow specific attributes to be set and accessed. This can help to enforce the structure of the form data and prevent unintended changes.

- **Parsing, transforming, and sanitizing incoming data** 

>Form objects are responsible for parsing, transforming, and sanitizing incoming data from an API request or form submission. This may involve converting data types, formatting data, or performing other data transformations. In addition, form objects may be responsible for sanitizing the incoming data to prevent malicious input or cross-site scripting attacks. By parsing, transforming, and sanitizing the incoming data, form objects can ensure that the data meets the requirements of the application and is safe to use.

- **Validating the data**

>Form objects are responsible for ensuring that the incoming data meets the requirements of the application. This may involve checking for required fields, validating the format of the data, and ensuring that the data is consistent with the business rules of the application.

**Based on these responsibilities let's try to define the key elemets for our new interface**

- For **Parsing, transforming, and sanitizing incoming data**  we need to add `attribute :name, :type` method for each attribute.
- For **Enforcing strict attribute definitions** we need to add `attributes` method. This method will return only allowed attributes with data to be stored in the database.
- For **Validating the data** we need to include `validators` (to set attribute validation rules) and `valid!` method (to validate attributes based on the rules)

Based on the above statement, let's try to build the desired interface.

```rb
class AnimalForm
  attribute :name, :string # prepare attributes

  validates :name, presence: true # set validation rules
  
  def valid!
    # validate attributes
  end

  def attributes
    # returns a hash with all allowed attributes and data, which is ready to be saved in the database
  end
end

form = AnimalForm.new(name: 'Name')
form.valid!

Animal.create(form.attrubutes)
```

This is exactly the interface we expect to see for the Form Object.

**Do we have a ready-made solution with a similar interface?** 🤔

Yes, we have and we'll consider 3 independent ways to create them in Rails. Let's look at these options one by one. 

---

## Plain Ruby class based on ActiveRecord model

Our first option is to use the same method as in the model, but in a separate class to avoid violating the Single Responsibility Principle. For example:

```rb
# app/units/forms/animal.rb

module Forms
  class Animal

    include ActiveModel::Model
    include ActiveModel::Attributes
    include ActiveModel::Validations

    attribute :name, :string

    validates :name, presence: true

  end
end

form = Forms::Animal.new(name: 'Cat')
form.validate! # true
form.attributes # {'name'=>'Cat'}
```

I like this interface, and we don't need to add any external gems, but I've never used this approach in production code, so I don't know if there are any hidden problems 😅

---

## Virtus Form Object

The second option is to use [`Virtus`](https://github.com/solnic/virtus) gem. For example:

```rb
# app/units/forms/animal.rb

module Forms
  class Animal

    include Virtus.model
    include ActiveModel::Validations

    attribute :name, String

    validates :name, presence: true

  end
end

form = Forms::Animal.new(name: 'Cat')
form.validate! # true
form.attributes # {:name=>"Cat"}

```

We've been using this gem in our production code for a long time and it works perfectly fine. However, the biggest drawback is that the gem is no longer supported and has been migrated to `dry-rb`.

---


## Dry-rb Form Object

Our third option to create Form Object is [`dry-rb`](https://dry-rb.org/gems/) gems. To be more precise, we are going to use the [`dry-validation`](https://github.com/dry-rb/dry-validation) gem. Let's take a look at the example below:

```rb
# app/units/forms/animal.rb

module Forms
  class Animal < Dry::Validation::Contract

    params do
      required(:name).filled(:string)
    end

    rule(:name) do
      key.failure('must be present') if value.blank?
    end

  end
end


form = Forms::Animal.new
form.call(name: 'Cat') # validate!
form.call(name: 'Cat').values.data # return attributes '{:name=>"Cat"}'
```

Honestly, I don't like this interface and haven't had enough time to get used to it and fully explore it 😅 However, it is still worth trying as the most modern approach.

---


**The final solution may look like this and be placed under the Service object:**

```rb
def create
  form = Forms::Animal.new(params)
  form.validate!

  animal = Animal.create(form.attributes)

  render json: animal
end
```

P.S. You can read [the following article](https://dev.to/vladhilko/how-to-implement-service-object-pattern-in-ruby-on-rails-2mh8) to understand how `Form` and `Service` objects can work together 

## Conclusion

In conclusion, Form Object is a useful design pattern in Ruby on Rails applications for managing and validating data. The advantages of using form objects are numerous, including the most important ones:

- **Improves code readability and maintainability**:

> Form objects can improve the readability and maintainability of the code by allowing form-related logic to be organized in a single place.

- **Decouples form data from model data**

> Form objects can decouple the form data from the model data, which can make it easier to make changes to the form without affecting the underlying model.

- **Improves testability**

> Form objects can be tested in isolation from the rest of the application, which can make it easier to test the form-related logic and ensure that it is working correctly.

- **Improves security**

> Form objects can improve the security of an application by providing a central place to handle input validation and sanitization.

- **Reusability**

> Allows for reuse of form logic by using form objects in multiple places within the application

- **Extensibility**

> We can create as many Form Objects as we want to cover all possible cases for one model. For example: `Forms::AnimalImport`, `Forms::AnimalCRM` and `Forms::AnimalAPI`.
