## Overview

> In this article, we will explore the `Authorizer` pattern in Ruby on Rails applications. We will provide a definition of the `Authorizer` pattern and discuss its use cases and examples. We will also delve into the need for this pattern and how it helps to solve certain problems in Rails applications.

## Definition

In simple terms, the **Authorizer** pattern allows us to define and encapsulate complex business rule in our Rails applications. If the requirement specified by this rule is not met, an error will be raised. You can think of an `Authorizer` as a custom `ActiveRecord Validator`, but instead of validating input data, it is used to enforce business requirements.

**Why do we need it and what problems can this pattern solve? 🤷🏻‍♂️**

Let's take a look at the following example.

```rb
def create
  user = User.first
  raise 'User is not active' if user.status != 'active' || user.approved_at.blank?
  
  article = Aricle.create(user: user, title: 'title', body: 'body')
  render json: article
end
```

In the following example, we want to ensure that certain requirements are fully met before calling the business logic. 

**Do we have any problems with this approach?** 🤔

Yes, we have.

- It is not possible to test separately from the controller.
- It is not reusable.
- We violate the Single Responsibility Principle because the controller becomes responsible for business rules validation.
- The following code increases 'cognitive load' and requires more time to understand.
- The following code is not scalable and it is difficult to add more rules.

## How can we solve these problems? 🤔

- We can move this logic to the model level and use `before_create` callback.
- We can create `Athorizer` class and move the logic there.

Let's take a look at these options one by one. 

---

## Model with callback

This is the easiest option, we just need to extend our model in the following way:

```rb
# app/models/article.rb

class Aricle < ApplicationRecord

  before_save :check_user_status!
  
  def check_user_status!
    raise 'User is not active' if user.status != 'active' || user.approved_at.blank?
  end
  
end

user = User.last
Aricle.create(user: user, article_params)
```

**Are there any disadvantages to this approach?**

There're 2 problems:
- We violate the Single responsibility SOLID principle. The model becomes too `FAT` and responsible for too many things. 
- Using callbacks in Rails models can make the code more difficult to understand and maintain due to their tight coupling with the model and lack of transparency. Therefore, it is generally considered to be a bad pattern.

So it would be nice to encapsulate this logic in a separate class. **How can we do that?**

---

## Authorizer Pattern

We're going to implement Athorizer Pattern via plain Ruby object because it's the clearest and the most straightforward solution.

Let's take a look at the following example.

```rb
# app/authorizers/has_active_user_authorizer.rb

class HasActiveUserAuthorizer

  def initialize(user)
    @user = user
  end

  def authorize!
    raise 'User is not active' unless active?
  end

  private

  attr_reader :user

  def active?
    user.status == 'active' && user.approved_at.present?
  end

end

HasActiveUserAuthorizer.new(user).authorize!
```

We just encapsulated our rules inside a separate class, and that's it. This code adheres to the SOLID principles and is much easier to understand and maintain.

## Naming convention

There are two common ways to name authorizers:
- Inside a namespace (for example, `Authorizers::HasActiveUser`)
- With a postfix (for example, `HasActiveUserAuthorizer`)

But they have one rule in common - the authorizer must clearly reflect in its name the rule it introduces. Let's take a look at other possible names that are commonly used for the Authorizer Pattern to get a wider picture:

- **CompanyHasNoEmployeesAuthorizer**

> Raises an error if the company has at least one employee 

- **ArticleNotApprovedAuthorizer**

> Raises an error if the article is approved

- **Authorizers::HasAtLeastOneComment**

> Raises an error if there are no comments

- **Authorizers::NotPopulated**

> Raises an error if the table is already populated with records

- **UserHasActiveSubscriptionAuthorizer**

> Raises an error if the user does not have an active subscription

- **CustomerAgeApprovedAuthorizer**

> Raises an error if the customer's age is not approved

etc ...

## Comparing the Authorizer Pattern to Other Patterns

### **Similar to Form Object**

Form objects and Authorizers both perform validation and raise errors before running business logic. So, what are the differences? The key differences are:

- Form objects validate data from the user input
- Authorizers validate general business logic rules

