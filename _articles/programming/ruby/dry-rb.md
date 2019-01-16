# Designing Services with dry-rb

> Original article: <https://medium.com/adhawk-engineering/designing-services-with-dry-rb-fe850f8dd4b7>

In a traditional Ruby on Rails application, models and controllers can become bloated as they take on additional responsibilities. It is not uncommon for controller actions to contain complex code for initializing models or for models to contain callbacks that send email or class methods that define intricate queries. Extracting services is one way to keep classes lean and separate business logic from framework code.

Some frameworks have built-in support for services. In Trailblazer, they are called "operations." In Hanami, they are called "interactors." In the [DCI](https://en.wikipedia.org/wiki/Data,_context_and_interaction)paradigm, they are similarly called "interactions."

I prefer to treat services like method objects. I name them after the actions they represent and invoke their behavior via a `call` method. Does thinking of services as methods change the way we design them?

In his book [Confident Ruby](https://amzn.to/2LEOGcF), Avdi Grimm lists the four responsibilities of a method:

1.  Gather input
2.  Perform work
3.  Deliver results
4.  Handle failure

This article explores how to fulfill these responsibilities within a service while using modern Ruby libraries like dry-rb to provide a consistent, testable design across a service library.

---

Let's start with a simple service that may be useful in a real-world application --- a service that posts a message to Slack.

```ruby
class NotifySlack
  def self.call(message:)
    notifier = Slack::Notifier.new(SLACK_WEBHOOK_URL)
    notifier.ping(message)
  end
end

NotifySlack.call(message: 'Hello World!')
```

### 1 Gathering Input

The first responsibility of a method is to gather input. This brings up two concerns: how we pass the input into our object and how we validate the input before executing any operations.

#### Dependency Injection

Dependency injection (DI) simply means passing dependencies into an object or code block instead of instantiating them within it. Using DI makes code more flexible and easier to test because we can swap out the dependency for another object provided that it responds to the messages that we send it.

`Dry::Initializer` provides the ability to configure the parameters for instantiating an object. If we modify our service to work with an instance, we can define `message` as a required parameter and give `notifier` a default value. Note that we still provide a `call` method on the class to maintain the same interface.

```ruby
class NotifySlack
  extend Dry::Initializer
  option :message
  option :notifier, default: -> { Slack::Notifier.new(URL) }

  def self.call(**args)
    new(**args).call
  end

  def call
    notifier.ping(message)
  end
end
```

#### Input Validation

Validating the input to a service can help avoid unwanted exceptions and provide meaningful feedback to the caller.

`Dry::Validation` provides the ability to define schemas and use them to validate a hash of values. We can use it to return `false` if the message is empty instead of posting an empty message to Slack.

```ruby
class NotifySlack
  extend Dry::Initializer
  option :message
  option :notifier, default: -> { Slack::Notifier.new(URL) }

  Schema = Dry::Validation.Schema do
    required(:message).filled
  end

  def self.call(**args)
    return false unless Schema.call(args).success?

    new(**args).call
  end

  def call
    notifier.ping(message)
  end
end
```

#### More Services

Suppose we add an additional service that sends an email notification. It may look very similar to our Slack notification service.

```ruby
class SendEmail
  extend Dry::Initializer
  option :to
  option :body
  option :emailer, default: -> { MyMailer }

  Schema = Dry::Validation.Schema do
    required(:to).value(format?: URI::MailTo::EMAIL_REGEXP)
    required(:body).filled
  end

  def self.call(**args)
    return false unless Schema.call(args).success?

    new(**args).call
  end

  def call
    emailer.send(to, body)
  end
end
```

Both services extend `Dry::Initializer` and the `call` class method is exactly the same in both services. We can extract a parent class or mixin that provides the behavior shared across both services. I'm using inheritance. However, another approach may make more sense in your project. Here's how the new parent service class looks:

```ruby
class Service
  extend Dry::Initializer

  def self.call(**args)
    return false unless self::Schema.call(args).success?

    new(**args).call
  end
end```

This allows us to cleanup our slack and email services:

```ruby
class NotifySlack < Service
  option :message
  option :notifier, default: -> { Slack::Notifier.new(URL) }

  Schema = Dry::Validation.Schema do
    required(:message).filled
  end

  def call
    notifier.ping(message)
  end
end

class SendEmail < Service
  option :to
  option :body
  option :emailer, default: -> { MyMailer }

  Schema = Dry::Validation.Schema do
    required(:to).value(format?: URI::MailTo::EMAIL_REGEXP)
    required(:body).filled
  end

  def call
    emailer.send(to, body)
  end
end
```

### 2. Perform Work

There's not much to say here beyond having on a single public `call` class method and adhering to good design principles (i.e. DRY, SRP, etc.) when building out the inner workings of a more complex service.

### 3. Deliver Results

We're returning `false` if our arguments are invalid. Otherwise, we're simply returning the result of the slack or email dependency. It would be nice to have a consistent interface across the results of all of our services.

`Dry::Monads` provides `Success` and `Failure` objects that can wrap the result of our service. First, we can return a `Failure` if the inputs are invalid.

```ruby
validation = self::Schema.call(args)
return Failure.new(validation.errors) unless validation.success?
```

We can also return a`Success` or `Failure` from each service's `call` method.

```ruby
email_result = emailer.send(to, body)

if email_result
  Success.new(email_result)
else
  Failure.new('email error')
end
```

If these services were being used in a Rails application, the controller action could respond differently based on the result.

```ruby
if NotifySlack.call(message: 'Hello World!').success?
  redirect_to some_path, notice: 'Success!'
else
  flash.now[:alert] = 'Uh oh!'
  render :new
end
```

### 4. Handle Failure

The final responsibility of a method is to handle failure. There's an interesting problem that we face when working with services --- especially those that depend on remote APIs. In certain cases, we consider some exceptions to be permissible and want to return a failure result. In other cases, we want our application to crash or the exception to be tracked. Adding a way for each service to specify which errors are permissible will make our service objects even more powerful.

`Dry::Core::ClassAttributes` provides a `defines` method to explicitly list the attributes that can be defined within a class. We can use it to define the permissible errors. Then, if a permissible exception occurs, we can simply return a `Failure` instead of crashing the app.

```ruby
class Service
  extend Dry::Initializer
  include Dry::Monads::Result::Mixin
  extend Dry::Core::ClassAttributes

  defines :permissible_errors
  permissible_errors []

  def self.call(**args)
    validation = self::Schema.call(args)
    return Failure.new(validation.errors) unless validation.success?

    new(**args).call
  rescue StandardError => error
    handle_error(error)
  end

  def self.handle_error(error)
    raise error unless permissible_errors.any? { |type|
      type === error }

    Failure.new(error)
  end
end
```

We can override the permissible error list in an individual service if there are any expected errors that should be allowed.

```ruby
class NotifySlack < Service
  option :message
  option :notifier, default: -> { Slack::Notifier.new(URL) }
  permissible_errors [Slack::SlackError]

  Schema = Dry::Validation.Schema do
    required(:message).filled
  end

  def call
    notifier.ping(message)

    Success.new(true)
  end
end
```

### Wrap-up

That covers applying the four responsibilities of methods to services. I'm not advocating using this `Service` class in a real world application. However, I think some of the libraries used in this post can help you create clean, flexible, testable services in your own applications.

All of the code in this article can be found at <https://github.com/psparrow/designing-services-with-dry-rb>.

### Further Reading

Rob Race has written some great articles about service objects at his [blog](https://hackernoon.com/@rob__race). The services used in this article are loosely-based on his examples.

--- *Patrick J. Sparrow*
