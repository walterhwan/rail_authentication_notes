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
end
```

### Create Database
```ruby
rails db:create
```

# How to approach the assessment
### Add all the models (i.e User, Link, and Comment) using `rails g model Objects`
  * Add `validates :column_name, presence: true`
  * Add relaitons such as `has_many`, `belongs_to` and `has_one`

##### Add authentication methods in `user.rb`
```ruby
validates :username, :password_digest, :session_token, presence: true
validates :password, length: { minimum: 6, allow_nil: true}
after_initialize :ensure_session_token

def self.find_by_credentials(username, password)

def ensure_session_token

def reset_session_token

def password=(password)

def is_password?(password)

def self.generate_session_token
```

##### Add methods in ApplicationController

```ruby
helper_method :current_user, :signed_in?

def current_user

def sign_in(user)

def signed_in?

def sign_out

def require_signed_in
```

### Add all the controllers

  * Add controllers using `rails g controller Objects`
  * Add routes i.e. write `resources` in `routes.rb`
  * In each controller files, add their controller action (i.e. `indexx`, `new`, `create`, etc)
  * Add needed `.html.erb` files
  * Add `object_params` private methods

#### Create all the views

# To-do list when adding a new feature/resource
* Migrations

  + `rails g migration CreateObjects`
* Model definition
  + `validates :username, :password_digest, :session_token, presence: true`

  + `belongs_to`, `has_many`, `has_one`
  + `helper_methods: `
* Routes
  + `resources :objects`
  + `resource :object`
* Controller + controller actions
* Views (should coincide with the actions that render them)

# Rails Commands

### Add bash alias
```bash
alias my_command='long bash commands'
```
For example, if you want to use ```ber``` as ```bundle exec rspec```
```bash
alias ber='bundle exec rspec'
```

## Useful Alias
Be careful when you type alias that are similar to default command like `rm`. You might accidently delete something.
```ruby
alias rr='rails routes'
alias rgm='rails g migration'
alias rdbmt='rails db:migrate db:test:load'

alias rdbm='rails db:migrate'
alias gm='rails g model'
alias gc='rails g controller'
```

### Generate Migration
```bash
rails g migration CreateObjects
```

### Re-create database, migrate your current schema, updates test database schema to mirror the development DB
```bash
rails db:reset db:migrate db:test:load
```

### Generate Model and Pre-populate Migration at once
```ruby
rails g model Object col_name1:datatype col_name2:datatype ...
```
i.e.
```ruby
rails g model User username:string password_digest:string session_token:string
```

### Generate Controller
```bash
rails g controller Objects
```
this will generate objects_controller.rb

```bash
rails g controller Objects method_name1 method_name2 ...
```

i.e.
```bash
rails g controller Users index show create
```

### Check Routes
```bash
rails routes
```

### Command line in Rails
```bash
rails c
```

### Start Rails Server
```bash
rails s
```


# Generate User Table
Database level validations may not matter in this assessment
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

### Other column attributes
```ruby
:default, :null, :inclusion, :presence
```

### Change Columns
```ruby
rename_column :table, :old_column, :new_columnx
add_column :table_name, :new_column
remove_column :table_name, :column_name
add_index :table_name, :column_name, unique: true
change_column :table_name, :column_name, :new_data_type
```


# Routes

### Create New Routes
```ruby
resources :objects
```

### Only add some routes
```ruby
resources :objects, only: [:action1, :action2]
```
### Nested Routes
```ruby
resources :objects do
  resources :sub_objects
end
```
### Add custom routes
```ruby
http_verb 'url_Path', to: 'url_pattern', as: 'prefix_verb'
```
i.e.
```ruby
get '/bands/:band_id/albums/new', to: 'albums#new', as: 'new_band_album'
```
this will create a route
```bash
Prefix Verb           URI Pattern                          Controller#Action
new_band_album GET    /bands/:band_id/albums/new(.:format) albums#new
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

### HTTP "methods" and Their Corresponding Controller Action
i.e if you have object resources

Prefix | http methods | Controller Action | (defalut) URI Pattern
--- | --- | --- | ---
objects | GET | index | /objects
new_object | GET | new | /objects/new
edit_object | GET | edit | /objects/:id
object | GET | show | /objects/:id
 | Post | create | /objects
 | PATCH | update | /objects/:id
 | PUT | update | /objects/:id
 | DELETE | destroy | /objects/:id

# View Helpers
`app/helpers/application_helper.rb`
This helper method should be put inside every `<form>` tag to enable CSRF protection
```ruby
def auth_token_input
    "<input type=\"hidden\"name=\"authenticity_token\"
      value=\"#{ form_authenticity_token }\">".html_safe
end
```

# Rails Views Code Snippet

### Use PATCH or DELETE http Method in `<form>` tag
```html
<input type="hidden" name="_method" value="PATCH">
<input type="hidden" name="_method" value="DELETE">
```
i.e.


### Button_to and Link_to
```html+erb
<%= button_to 'my_link_text', aciton_rul, method: :http_method %>
<%= link_to 'my_link_text', aciton_rul, method: :http_method %>
```
i.e.
```html+erb
<%= link_to 'Sign In', session_url %>
<%= button_to 'Sign Out', session_url, method: :delete %>
```

### Create a `_form.html.erb` so that both `new.html.erb` and `edit.html.erb` can use the partial
```html+erb
<% action_url = object.persisted? ? object_url(object) : objects_url %>
<% method = object.persisted? ? "patch" : "post" %>

<!-- ... -->
<% submit_val = album.persisted? ? "Update" : "Create" %>
```

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

### Record error messages in flash cookie
```ruby
flash[:errors] = object.errors.full_messages

flash.now[:errors] = object.errors.full_messages
```
