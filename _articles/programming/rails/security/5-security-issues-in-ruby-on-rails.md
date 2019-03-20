# 5 security issues in Ruby on Rails apps from real life

> Original article: <https://frontdeveloper.pl/2018/10/5-security-issues-in-ruby-on-rails/>

...and how to fix them 🙂

I have had several opportunities to find and fix various security issues within Ruby on Rails applications over time. Based on my own experience I would like to help you with making Rails apps more secure. At the same time, I hope that you won't find any of the below issues in any of your apps.

If you want to have a better understanding of built-in Ruby on Rails security mechanism you can take a look at the official *[Securing Rails Applications](https://guides.rubyonrails.org/security.html)*guide.

## 1. Lack of session expiration mechanism

### Security issue description

According to the *Securing Rails Applications* guide:

> Sessions that never expire extend the time-frame for attacks such as cross-site request forgery (CSRF), session hijacking and session fixation.

Well, even though an infinite session duration seems to be a right approach from user experience perspective (because user stays signed in forever, so she doesn't have to sign in every time she visits your application) it's a bad idea. To prevent situations when somebody hijacks user's session because she forgot to sign out using a computer in a public library, the session should expire as soon a possible.

### Solution(s)

The simplest solution is to set an expiration timestamp of the session cookie within the config/initializers/session_store.rb initializer. The below line:

```ruby
Rails.application.config.session_store  :cookie_store, expire_after: 12.hours
```

would set the session cookie to expire automatically 12 hours after creation. This solution is straightforward to be implemented, however, has a major drawback. It sets an expiration timestamp in a user's browser. **Anybody who would gain an access to the session cookie can easily extend the timestamp by modifying the cookie.**

To solve the issue in a much more secure way the expiration timestamp should be stored on a server side. This is also a solution proposed in the *Securing Rails Application* guide:

> One possibility is to set the expiry time-stamp of the cookie with the session ID. However the client can edit cookies that are stored in the web browser so expiring sessions on the server is safe.

If you use `devise` [gem](https://github.com/plataformatec/devise) for users authentication in your Ruby on Rails app it has a built-in `Timeoutable` [module](https://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/Timeoutable) which takes care of verifying whether a user session has already expired or not. To use it you need to enable it inside a model which represents a user in your application:

```ruby
class  User < ActiveRecord::Base
  devise :timeoutable
end
```

After that you can set `timeout_in` option of `devise` initializer to a value which fits your needs (by default it's 30 minutes):

```ruby
# ==> Configuration for :timeoutable
# The time you want to timeout the user session without activity.
# After this time the user will be asked for credentials again.
# Default is 30 minutes.
config.timeout_in = 30.minutes
```

If you don't use the gem you can create a `Session` model which would store a user session together with `created_at` & `updated_at` timestamps and take care of removing outdated records. Again, you can find an example in [the official guide](https://guides.rubyonrails.org/security.html#session-expiry).

> **UPDATE 09.10.18:** Thanks to [InCaseOfEmergency's ](https://www.reddit.com/user/InCaseOfEmergency) [comment](https://www.reddit.com/r/ruby/comments/9l48jk/5_security_issues_in_ruby_on_rails_apps_from_real/e769se8/) I learned that Rails 5.2 improved the built-in solution and the expiration timestamp is a part of the session cookie. Awesome!

## 2. Missing a lockout mechanism

### Security issue description

How many times a single user can try to sign in into your application before being banned? If your answer is infinitely, that means that you have a security hole. If a user can try many e-mail and password combinations without any consequence, it also means that an attacker can. Preparing a script to make a dictionary or brute-force attack is a matter of minutes today.

> **A brute-force attack** is trying every possible combination.\
> **A dictionary attack** is a guessing attack based on a precompiled list of options like most commonly used passwords.

To fix the issue user should be blocked after providing an incorrect combination of login and password X number of times.

### Solution(s)

If you use `devise` gem the solution is as simple as the previous one. There is `Lockable` module which allows blocking a user access after a certain number of attempts. A number of allowed attempts is up to you, but five seems to be a good starting point. You can always tweak the value if you start to receive valid complaints from users.

The module provides two unlocking strategies:

1.  `:time` which unblocks a user automatically after some configured time.
2.  `:email` which sends an email to a user when the lock happens, containing a link to unlock an account.

Each of them is much better than none, but again the final decision is up to you. `devise` also make it possible to use both of them at the same time. You can find more details in[ the official documentation](https://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/Lockable).

If you don't use the gem you can implement a similar solution by yourself (the code is open 🙂 ) or check if a library you use offers a similar solution.

![Implementing CAPTCHA also helps in preventing brute-force and dictionary attacks.\
Source: https://hakiri.io/blog/rails-login-security](../../../../.gitbook/assets/e56e320e-d4ba-456b-aeb6-269c327becb1/362c34f7.png)

## 3. User enumeration / guessable email addresses

### Security issue description

Not so obvious, but a serious problem. Please visit your application and go to no-so-often-visited **Reset your password** page. What would happen if you provided an e-mail address which is not associated with any user of the application?

I hope that's not a validation error with **User with the provided email address does not exist** message. Why? For a potential attacker, it provides an easy way to collect email addresses which exist in the system.

It's an easy-peasy job to prepare or find a ready-to-use script for making millions of request with existing email addresses and based on your application responses determine which one of them exist. Having such a list an attacker can use a security hole described in the previous point and without a lockout mechanism, she can gain access to user accounts.

### Solution(s)

An application should respond the same (either it is a JSON response from API or a redirection to a page with some confirmation message) when a user provides email address assigned to one of the application users or a random one. As a result, an attacker won't be able to collect email addresses of your users.

If you use `devise` gem, there is a configuration option called `paranoid` which according to [the code's comment](https://github.com/plataformatec/devise/blob/715192a7709a4c02127afb067e66230061b82cf2/lib/generators/templates/devise.rb#L81):

> It will change confirmation, password recovery and other workflows to behave the same regardless if the e-mail provided was right or wrong.

If you don't use `devise` you should adjust your application to behave the same regardless if a user provides her email address or some other.

![A very clear message (actually two, one under another) which does not divulge any email address.\
Source: https://help.theatremanager.com/book/export/html/4064](../../../../.gitbook/assets/e56e320e-d4ba-456b-aeb6-269c327becb1/604ef9c6.jpg)

## 4. Privilege escalation aka unauthorized access to resources

### Security issue description

Such mistakes should not happen, but they simply just happen. Let's assume that you created a new API endpoint to fetch user's project based on its ID:

```
GET https://my-rails-app.com/api/projects/:project_id
```

You tested it making some cURL requests using IDs of projects assigned to your test user and your application responded with expected JSON payload containing projects' details. You deployed the endpoint to production finally. But wait! Have you checked what would happen if you made a request for a project assigned to another user?

Boom! You forgot to limit access to only `current_user`'s projects.

**There is an excellent sentence summing up this security issue in the official Rails guide:**

> As a rule of thumb, no user input data is secure, until proven otherwise, and every parameter from the user is potentially manipulated.[^1]

### Solution(s)

Always remember about limiting access to as narrow as possible. If you have access to `current_user` method in your application's controllers the quickest fix is replacing:

```ruby
Project.find(params[:id])
```
by:

```ruby
current_user.projects.find(params[:id])
```

If you want to have control over resources in an object-oriented way you can choose either `pundit` or `cancan` gem.

I am more familiar with the first one and it has one useful feature that you should always use in a development environment. By adding the below filter in e.g. your main controller that others inherit from:

```ruby
after_action :verify_authorized
```

the gem will shout on you if you forget to call its authorize method (which limits access to resources basically) in any of controllers' actions[^2]. Thanks to that you can act even before pushing code to a repository.

If you have played with neither `pundit` nor `cancan` yet I recommend giving them a shot.

## 5. Allowing users to use weak passwords

### Security issue description

The vast majority of our applications' users do not have access to tools like [1password](https://1password.com/) or [KeePass](https://keepass.info/) which allow us to generate secure passwords, store them securely and fill them out automatically during every sign in.

An average user chooses passwords that are easy to remember and very often use the same password for every application.

> **In my opinion it is our responsibility to educate users and take care of their security even if it may be a bit unpleasant for them.**

Please don't allow users to create accounts in your application using passwords like `12345678` or `qwerty`. They make brute-force and dictionary attack way much easier.

### Solution(s)

Introduce and apply a password policy.

> A password policy is a set of rules designed to enhance computer security by encouraging users to employ strong passwords and use them properly[^3].

To apply a basic policy just add a custom validation method in your `User` model:

validate :password_complexity

```ruby
def password_complexity
  return  if password.blank? || password =~ /^(?=.*?[A-Z])(?=.*?[a-z])(?=.*?[0-9])(?=.*?[#?!@$%^&*-]).{8,70}$/

  errors.add  :password, "Complexity requirement not met. Length
  should be 8-70 characters and include: 1 uppercase, 1 lowercase,
  1 digit and  1 special character"
end
```

The above method taken from [Devise's wiki](https://github.com/plataformatec/devise/wiki/How-To:-Set-up-simple-password-complexity-requirements) should be a part of your application's code as soon as you finish reading this article 🙂

If you want to have something fancier you can take a look at `strong_password` [gem](https://github.com/bdmac/strong_password), but it may be an overkill 🙂

![Password strength indicators from eBay, Gmail, and Dropbox. Informing users about a password policy in an easy-to-understand way is as important as the policy.\
Source: https://css-tricks.com/password-strength-meter/](../../../../.gitbook/assets/e56e320e-d4ba-456b-aeb6-269c327becb1/d3d4dc6e.png)

## Summary

There were, are and will be security issues in Ruby on Rails applications.

I hope that none of the above issues are present in an application you develop. If you found something that may be corrected I hope that the proposed solutions will help you fix security holes. Good luck 🙂

#### Useful links

1.  [The official OWASP Ruby on Rails security checklist](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet)
2.  [Rails Security Checklist](https://github.com/eliotsykes/rails-security-checklist)
3.  [The SaaS CTO Security Checklist](https://cto-security-checklist.sqreen.io/)

#### Footnotes

[^1]:  <https://edgeguides.rubyonrails.org/security.html#privilege-escalation>[](https://frontdeveloper.pl/2018/10/5-security-issues-in-ruby-on-rails/#easy-footnote-1-232)
[^2]:  <https://github.com/varvet/pundit#ensuring-policies-and-scopes-are-used>[](https://frontdeveloper.pl/2018/10/5-security-issues-in-ruby-on-rails/#easy-footnote-2-232)
[^3]:  <https://en.wikipedia.org/wiki/Password_policy>[](https://frontdeveloper.pl/2018/10/5-security-issues-in-ruby-on-rails/#easy-footnote-3-232)

--- *Igor Springer*
