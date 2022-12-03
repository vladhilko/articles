In this article we'll discuss 8 design principles that every Ruby developer should know. We'll go through each principle, giving a definition, a simple example, and the benefits they give us.

**So, what are they?**

- DRY
- KISS
- YAGNI
- SOLID
  - Single-responsibility principle
  - Open-closed principle
  - Liskov substitution principle
  - Interface segregation principle
  - Dependency inversion principle

Let's go through them one by one.

---

### DRY

DRY (don’t repeat yourself) means don’t write duplicate code, instead use Abstraction to abstract common things in one place.

For example, if we had the following code:

```rb
p 'hello'  # any complex logic 
p 'hello'  # any complex logic 
p 'hello'  # any complex logic 
p 'hello'  # any complex logic 
p 'hello'  # any complex logic 
p 'hello'  # any complex logic 
p 'hello'  # any complex logic 
```

Then it would make sense to encapsulate it in a new method

```rb
def hello
  p 'hello' # any complex logic 
end

hello
hello
hello
hello
```

**Why do we need it?**

- Easier to Read/Understand
- Easier to Change/Maintain
- Easier to Test
- Easier to Reuse

---

### KISS

Keep it simple, stupid (KISS) is a design principle which states that designs and/or systems should be as simple as possible.

**Why do we need it?**

- Easier to read, understand and maintain
- Chances of producing a bug becomes less
- Code review takes less time

---

### YAGNI

YAGNI stands for You aren't gonna need it. This principle means you should implement only required functionalities.

**Why do we need it?**

- Saves your time and company money
- Avoids supporting *DEAD* code and simplifies refactoring

---

## SOLID

SOLID is an acronym for five separate object-oriented design principles:

- The single-responsibility principle
- The open-closed principle
- The Liskov substitution principle
- The interface segregation principle
- The dependency inversion principle

---

###  The single-responsibility principle

The basic idea of the single-responsibility principle is that a class should basically serve one purpose. As a commonly used definition, "every class should have only one reason to change".

Let's take a look at the simplest example:

```rb
def hello
  calculation = 1 + 1 + 3 # any complex logic 

  puts calculation # any complex output 
end

hello
```

The code above violate Single Responsibility principle because it does 2 things at the same time ( Calculation and Printing). To fix this, we need to do the following:

```rb
def calculation # complex logic encapsulated
  1 + 1 + 3 
end

def print_calculation # complex output encapsulated
  puts calculation
end

print_calculation
```
Now we have 2 methods, each with only one purpose and reason for the change.

**Why do we need it?**

- Easier to Understand
- Easier to Test
- Easier to Maintain, Refactor or Replace
- Code becomes reusable

---

### The open-closed principle

The open-closed principle states that a class/object should be open for extension, but closed for modification. It means that if it is necessary to change a specific class because of new business requirements, it is better to create a new code for the changes or new functionality instead of affecting the existing one in the cases where it is possible. Open/closed principle is intended to mitigate risk when introducing new functionality.

For example if we want to change `p 'hello'` in the following code:

```rb
class Animal
  def hello
    p 'hello'
  end
end

Animal.new.hello
```

It would be better to create a separate class, inherit everything from the base class, and change `p 'hello'` there:

```rb
class AnimalExtension < Animal
  def hello
    p 'Hello World!'
  end
end

AnimalExtension.new.hello
```

**Why do we need it?**

- Reduces chances of breaking existing logic and increases product stability
- It reduces maintenance cost

---

### The Liskov substitution principle

The Liskov substitution principle states that any place in the code where you can use an object of type T, you can also use an object of a subtype of T. In terms of Ruby, this means that any place in your code where you are using an instance of a class, you can also use an instance of a subclass without anything breaking.

Let's take a look at the following example:

```rb
class Animal
  def hello
    p 'Hello'
  end 
end

animal = Animal.new
animal.hello 
```

We expect that the above method will always print something. To break this principle, we just need to create a subclass that does something else, like:

```rb
class Cat < Animal
  def hello
    return true
  end
end
```

Or 

```rb
class Dog < Animal
  def hello
    raise 'error'
  end
end
```

And instead of using

```rb
animal = Animal.new
animal.hello 
```
We start using subclass instance

```rb
animal = Cat.new
animal.hello 
```

If our code doesn't print anything we're violating **The Liskov substitution principle**.

**Why do we need it?**

- The code becomes predictable, so the cost of maintenance is reduced

---

### The interface segregation principle

The interface segregation principle states that clients should not be forced to depend on methods they do not use. The goal of this principle is to reduce the side effects of using larger interfaces by breaking application interfaces into smaller ones.

For example, we have the following module:

```rb
module ManageExtensions
  def create
    p 'created'
  end 

  def destroy
    p 'destroyed'
  end
end

class CreateAnimal
  include ManageExtensions
end

class DestroyAnimal
  include ManageExtensions
end

CreateAnimal.new.create
DestroyAnimal.new.destroy
```

This module contains too many methods that we don't use in the classes. To apply **interface segregation principle** we have to rewrite the module as follows:

```rb
module CreateExtensions
  def create
    p 'created'
  end 
end

module DestroyExtensions
  def destroy
    p 'destroyed'
  end
end

class CreateAnimal
  include CreateExtensions
end

class DestroyAnimal
  include DestroyExtensions
end

```

Now the class only has access to the methods it is supposed to use. 

**Why do we need it?**

- Reduces unexpected bugs when the Class does not have the ability to perform an action
- Better code organization
- Easier to Understand and maintain

---

### DEPENDENCY INVERSION

The dependency inversion principle states that high-level modules should not depend on low-level modules, and both high-level and low-level modules should depend on abstractions. It also states that abstractions should not depend on concrete implementations, but that concrete implementations should depend on abstractions.

For example, we have the following class that just print something:

```rb
class Animal
  def hello
    p 'Hello'
  end
end

Animal.new.hello
```

But sometimes we want to change the print method for example from ` p 'Hello'` to `puts 'Hello'` or `print 'Hello'`. 

**What should we do according to DEPENDENCY INVERSION principle?**

We should encapsulate `puts` and `print` method into abstractions:

```rb
class Puts
  def show(text)
    puts text
  end
end

class Print
  def show(text)
    print text
  end
end
```

And then pass this class as a dependency

```rb
class Animal
  attr_reader :printer

  def initialize(printer)
    @printer = printer
  end

  def hello
    printer.show('Hello')
  end
end

Animal.new(Puts.new).hello
```

**Why do we need it?**

- Keeps your code loosely coupled
- Code becomes reusable and DRY

### Conclusion

 We've just covered the 8 most important Ruby Design Principles. I hope this will help you to avoid serious mistakes during writing complex programs.