**P.S.**. To learn more about using Form object pattern in Ruby on Rails, you can check out this [article](https://dev.to/vladhilko/how-to-implement-form-object-pattern-in-ruby-on-rails-5gi3).

### **Similar to Policy Object**

Authorizers are also similar to Policy objects because both encapsulate business rules. The key differences are:

- Authorizers may contain only one business rule, but Policy objects can contain many.
- Authorizers are more strict and demand that a business rule be followed, whereas a Policy object will only ask if the rule is true or false. This difference is reflected in the use of `!` by Authorizers and `?` by Policy objects at the end of their methods.

**P.S.**. To learn more about using Policy object pattern in Ruby on Rails, you can check out this [article](https://dev.to/vladhilko/how-to-implement-policy-object-pattern-in-ruby-on-rails-54cb).
 

### **Similar to Custom Validators**

- Authorizers are similar to custom ActiveRecord validations in terms of their responsibility, but custom validators validate one data field, while Authorizers validate general rules for the whole model.

### **Note:** 

> Don't confuse the Authorizer pattern with authorization logic. Authorizers ask if the service is authorized to perform the action under the current conditions, while authorization logic happens at a higher level (usually on the controller level).


## Bonus Chapter: Adding Authorizers to Service Objects

In the previous chapter, we mentioned that Authorizers are similar to custom validators. Let's take a look at the example of custom validators:

```rb
# app/models/article.rb

class Article < ApplicationRecord
  # ...
  validate :name_presence
  # ...
  def name_presence
    errors.add :base, "Name is empty" if name.blank?
  end
end

```

Or in a separate class:

```rb
# app/models/article.rb

class Article < ApplicationRecord
  # ...
  validates_with NamePresenceValidator
  # ...
end

# app/validators/name_presence.rb

class NamePresenceValidator < ActiveModel::Validator
  def validate(record)
    record.errors.add :base, "Name is empty" if record.name.blank?
  end
end

```

Essentially, we want to have a similar interface for our Authorizers within Service objects, something like this:

```rb
# app/services/create_article_service.rb

class CreateArticleService
  # ...
  authorize 'has_active_user_authorizer'
  authorize 'another_authorizer'
  authorize 'one_more_authorizer'
  # ...
end
```

How can we do this? Let's consider what we need:

- Service (to keep the `authorize` method)
- A module with the `authorize` method definition (to be able to include it in every service)
- Authorizer (to encapsulate complex business rule)
- A base Authorizer (to avoid repeating the same code for every new Authorizer)

Let's add them one by one.

 **Note**: We will only provide code snippets without explanation to keep it brief.

## Service

```rb
# app/services/create_article_service.rb

class CreateArticleService

  include Authorizable

  authorize 'has_active_user_authorizer'

  attr_reader :user, :params

  def initialize(user: , params:)
    @user = user
    @params = params
  end

  def call
    authorize!
    # main logic
  end

end
```

## Authorizable module

```rb
# lib/authorizable.rb

module Authorizable
  extend ActiveSupport::Concern

  included do
    class_attribute :authorizers, default: []
  end

  class_methods do
    def authorize(authorizer_name)
      self.authorizers += [authorizer_name]
    end
  end

  def authorize!
    authorizers.each do |authorizer|
      authorizer.to_s.classify.constantize.new(self).authorize
    end
  end
end
```

## Authorizer

```rb
# app/authorizers/has_active_user_authorizer.rb

class HasActiveUserAuthorizer < BaseAuthorizer

  def authorize!
    raise 'User is not active' unless active?
  end

  private

  def active?
    service_object.user.status == 'active' && service_object.user.approved_at.present?
  end

end
```

## BaseAuthorizer

```rb
# lib/base_authorizer.rb

class BaseAuthorizer

  def initialize(service_object)
    @service_object = service_object
  end

  def authorize!
    raise 'NotImplementedError'
  end

  private

  attr_reader :service_object

end

```

That's it! Now you can use the Authorizer pattern in your service objects to encapsulate complex business rules and ensure that they are followed before the main logic is executed. To learn more about using the Service object pattern in Ruby on Rails, you can check out my other [article](https://dev.to/vladhilko/how-to-implement-service-object-pattern-in-ruby-on-rails-2mh8).


**So the final solution may look like this:**

```rb
def create
  user = User.first
  UserIsActiveAuthorizer.new(user).authorize!
  
  article = Aricle.create(user: user, title: 'title', body: 'body')
  render json: article
end
```

Or using a service object:

```rb
def create
  user = User.first

  article = CreateArticleService.new(user: user, params: { title: 'title', body: 'body' }).call
  render json: article
end
```

## Conclusion
- The Authorizer Pattern helps us better separate logic, making it easier to test our code.
- It is DRY and reusable.
- The Authorizer Pattern allows us to avoid having a "fat" model and adhere to SOLID principles.
- It reduces "cognitive overhead", making the code easier to understand.
- It ensures that any necessary requirements are met as soon as possible by raising an error before running any form validations or business logic calculations.
