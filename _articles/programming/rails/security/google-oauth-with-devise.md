# Google OAuth with Devise

> Original article: <https://ankane.org/google-oauth-with-devise>

Here's a quick guide to setting up Google OAuth as your app's exclusive authentication method

Add to your Gemfile

```ruby
gem 'devise'
gem 'omniauth-google-oauth2'
gem 'dotenv-rails', groups: [:development, :test]

```

And run

```command
rails generate devise:install

```

Create a `User` model

```command
rails g model User

```

In the migration, add

```ruby
create_table :users do |t|
  t.string :name
  t.string :email
  t.string :provider
  t.string :uid
  t.string :remember_token
  t.datetime :remember_created_at
  t.timestamps null: false
end

add_index :users, :email, unique: true
add_index :users, [:uid, :provider], unique: true

```

In your `User` model, add

```ruby
devise :rememberable, :omniauthable, omniauth_providers: [:google_oauth2]

```

Create a controller

```ruby
rails g controller OmniauthCallbacks

```

with

```ruby
class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  # replace with your authenticate method
  skip_before_action :authenticate_user!

  def google_oauth2
    auth = request.env["omniauth.auth"]
    user = User.where(provider: auth["provider"], uid: auth["uid"])
            .first_or_initialize(email: auth["info"]["email"])
    user.name ||= auth["info"]["name"]
    user.save!

    user.remember_me = true
    sign_in(:user, user)

    redirect_to after_sign_in_path_for(user)
  end
end

```

In your routes, add

```ruby
devise_for :users, controllers: {omniauth_callbacks: "omniauth_callbacks"}

```

In `config/devise.rb`, add

```ruby
config.omniauth :google_oauth2, ENV["GOOGLE_CLIENT_ID"], ENV["GOOGLE_CLIENT_SECRET"], access_type: "online"

```

Follow the [Google API Setup](https://github.com/zquestz/omniauth-google-oauth2#google-api-setup) instructions and add your credentials in `.env`

```command
GOOGLE_CLIENT_ID=0000000
GOOGLE_CLIENT_SECRET=0000000

```

Bonus
-----

To remove the hash from the end of the URL after sign-in, use:

```javascript
var href = window.location.href;
if (href[href.length - 1] === "#") {
  if (typeof window.history.replaceState == "function") {
    history.replaceState({}, "", href.slice(0, -1));
  }
}

```
