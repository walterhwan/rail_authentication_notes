# Ruby on Rails - Authentication Notes

This document provide collection of short reference of how to write various rails code in a single page so it is easier to search. However, it won't explain exactly how to use them.

# Start Your Rails Project

### Creating a new rails project in postgre
```bash
rails new app_name --database=postgresql
```

### Add useful gems
```ruby
# Gemfile
gem 'bcrypt'

group :development do
  gem 'better_errors'
  gem 'binding_of_caller'
  gem 'pry-rails'
  gem 'annotate'
  gem 'faker'
end
```

### Reset Database
```ruby
rails db:reset
```

# To-do list when adding a new feature/resource
+ Migrations
+ Model definition (validations, associations, helper methods)
+ Routes
+ Controller + controller actions
+ Views (should coincide with the actions that render them)

# Rails Commands

### Add bash alias
```bash
alias my_command='long bash commands'
```
For example, if you want to use ```ber``` as ```bundle exec rspec```
```bash
alias ber='bundle exec rspec'
```

### Generate Migration
```bash
rails g migration CreateObjects
```

### Destroy database, re-create database, and migrate your current schema
```bash
rails db:drop db:reset db:migrate
```

### Generate Model and Pre-populate Migration at once
```ruby
rails g model Object col_name1:datatype col_name2:datatype ...
```
i.e.
```ruby
rails g model User username:string password_digest:string session_token:string
```

### Generate Controllers
```bash
rails g controller Objects
```
this will generate objects_controller.rb

### Check Routes
```bash
rails routes
```

### Command line in Rails
```bash
rails c
```

# Useful Alias
```ruby
alias rr='rails routes'
alias rsgm='rails g migration'
alias rsm='rails db:migrate'
```

# General User Table
```ruby
def change
  create_table :users do |t|
    t.string :username, null: false
    t.string :password_digest, null: false
    t.string :session_token, null: false

    t.index :session_token, unique: true

    t.timestamp
  end
end
```


# BCrypt

### Digests a password and builds a Password object.
```ruby
BCrypt::Password.create(password)
```

### Builds a Password object for the digest.
```ruby
BCrypt::Password.new(password_digest)
```

### Checks if a password matches a digest.
```ruby
BCrypt::Password.is_password?(password)
```

### Check if a password_digest String is indeed create by a password
```ruby
BCrypt::Password.new(password_digest).is_password?(password)
```

# Session Token

### Generate Session Token
```ruby
def self.generate_session_token
  SecureRandom::urlsafe_base64(16)
end
```

### Reset Session Token
```ruby
def reset_session_token!
  self.session_token = self.class.generate_session_token
  self.save!
  self.session_token
end
```

### Ensure Session Token
Use `after_initialize` to ensure user have `session_token` right after `User.new`
```ruby
after_initialize :ensure_session_token

private
def ensure_session_token
  self.session_token ||= self.class.generate_session_token
end
```

### HTML "methods" and Their Corresponding Controller Action
Prefix | HTML methods | Controller Action
--- | --- | ---
bands | GET | index
new_band | GET | new
edit_band | GET | edit
band | GET | show
 | Post | create
 | PATCH | update
 | PUT | update
 | DELETE | destroy

bands GET    /bands(.:format)          bands#index
            POST   /bands(.:format)          bands#create
   new_band GET    /bands/new(.:format)      bands#new
  edit_band GET    /bands/:id/edit(.:format) bands#edit
       band GET

# View Helpers
`app/helpers/application_helper.rb`
This helper method should be put right after every `<form>` tag to enable CSRF protection
```ruby
def auth_token_input
    "<input type=\"hidden\"name=\"authenticity_token\"
      value=\"#{ form_authenticity_token }\">".html_safe
end
```

# Views Code Snippet

### Use PATCH or DELETE HTML Method in `<form>` tag
```html
<input type="hidden" name="_method" value="PATCH">
<input type="hidden" name="_method" value="DELETE">
```
i.e.


### Insert a partial to a erb file
```html+erb
<%= render "my_partial" %>
```

i.e. Insert a partial with local variables

```html+erb
<!-- app/views/user/new.html.erb -->
<%= render "form", user: @user, action: :new %>

<!-- app/views/user/edit.html.erb -->
<%= render "form", user: @user, action: :edit %>

<!-- app/views/user/_form.html.erb -->
<!-- Is this a new user to create, or an existing one to edit? -->
<% action_url = (action == :new) ? users_url : user_url(user) %>

<form action="<%= action_url %>" method="post">
  <% if action == :edit %>
    <input type="hidden" name="_method" value="patch">
  <% end %>
  <!-- inputs go here... -->
</form>
```
