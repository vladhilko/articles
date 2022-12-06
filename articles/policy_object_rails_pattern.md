In simple terms, **Policy Object** allows us to encapsulate complex business rules and is used to replace complex conditions.

**Why do we need it and what problems can this pattern solve?**

Sometimes we have complex business rules that are used directly in the business logic. For example, the following code can be used many times in different controllers and service objects:

```rb
def show
  user = User.first

  if user.status == 'active' && user.approved_at.present?
    render json: user
  else
    render status: :forbidden
  end
end
```

**Problems:**

- **NOT** possible to test separately from the controller.
- **NOT** DRY
- Violates single-responsibility principle
- **NOT** clear, hard to read and understand what's going on.

**So what can we do to fix it?**
- We can move this logic to the model level.
- We can create a Policy Object class and move the logic there.

Let's take a look at these options one by one. 

---

**Model methods** 

Adding new model methods is the easiest option, we just need to extend our model in the following way:

```rb
# frozen_string_literal: true

class User < ApplicationRecord

  def active?
    status == 'active' && approved_at.present?
  end

  def cancelled?
    status == 'cancelled' && cancelled_at.present?
  end
  
end

user = User.last
user.active?
user.cancelled?
```

**Why do we need other options if the model methods solve our problems?**

There're 2 problems:
- The first one is that we violate the Single responsibility and the open-closed SOLID principles (*'a class should basically serve one purpose'* and *'a class/object should be open for extension, but closed for modification'*).
The model becomes too `FAT` and responsible for too many things. 
In real projects, we'll end up with a hundred different methods and lines of codes, because one such method can contain 10 private methods under the hood.
You can probably imagine how difficult it would be to read, modify and maintain this class. 
- The second problem here is potential additional arguments. Right now we only use the `user` object, but in the future the business rules may require other objects as well, so it won't be easy to maintain such code. 


That's why it would be nice to have the same interface, but move all the methods into a separate class.

--- 

**How can we do that?**

---

**Meet Policy Object**

We're going to implement Policy Object via plain Ruby object because it's the clearest and the most straightforward solution.

Let's take a look at the following example.

```rb
# app/units/policies/user.rb

# frozen_string_literal: true

module Policies
  class User

    def initialize(user)
      @user = user
    end

    def active?
      user.status == 'active' && user.approved_at.present?
    end

    def cancelled?
      user.status == 'cancelled' && user.cancelled_at.present?
    end

    private

    attr_reader :user

  end
end

user_policy = Policies::User.new(User.last)
user_policy.active?
user_policy.cancelled?
```

We just encapsulate our rules inside a separated class and that's it. The following code stick to SOLID principle and can take as many arguments as we want. Pretty simple, isn't it? Let's take a look at other possible names that are commonly used inside policy objects to get a wider picture:


```rb
def apply?; end

def allowed?; end

def permitted?; end

def eligible?; end

def satisfy?; end

def excluded?; end

def locked?; end

def approved?; end

def not_approved?; end

def required?; end

# etc...
```

Have you figured out what all these Policy Object methods have in common yet? Here they are:

- Policy object methods always return boolean values (`true` or `false`)
- Policy object methods always end with a `?`
- Policy objects must not have any side-effects

**So the final solution may look as follows:**

```rb
def show
  user = User.first

  if Policies::User.new(user).active?
    render json: user
  else
    render status: :forbidden
  end
end
```

## Conclusion

- Thanks to Policy Object we can separate logic better and test our code more easily.
- Policy Object is DRY and reusable
- Policy Object allows us to avoid FAT model and stick to SOLID principles
- Policy Object reduces "Cognitive Overhead", so the code is much easier to understand.
