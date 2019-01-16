# Using Pundit for authorization in Rails - recipes and best practices

> Original article: https://crypt.codemancers.com/posts/2018-07-29-leveraging-pundit/

Web applications involving user management has two parts to it, which is authentication and authorization. And you don't get to authorization without authentication, as we can't determine what you can do unless we know who you are in the first place.

Hand rolling out user authentication is a tedious task and majority of the Rails community has delegated out authentication to cool gems such as [Devise](https://github.com/plataformatec/devise).

So in this post, we will be talking about another awesome gem which you can leverage to delegate out authorization. And that is [Pundit](https://github.com/varvet/pundit).

## So what is Pundit?

When there arises need for restricting access to your application for certain users, role based authorization comes into play. This is where you can make leverage of Pundit. Pundit helps us to define policies which are PORC - Plain Old Ruby Classes - which means that the class does not inherit from other classes nor include in other modules from the framework. Thus makes it very easy to understand the code.

We would still need to define roles for our Users. But now the advantage is that we get to keep our controllers and models skinny. Policies that you define takes away code complexity from the model/controller which otherwise would have been used to determine access to a particular page. Makes our life easy, don't you think?

## Setting up Pundit

It's very easy to set it up into your application. The [documentation](https://github.com/varvet/pundit#installation) for the gem is well explained.

Nonetheless, let me put it down here:

-   Add `gem 'pundit'` to your `Gemfile`.
-   Within your application controller `include Pundit`.
-   Run command `bundle install`.
-   Optionally run `rails g pundit:install` which will set up an application policy with some useful defaults.

The Policies will be defined in `app/policies/` directory. And don't forget to restart the Rails server so that Rails can pick up new classes that you define there.

## Understanding Policies

Like mentioned earlier, policies are PORC, which houses the authorization for a particular page.

Let's look at a policy class example taken out from the documentation.

```ruby
  class PostPolicy
    attr_reader :user, :post

    def initialize(user, post)
      @user = user
      @post = post
    end

    # CRUD actions
    def update?
      user.admin? or not post.published?
    end
  end
```

This is a policy defined to impose restriction for updating a post if the user is an admin, or if the post is unpublished.

### Characteristics of Policy class

-   The policy name should begin with the name of the model it corresponds to and should always be suffixed with `Policy`. So in the above example - `PostPolicy` would be the policy for `Post`model.
-   The initialize method of the policy would need the instance variable user and the model to be authorized. On a sidenote, we can also get by if the model is simply some other object we want to authorize. For example, say a service or form object which has conditions to be checked on it so as to perform the controller action.
-   The method names should correspond to controller actions suffixed with a `?`. So for controller actions such as `new`, `create`, `edit` etc, the policy methods `new?`, `create?`, `edit?` etc are to be defined

> NOTE: Incase the controller does not have access to current_user method we can define a pundit_user method which will then be used instead.

```ruby
def pundit_user
  User.find_by_other_means
end
```

We can further abstract this Policy if we run the generator `rails g pundit:install`, which creates an Application policy with defaults for controller actions and also takes care of the initialization part. This can be inherited by other policies.

```ruby
class ApplicationPolicy
  attr_reader :user, :record

  def initialize(user, record)
    @user = user
    @record = record
  end

  def index?
    false
  end

  def show?
    false
  end

  def create?
    false
  end

  def new?
    create?
  end

  def update?
    false
  end

  def edit?
    update?
  end

  def destroy?
    false
  end

  class Scope
    attr_reader :user, :scope

    def initialize(user, scope)
      @user = user
      @scope = scope
    end

    def resolve
      scope
    end
  end
end
```

But hold on sec, what is a class `Scope` doing in the generated `ApplicationPolicy?`. And that is what makes Pundit even more awesome, which we will be getting into soon.

With this generated base policy can simpify our `PostPolicy` as

```ruby
  class PostPolicy < ApplicationPolicy
    # Here we are overriding :update? inherited from ApplicationPolicy
    def update?
      user.admin? or not record.published?
    end
  end
```

With this setup in place, let's see what changes at the controller level:

```ruby
class PostController < ApplicationController
  def update
    post = current_user.posts.find(params[:id])
    authorize post
    if post.update(post_params)
      redirect_to post
    else
      render :edit
    end
  end

  # other controller actions
end
```

With this code in plate, update action of the controller when invoked is authorized and the `authorize` method that we invoke here will retrieve the policy for the given record, initialize it with the record and current user and finally throw an error if the user is not authorized to perform the given action.

## Understanding Scopes

Scopes are just like using the scopes you define for a model. But in our case, these scopes are done within the policy in context of the user's role for a particular controller action. Scopes are used to retrieve a subset of the records that we have. For example, in a blog app, a non admin user should be restricted to see only posts which has been published but not in draft state. I see you already imagining the controllers and models becoming thinner.

Let's rework our Post policy:

```ruby
class PostPolicy < ApplicationPolicy
  # Inheriting from the application policy scope generated by the generator
  class Scope < Scope
    def resolve
      if user.admin?
        scope.all
      else
        scope.where(published: true)
      end
    end
  end

  def update?
    user.admin? or not record.published?
  end
end
```

Here we have created a class which will scope the posts based on the user's role. And in order to use it in our controller, we just need to make use of the method `policy_scope`.

### Characteristics of Scope class

-   They are too PORC, which are to be nested within the policy class.
-   It needs to initialize with a user and a scope which can either a be ActiveRecord class or ActiveRecord::Relation.
-   It needs to define a resolve method which scopes based on the user role.

So now, we revise our Post controller's `index` to be like:

```ruby
class PostController < ApplicationController
  def new
    # code to render new view
  end

  def create
    # code to create
  end

  def edit
    # code to render edit
  end

  def update
    post = current_user.posts.find(params[:id])
    authorize post
    if post.update(post_params)
      redirect_to post
    else
      render :edit
    end
  end

  def show
    # code to render show
  end

  def index
    policies = policy_scope(Post)
    # code to render index
  end
end
```

The index action will show only published posts unless the user is an admin.

Good Practices that can be leveraged using pundit

### Keeping authorization explicit

Rather than making authorization or scoping implicit we rather be explict about it. We can add in checks at the `ApplicationController` level so that exception is raised if we forget to add in `authorize` or `policy_scope` in our controller.

```ruby
class ApplicationController < ActionController::Base
  include Pundit
  after_action :verify_authorized, except: :index
  after_action :verify_policy_scoped, only: :index
end
```

But still, we can make use of `skip_authorization` or `skip_policy_scope` in circumstances where you don't want to disable verification for the entire action.

### Keeping a closed system

If we are making use of a Base policy such as `ApplicationPolicy`. We can fail gracefully if at all an unauthenticated user makes through.

```ruby
class ApplicationPolicy
  def initialize(user, record)
    raise Pundit::NotAuthorizedError, "must be logged in" unless user
    @user = user
    @record = record
  end
end
```

### Handling errors on authorization

Since `Pundit::NotAuthorizedError` will be raised if not authorized, we'd need to handle it gracefully. This can be done by making use of `rescue_from` directive for `Pundit::NotAuthorizedError` and then pass in a method to handle the exception.

We can also go a step further and customize error messages based on which policy's action was not authorized.

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery
  include Pundit

  rescue_from Pundit::NotAuthorizedError, with: :user_not_authorized

  private

  def user_not_authorized
    policy_name = exception.policy.class.to_s.underscore

    flash[:error] = t "#{policy_name}.#{exception.query}", scope: "pundit", default: :default
    redirect_to root_path
  end
end
```

And you can have your locale file to be like this:

```yaml
en:
 pundit:
   default: 'You cannot perform this action.'
   post_policy:
     update?: 'You cannot edit this post!'
     create?: 'You cannot create posts!'
```

This is a way to setup error messages for authorization as here we make use of the information `NotAuthorizedError` provide ie. what query (e.g. :create?), what record (e.g. an instance of Post), and what policy (e.g. an instance of PostPolicy) caused the error to be raised. Ultimately, it's up to you on how you organize your locale files. Alternatively, we can also serve them with 403 error page by configuring in `application.rb`

```ruby
config.action_dispatch.rescue_responses["Pundit::NotAuthorizedError"] = :forbidden
```

### Extending policy with multiple roles

Often there comes in requirement that a particular CRUD action's authorization varies on multiple roles. In context of our example, say, there also comes in a role 'premium'. And now there exists posts which can be viewed by premium users and the admin only. No worries, just create a new 'premium' role and update our PostPolicy as below:

```ruby
class PostPolicy < ApplicationPolicy
  # Inheriting from the application policy scope generated by the generator
  class Scope < Scope
    def resolve
      if user.admin?
        scope.all
      elsif user.premium?
        scope.where(published: true)
      else
        scope.where(published: true, premium: false)
      end
    end
  end

  def update?
    user.admin? || !record.published?
  end

  def show?
    return user.premium? || user.admin? if record.premium?
    true
  end
end
```

With the above changes now a normal user can't view premium posts in the index view listings as we are scoping it out and also we are authorizing the show page as to not allow non-premium users to see premium post content. Pretty neat isn't it? We no longer need to delegate the app execution flow to model or controller and let Pundit do all the heavy lifting.

This gives us fine granularity in controlling role based access and now that we understand how Pundit is structured and what conventions we need to follow, writing authorization code becomes intuitive. Skinny controllers and skinny models FTW!

If you have any questions or feedback, feel free to drop us a mail at <team@codemancers.com>.

--- *Akshay Sasidharan*
