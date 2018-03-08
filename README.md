# Ruby on Rails - Authentication Notes

This document provide collection of short reference of how to write various rails code in a single page so it is easier to search

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

## Useful Alias
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

    t.timestamp
  end

  add_index :users, :session_token, unique: true
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
HTML methods | Controller Action
--- | ---
GET | index
Post | create
GET | new
GET | edit
GET | show
PATCH | update
PUT | update
DELETE | destroy


# View Helpers
`app/helpers/application_helper.rb`
This helper method should be put right after every `<form>` tag to enable CSRF protection
```ruby
def auth_token_input
    "<input type=\"hidden\"name=\"authenticity_token\"
      value=\"#{ form_authenticity_token }\">".html_safe
end
```
