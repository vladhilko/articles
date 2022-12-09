A Service object in Rails is the heart of an application, providing a central location for business logic and complex operations. It helps to keep the code organized and maintainable, allowing for easy reusability and testing.

**Why do we need it and what problems can this pattern solve?**

Sometimes we have complex operations and business logic that does not fit well within a model or controller. A Service object helps to keep this code organized and maintainable, solving the problem of cluttering our models and controllers with unnecessary code.

For example, the following code can be used many times in different places:

```rb
def create
  # 'code' to validate authorization logic
  # 'code' to validate params
  article = Article.create(params)
  # 'code' to send notifications to subscribers

  render json: article
end
```

**Problems:**

- Code duplication and violations of the DRY principle
- Unorganized and difficult to maintain
- Violations of the SOLID principles, such as the Single Responsibility principle
- Lack of reusability and testability (impossible to test separately from the controller)
- Cluttered controllers with complex operations and business logic (not clear, hard to read and understand what's going on)

**So what can we do to fix it?**
- We can use model methods to encapsulate specific functionality within a model.
- We can extract complex operations and business logic into a separate Service object.


Let's examine these options:

---

## Model methods

There are two possible ways to implement it on the model level:

- Using callbacks
- By creating explicit methods

---

## Callbacks

Here is an example of using callbacks:

```rb
# frozen_string_literal: true

class Article < ApplicationRecord
  before_create :authorize_user
  after_create :send_notifications
  
  validates :title, :body, presence: true

  private

  def authorize_user
    p 'authorize user'
  end

  def send_notifications
    p 'send notifications'
  end
end

Article.create(params)
```
Callbacks can be a bad pattern because:
- They can make the code difficult to understand and maintain.
- They can make it difficult to follow the flow of the action and understand the sequence of events. 
- Callbacks can be hard to test and modify, and can lead to tight coupling of code, making it more difficult to make changes and modifications. 

That's why, it is generally better to use explicit methods and design patterns instead of callbacks to keep the code organized, maintainable, and testable.

---

## Explicit methods

```rb
# frozen_string_literal: true

class Article < ApplicationRecord
  validates :title, :body, presence: true

  def self.create_article(params)
    article = new(params)

    article.authorize_user
    article.save!
    article.tap { _1.send_notifications }
  end

  def authorize_user
    p 'authorize user'
  end

  def send_notifications
    p 'send notifications'
  end
end

Article.create_article(params)
```

Without proper organization and abstraction, this code can quickly become cluttered and difficult to manage, with a hundred different methods and lines of code.

One possible improvement is to use a Service object to extract the complex operations and business logic into a separate class. 

**How can we do that?**

Meet Service Object Pattern.

In this article we'll consider 2 ways to create Service Object. Here they are:
- Standart Way
- EntryPoint Way

Let's take a look at the examples with each options. 

---

## Standard Service Object

To create a Service object in Ruby, you can create a new class and define the necessary methods for performing the desired operations and business logic. For example, you could create a `ArticleCreatorService` class with a call method that performs the necessary steps for creating an article.

```rb
# app/services/article_creator_service.rb

# frozen_string_literal: true

class ArticleCreatorService

  def initialize(params)
    @params = params
  end

  def call
    authorize_user!
    validate_params!

    article.save!(params)
    article.tap { send_notifications }
  end

  private

  attr_reader :params
  
  def article
    Article.new(params)
  end
  
  def validate_params!
    p 'validate params'
  end

  def authorize_user
    p 'authorize user'
  end

  def send_notifications
    p 'send notifications'
  end
end

ArticleCreatorService.new(params).call
```

The interface looks good, but there are some cases that we can improve:

- We can use inheritance and extract common methods and functionality into a base class. This can help to reduce code duplication and improve maintainability.
- We can split the Service object into two classes with different responsibilities. For example, we could create an `EntryPoint`  class to handle data validation and authorization, and a separate `Action` class containing the actual business logic. This allows for a cleaner separation of concerns and easier testing and modification.

Let's try to change this with our second option - `EntryPoint Service`

---

## EntryPoint Service


Let's take a look at the following examples:

First of all, we need to create `EntryPoint` class. All interaction with this service has to go through this `EntryPoint` because it's the only correct way to call the service object.

**EntryPoint**

```rb
# app/units/create_article/entry_point.rb

# frozen_string_literal: true

module CreateArticle
  class EntryPoint < BaseEntryPoint
    
    def initialize(params:)
      @action = Action.new(params)
    end

  end
end
```

We also need to create a base class for all service object that we're going to use. This class should handle data validation and authorization and call the class with main business logic:

**BaseEntryPoint**

```rb
# lib/base_entry_point.rb

# frozen_string_literal: true

class BaseEntryPoint

  def call
    authorize!
    validate_params!
    action.call
  end

  def self.call(*args, **kwargs)
    new(*args, **kwargs).call
  end

  private

  attr_accessor :action
  
  # TODO: Should be handled via Authorizer pattern (out of scope of this article)
  def authorize!
    p 'authorize!'
  end

  # TODO: Should be handled via Form Object pattern (out of scope of this article)
  def validate_params!
    p 'validate_params!'
  end

end
```

Additionally, we must develop a specific class that contains the actual business logic.

**Action**

```rb
# app/units/create_match/action.rb

# frozen_string_literal: true

module CreateArticle
  class Action

    def initialize(params)
      @params = params
    end

    def call
      # TODO: Should be handled via Command pattern (out of scope of this article)
      article.save!
      article.tap { _1.send_notifications } 
    end

    private

    attr_reader :params

    def article
      Article.new(params)
    end

    # TODO: should be handled via Pub/Sub pattern (out of scope of this article)
    def send_notifications
      p 'send notifications'
    end

  end
end

```

Why do we need it?

- By extracting common methods and functionality into a base class, you can avoid repeating the same code in multiple Service objects. This can save time and effort, and reduce the risk of errors and inconsistencies. Additionally, it can make it easier to modify and maintain the code, by keeping the common methods and functionality in a single, central location. This can improve the overall design and architecture of the code, and make it more flexible and extensible.
- By separating the responsibilities of data validation and authorization from the actual business logic, you can create cleaner and more testable code. 
- It allows you to create subfolders/subactions and organize your code using different patterns inside this folder (`app/units/create_match`)

---


**So the final solution may look as follows:**

```rb
def create
  article = CreateArticle::EntryPoint.call(params)

  render json: article
end
```

## Conclusion

1. It helps to keep the code organized and maintainable, allowing for easy reusability and testing.
2. It prevents the cluttering of models and controllers with unnecessary code, and keeps the codebase clean and readable.
3. It allows for the implementation of design patterns, such as the Decorator pattern, to add additional functionality without cluttering the existing code.
4. It improves the flexibility and extensibility of the code, making it easier to modify and adapt to changing requirements.
5. It allows for the separation of concerns and the creation of logical and testable units, making the code easier to understand and maintain.
6. It can improve the performance and efficiency of the code, by reducing code duplication and unnecessary operations.
7. It can improve the collaboration and teamwork within a project, by providing a clear and consistent structure for organizing and implementing complex operations and business logic.
8. It helps to adhere to the SOLID principles, such as the Single Responsibility principle, and improve the overall design and architecture of the code.
9. It can reduce the risk of errors and bugs, by keeping the code organized and maintainable, and allowing for easy testing and debugging.
