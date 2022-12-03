Decorator design pattern allows us to add new functionality to an object without affecting the behavior of the original class.

**Why do we need it and what problems can this pattern solve?**

Sometimes we have complex logic that is used directly in the business logic. For example, the following code can be used many times in different controllers and service objects:

```rb
def show
  user = User.first
  full_name = "#{user.name} #{user.surname}"
  full_address = "#{user.country} #{user.city} #{user.street}"
  
  render json: { name: full_name, address: full_address }
end
```

**Problems:**

- It's impossible to test separately from the controller.
- It's not DRY
- It's violates single-responsibility principle
- It's not clear, hard to read and understand what's going on.

**So what can we do to fix it?**
- We can move this logic to the model level.
- We can create a Decorator class and move the logic there.

Let's look at these options one by one. 

---

**Model methods** 

Adding new model methods is the easiest option, we just need to extend our model in the following way:

```rb
# frozen_string_literal: true

class User < ApplicationRecord

  def full_name
    "#{name} #{surname}"
  end
  
  def full_address
    "#{country} #{city} #{street}"
  end
  
end

user = User.new
user.full_name
user.full_address
```

**Why do we need other options if the model methods solve our problems?**

The problem here is that we violate the Single responsibility and the open-closed SOLID principles (*'a class should basically serve one purpose'* and *'a class/object should be open for extension, but closed for modification'*). The model becomes too `FAT` and responsible for too many things. In real projects, we'll end up with a hundred different methods and lines of codes. You can probably imagine how difficult it would be to read, modify and maintain this class. That's why it would be nice to have the same interface, but move all the methods into a separate class.

**How can we do that?**

Meet Decorator Pattern.

In this article we'll consider 3 ways to create Decorator from the simplest to the most complex, here they are:
- Adding Decorator via Plain Ruby
- Adding Decorator via SimpleDelegator
- Adding Decorator via Draper gem

Let's take a look at the examples with each options. 

---

## Plain Ruby Decorator

Plain Ruby Decorator is just a class that accepts a model object and returns a new object with all desired methods. For example:

```rb
# app/units/decorators/user.rb

# frozen_string_literal: true

module Decorators
  class User

    attr_reader :user

    def initialize(user)
      @user = user
    end

    def full_name
      "#{user.name} #{user.surname}"
    end

    def full_address
      "#{user.country} #{user.city} #{user.street}"
    end

  end
end

decorated_user = Decorators::User.new(User.last)
decorated_user.full_name
decorated_user.full_address
```

The interface looks good, but we have to repeat `user` every time inside the decorator. Let's try to change this with our second option - `SimpleDelegator Decorator`

---

## SimpleDelegator Decorator

[SimpleDelegator](https://ruby-doc.org/stdlib-2.5.1/libdoc/delegate/rdoc/SimpleDelegator.html) Decorator is the same as a Plain Ruby Decorator, but with only one difference - we delegate all supported method calls to the object passed into the constructor. Let's take a look at the following example: 

```rb
# app/units/decorators/user.rb

# frozen_string_literal: true

module Decorators
  class User < SimpleDelegator

    def full_name
      "#{name} #{surname}"
    end

    def full_address
      "#{country} #{city} #{street}"
    end

  end
end

decorated_user = Decorators::User.new(User.last)
decorated_user.full_name
decorated_user.full_address
```

With SimpleDelagator our solution looks much clearer and more elegant. This approach solves 95% of the common cases and I really like it because of its simplicity, but if you need something more complex, here is our last option, `draper` decorator.

---

## Draper Decorator

[Draper](https://github.com/drapergem/draper) is the most popular gem to implement Decorator pattern for Rails. With this gem our example would look like this:

```rb
# app/units/decorators/user.rb

# frozen_string_literal: true

module Decorators
  class User < Draper::Decorator

    delegate_all

    def full_name
      "#{name} #{surname}"
    end

    def full_address
      "#{country} #{city} #{street}"
    end

  end
end

decorated_user = Decorators::User.new(User.last)
decorated_user.full_name
decorated_user.full_address
```

You can read more about draper [here](https://github.com/drapergem/draper) and decide for yourself whether it makes sense to add it or not.

**So the final solution may look as follows:**

```rb
def show
  user = Decorators::User.new(User.first)
  
  render json: { name: user.full_name, address: user.full_address }
end
```

## Conclusion

- Decorator can be easily tested separately from the controller
- Decorator is DRY and reusable
- Decorator allows us to avoid FAT model and stick to SOLID principles
- Decorator reduces chances of breaking existing logic and increases product stability
- Decorator reduces maintenance cost
