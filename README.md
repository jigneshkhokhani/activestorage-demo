# How to use activestorage with rails >=5.1 (this is in ~> 5.2.0.beta2)

# Installation

Add Gem in gemfile

```ruby
gem 'activestorage'
```

Enable gem which is use for image processing

```ruby
# Use ActiveStorage variant
gem 'mini_magick', '~> 4.8'
```

Generate migration for activestorage by running

```bash
bin/rails active_storage:install:migrations
bin/rails db:migrate
```
This migration generate two tables `active_storage_blobs` and `active_storage_attachments`.


# Configurations
Active Storage is enabled by default when you create a new Rails application. `config/storage.yml` is created and `config.active_storage.service` is set to `:local` on both development.rb and production.rb

Update configuration as per your need in `development.rb`.

```ruby
# Store uploaded files on the local file system (see config/storage.yml for options)
config.active_storage.service = :local
```

Other values are `:amazon`, `:google`, `:microsoft`, `:mirror`

Add new environment in `config/storage.yml` and use that in `config.active_storage.service`


# Uses

Generate any model which use image or other doc without any attachment field eg.

```bash
rails g resource User name:string
```

## One attachment

Don't use activestorage two table directly. To store attachment use `has_one_attached`

```ruby
class User < ApplicationRecord
	has_one_attached :image
end
```

You don't need extra attachment field in `User` table or no need to create seperate table for that.

You can store attachment in DB in two way.

### Way 1 :-
```ruby
def create
  @user = User.new(user_params)

  respond_to do |format|
    if @user.save
      format.html { redirect_to @user, notice: 'User was successfully created.' }
    else
      format.html { render :new }
    end
  end
end

private

	def user_params
    params.require(:user).permit(:name, :image)
  end
```

### Way 2 :-

```ruby
def create
  @user = User.create!(user_params)
  @user.image.attach(params[:user][:image])

  respond_to do |format|
    if @user.persist?
      format.html { redirect_to @user, notice: 'User was successfully created.' }
    else
      format.html { render :new }
    end
  end    
end

private

	def user_params
    params.require(:user).permit(:name)
  end
```

Don't forgot to add `image` attributes in permit parameters.