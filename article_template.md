Definition 

**Why do we need it and what problems can this pattern solve?**

Sometimes we have complex logic that is used directly in the business logic. For example, the following code can be used many times in different controllers and service objects:

```rb
def show
  user = User.first

  render json: user
end
```

**Problems:**

- It's impossible to test separately from the controller.
- It's not DRY
- It's violates single-responsibility principle
- It's not clear, hard to read and understand what's going on.

**So what can we do to fix it?**
- We can move this logic to the model level.
- We can create a 'Pattern' class and move the logic there.

Let's look at these options one by one. 

---

**Model methods** 

Adding new model methods is the easiest option, we just need to extend our model in the following way:

```rb
# frozen_string_literal: true

class User < ApplicationRecord  
end

user = User.new
user.name
```

**Why do we need other options if the model methods solve our problems?**

The problem here is that we violate the Single responsibility and the open-closed SOLID principles (*'a class should basically serve one purpose'* and *'a class/object should be open for extension, but closed for modification'*). The model becomes too `FAT` and responsible for too many things. In real projects, we'll end up with a hundred different methods and lines of codes. You can probably imagine how difficult it would be to read, modify and maintain this class. That's why it would be nice to have the same interface, but move all the methods into a separate class.

**How can we do that?**

Meet Pattern.

In this article we'll consider 3 ways to create 'PATTERN' from the simplest to the most complex, here they are:
- Option 1 
- Option 2
- Option 3

Let's take a look at the examples with each options. 

---

## Option 1

Option 1 explanation. For example:

```rb
# path to the file

# frozen_string_literal: true

module Pattern
  class User

  end
end

user = Pattern::User.new(User.last)
```

The interface looks good, but EXPLAIN PROBLEM. Let's try to change this with our second option - `Option 2`

---

## Option 2

Option 2 explanation. Let's take a look at the following example: 

```rb
# path to the file

# frozen_string_literal: true

module Pattern
  class User

  end
end

user = Pattern::User.new(User.last)
```

Explain why option 2 is useful. 

---


**So the final solution may look as follows:**

```rb
def show
  user = Pattern::User.new(User.first)
  
  render json: { name: user.name }
end
```

## Conclusion

- Pattern can be easily tested separately from the controller
- Pattern is DRY and reusable
- Pattern allows us to avoid FAT model and stick to SOLID principles
- Pattern reduces chances of breaking existing logic and increases product stability
- Pattern reduces maintenance cost
