In simple terms, **Dependency Injection** is a way of organizing code that makes it easier to develop, test, and maintain by avoiding hard-coded dependencies. It does this by allowing the dependencies of a particular piece of code to be "injected" from the outside.

**Why do we need it? And what problems can this pattern solve?**

First of all, let's define what is hard-coded dependency. For example, we have 2 classes: `LoggerService` and `Service`.

```rb
# app/services/logger_service.rb

class LoggerService
  def self.call(params)
    puts params
  end
end
```

And

```rb
# app/services/service.rb

class Service
  def call
    LoggerService.call('Something happened')
  end
end

Service.new.call
```

So here, `LoggerService` is a hard-coded dependency inside `Service` class.

**Do we have any problems with hard-coded approach?**

Yes, here they are:

- We violate SOLID dependency inversion principle
- It's pretty difficult to change the code because we have to update the inner methods of the class, and there's a risk of breaking something.
- It will be difficult to read and understand when we start using many hardcoded services inside. For example, hardcoded services could be under several namespaces (`Namespace1::Namespace2::LoggerService, etc.`) so it really increases "cognitive load"). 

Let's see how the following code can be rewritten using dependency injection.

There're 2 approaches that we usually use:
- Plain Ruby Dependency Injection
- Dependency Injection via `dry.rb` gems

## Plain Ruby Dependency Injection

Let's take a look at the following example:

```rb
class Service
  attr_reader :logger_service

  def initialize(logger_service:)
    @logger_service = logger_service
  end

  def call
    logger_service.call('Something happened')
  end
end

# Now you can inject any logger service you want into the Service class
service = Service.new(logger_service: LoggerService)
service.call
```

In this example we simply send our dependency as an additional argument. Thus, this solves the SOLID dependency inversion problem and the maintenance problem.

> ## Dependency Inversion Explanation.
>
>The dependency inversion principle is one of the five SOLID principles of object-oriented design. It states that:
>
> - High-level modules (e.g. classes or components) should not depend on low-level modules. Both should depend on abstractions.
> - Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.
>
> In the context of the code example provided, the Service class could be considered a high-level module, and the LoggerService class could be considered a low-level module. By using dependency injection, the Service class no longer depends on the specific implementation of the LoggerService class, but instead depends on an abstraction (i.e. an object with a call method that accepts a params argument).
>
> This follows the dependency inversion principle, because the high-level Service class depends on an abstraction (the logger_service object) instead of a low-level module (the LoggerService class). It also follows the principle because the details (the concrete implementation of the LoggerService class) depend on the abstraction (the logger_service object), rather than the other way around.
>
> Overall, using dependency injection and following the dependency inversion principle can help to improve the design and architecture of your software, by making it more flexible, maintainable, and testable.


**Is there anything else we can improve here?**

There is another problem that has not been solved by **plain ruby dependency injection**. This is readability and usability. Imagine what happens when we send 5 services as dependencies. The list of arguments and `initialize` method will be huge! Let's consider our second option:

## Dependency Injection via dry.rb gems

Here is an example of how you could rewrite the code using the `dry-container` and `dry-auto_inject` libraries from the `dry.rb` ecosystem:

We need to install the following gems:

```rb
gem 'dry-container'
gem 'dry-auto_inject'
```

Then create a new container class where we should keep all our dependencies.

```rb
# app/dependencies/logger_dependencies.rb

# frozen_string_literal: true

class LoggerContainer

  extend Dry::Container::Mixin

  register 'logger_service' do
    LoggerService
  end

end

LoggerDependencies = Dry::AutoInject LoggerContainer
```

After that we need to do the following:

```rb
class Service
  include LoggerDependencies['logger_service']

  def call
    logger_service.call('Something happened')
  end
end

Service.new.call
```

> ## Explanation.
>
> In this example, the dry-container library is used to define a container that holds the LoggerService object. This container can then be used by the `dry-auto_inject` library to automatically inject the `logger_service` object into the `Service` class when it is instantiated.
>
> This allows you to use the Service class without having to inject the `logger_service` object manually, which makes the code more concise and easier to understand. It also makes it easier to update the LoggerService object used by the Service class, because you only have to update the container, rather than modifying the Service class itself.
>
> Overall, using the `dry-container` and `dry-auto_inject` libraries can help to improve the design and maintainability of your code, by making it easier to use dependency injection and follow the dependency inversion principle.


## Conclusion

That's it. Pretty simple, isn't it? Let's summarize our advantages:

- **Easier to change**
> It makes the code more flexible and easier to change, because the Service class is no longer tied to a specific implementation of the LoggerService class. 

- **Easier to test**
> It allows you to write unit tests for the Service class without having to instantiate a real LoggerService object. Instead, you can use a mock or stub object that simulates the behavior of the LoggerService class.

- **Easier to understand**
> It makes the code easier to understand, because the Service class doesn't have to include any logic for creating or managing the LoggerService object. This makes the code more focused and modular.

- **Easier to customize**
> It allows you to customize the behavior of the Service class by injecting different implementations of the LoggerService class. For example, you could inject a LoggerService class that logs messages to a file, a database, or a remote server, depending on your needs.

- **Easier to maintain**
> It makes the code more maintainable, because you can easily update the LoggerService object used by the Service class without having to modify the Service class itself.

- **More scalable**
> It makes the code more scalable, because you can easily add new features to the Service class by injecting new objects or services that it can use. This allows the Service class to evolve and grow over time without becoming overly complex or difficult to manage.
