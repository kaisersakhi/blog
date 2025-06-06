---
title: Ruby on Rails interview questions [Part-1]
date: 2025-06-06 15:01:20
tags: ruby, ruby on rails inteview, ruby interview questions
---

<img src="https://images.pexels.com/photos/1557688/pexels-photo-1557688.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2" />

### Q:1 What is the MVC architecture pattern and how does Rails implement it?

**Approach:** Explain the separation of concerns between Model, View, and Controller components and their responsibilities in Rails.

The MVC (Model-View-Controller) pattern separates application logic into three interconnected components. In Rails, Models handle data and business logic using Active Record, Views manage the presentation layer through ERB templates and helpers, and Controllers orchestrate the flow between models and views while handling HTTP requests. This separation promotes code organization, reusability, and maintainability. Rails follows convention over configuration, automatically mapping URLs to controller actions and providing structured directories for each component.

---

### Q:2 Explain the difference between `has_many` and `belongs_to` associations.

**Approach:** Define the relationship direction and explain the foreign key placement in database tables.

`belongs_to` establishes a one-to-one connection where the current model contains the foreign key, indicating it "belongs to" another model. `has_many` creates a one-to-many relationship where the current model can have multiple instances of another model. The foreign key is stored in the associated model's table, not the current model. For example, a User `has_many` Posts, while a Post `belongs_to` a User. The posts table contains a user_id foreign key, establishing this relationship in the database schema.

```ruby
class User < ApplicationRecord
  has_many :posts
end

class Post < ApplicationRecord
  belongs_to :user
end
```

---

### Q:3 What are Rails migrations and why are they important?

**Approach:** Explain database schema versioning and the benefits of using migrations for database changes.

Rails migrations are Ruby classes that provide a database-agnostic way to alter your database schema over time. They're important because they allow teams to collaborate on database changes, maintain version control of schema modifications, and provide rollback capabilities. Migrations ensure that database changes are applied consistently across different environments (development, staging, production). They're timestamped and executed in order, maintaining a clear history of database evolution. This approach eliminates manual SQL scripts and reduces deployment errors.

```ruby
class CreatePosts < ActiveRecord::Migration[7.0]
  def change
    create_table :posts do |t|
      t.string :title, null: false
      t.text :content
      t.references :user, null: false, foreign_key: true
      t.timestamps
    end
  end
end
```

---

### Q:4 How do you handle validation in Rails models?

**Approach:** Describe built-in validations and custom validation methods to ensure data integrity.

Rails provides built-in validations that run before saving records to the database, ensuring data integrity at the application level. Common validations include presence, length, format, and uniqueness checks. You can also create custom validations using methods or validator classes. Validations prevent invalid data from being persisted and provide user-friendly error messages. They're triggered automatically during save operations and can be bypassed using methods like `save(validate: false)` when necessary.

```ruby
class User < ApplicationRecord
  validates :email, presence: true, uniqueness: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :name, presence: true, length: { minimum: 2, maximum: 50 }
  validates :age, numericality: { greater_than: 0, less_than: 150 }

  validate :custom_validation

  private

  def custom_validation
    errors.add(:email, "cannot be from blocked domains") if email&.ends_with?("blocked.com")
  end
end
```

---

### Q:5 What is the difference between `find` and `where` in Active Record?

**Approach:** Compare the return types and use cases for these two query methods.

`find` is used to retrieve a single record by primary key and raises an exception if not found, while `where` returns an Active Record relation (which can contain multiple records) and never raises an exception for empty results. `find` immediately executes the query and returns the model instance, whereas `where` is lazy-loaded and can be chained with other query methods. Use `find` when you need a specific record by ID and want to handle missing records with exceptions, and use `where` for filtering records based on conditions.

```ruby
# find - returns single record or raises RecordNotFound
user = User.find(1)  # Raises ActiveRecord::RecordNotFound if ID 1 doesn't exist

# where - returns ActiveRecord::Relation
users = User.where(active: true)  # Returns empty relation if no matches
active_admins = User.where(active: true).where(role: 'admin')  # Chainable
```

---

### Q:6 Explain the Rails request/response cycle.

**Approach:** Walk through the complete flow from browser request to server response.

The Rails request/response cycle begins when a browser sends an HTTP request to the Rails server. The request hits the router, which matches the URL to a specific controller action based on routes.rb configuration. The dispatcher creates an instance of the appropriate controller and calls the specified action method. The controller interacts with models to fetch or manipulate data, then renders a view template (or returns JSON). The view combines the template with data to generate HTML, which is sent back as an HTTP response to the browser. Middleware components can intercept and modify requests/responses throughout this cycle.

---

### Q:7 What are Rails callbacks and when would you use them?

**Approach:** Explain the lifecycle hooks available in Active Record and provide practical examples.

Rails callbacks are hooks that allow you to trigger logic at specific points in an object's lifecycle, such as before/after creation, validation, saving, or destruction. They're useful for tasks like setting default values, sending notifications, logging changes, or cleaning up associated data. Common callbacks include `before_save`, `after_create`, `before_destroy`, and `after_update`. While powerful, callbacks should be used judiciously as they can make code harder to test and debug. Consider using service objects or observers for complex business logic instead of overloading models with callbacks.

```ruby
class User < ApplicationRecord
  before_save :normalize_email
  after_create :send_welcome_email
  before_destroy :cleanup_user_data

  private

  def normalize_email
    self.email = email.downcase.strip if email.present?
  end

  def send_welcome_email
    UserMailer.welcome_email(self).deliver_later
  end

  def cleanup_user_data
    posts.destroy_all
    profile.destroy if profile.present?
  end
end
```

---

### Q:8 How do you implement authentication in a Rails application?

**Approach:** Discuss popular authentication solutions and basic implementation approaches.

Authentication in Rails can be implemented using gems like Devise, Clearance, or custom solutions. The basic approach involves creating a User model with password encryption (using bcrypt), implementing session management, and adding controller methods to handle login/logout. Devise is the most popular solution, providing generators for views, controllers, and models with features like password reset, email confirmation, and account locking. Custom implementation gives more control but requires handling security concerns like password hashing, session management, and CSRF protection manually.

```ruby
# Custom authentication example
class User < ApplicationRecord
  has_secure_password
  validates :email, presence: true, uniqueness: true
end

class SessionsController < ApplicationController
  def create
    user = User.find_by(email: params[:email])
    if user&.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to root_path
    else
      flash[:alert] = "Invalid credentials"
      render :new
    end
  end

  def destroy
    session[:user_id] = nil
    redirect_to root_path
  end
end
```

---

### Q:9 What is the difference between `render` and `redirect_to`?

**Approach:** Explain the HTTP behavior and use cases for each method.

`render` generates and returns an HTTP response without changing the URL, staying within the same request cycle. It's used to display templates, JSON, or other content types. `redirect_to` sends an HTTP redirect response (302 by default) that instructs the browser to make a new request to a different URL, creating a new request cycle. Use `render` when you want to display a view with the current request's data (like form errors), and use `redirect_to` after successful form submissions or when you want to change the URL. Each controller action should only call one of these methods.

```ruby
class PostsController < ApplicationController
  def create
    @post = Post.new(post_params)

    if @post.save
      redirect_to @post, notice: 'Post created successfully'  # New request
    else
      render :new  # Same request, shows validation errors
    end
  end

  def show
    @post = Post.find(params[:id])
    render json: @post  # Returns JSON response
  end
end
```

---

### Q:10 How do you handle background jobs in Rails?

**Approach:** Discuss job processing libraries and queuing strategies for asynchronous tasks.

Background jobs in Rails are handled using libraries like Sidekiq, Resque, or Active Job (Rails' built-in abstraction). These tools allow you to perform time-consuming tasks asynchronously, improving application responsiveness. Common use cases include sending emails, processing images, generating reports, or calling external APIs. Active Job provides a unified interface that works with multiple backends. Jobs are typically stored in Redis or database queues and processed by worker processes. Proper error handling, retry logic, and monitoring are essential for production deployments.

```ruby
# Using Active Job
class SendEmailJob < ApplicationJob
  queue_as :default

  def perform(user_id, email_type)
    user = User.find(user_id)
    case email_type
    when 'welcome'
      UserMailer.welcome_email(user).deliver_now
    when 'newsletter'
      UserMailer.newsletter(user).deliver_now
    end
  rescue StandardError => e
    Rails.logger.error "Failed to send email: #{e.message}"
    raise  # Re-raise to trigger retry
  end
end

# Enqueuing jobs
SendEmailJob.perform_later(user.id, 'welcome')
SendEmailJob.set(wait: 1.hour).perform_later(user.id, 'newsletter')
```

---

### Q:11 What are Rails concerns and how do you use them?

**Approach:** Explain the DRY principle application through mixins and shared functionality.

Rails concerns are modules that encapsulate shared functionality and can be included in multiple classes, promoting code reuse and organization. They're stored in `app/models/concerns` and `app/controllers/concerns` directories. Concerns help extract common methods, validations, associations, or scopes that multiple models or controllers might need. They support both class and instance methods through `ClassMethods` modules and can include other concerns. This pattern follows the DRY principle and makes code more maintainable by centralizing shared logic.

```ruby
# app/models/concerns/timestampable.rb
module Timestampable
  extend ActiveSupport::Concern

  included do
    scope :recent, -> { where('created_at > ?', 1.week.ago) }
    before_save :update_modified_by
  end

  class_methods do
    def created_today
      where(created_at: Date.current.beginning_of_day..Date.current.end_of_day)
    end
  end

  def time_since_creation
    Time.current - created_at
  end

  private

  def update_modified_by
    self.modified_by = Current.user&.id
  end
end

# Usage in models
class Post < ApplicationRecord
  include Timestampable
end

class Comment < ApplicationRecord
  include Timestampable
end
```

---

### Q:12 How do you optimize database queries in Rails?

**Approach:** Discuss N+1 queries, eager loading, and other performance optimization techniques.

Database query optimization in Rails involves several strategies: using `includes` to prevent N+1 queries through eager loading, `joins` for filtering with associations, and `select` to limit returned columns. Index your database columns that are frequently queried. Use `find_each` for batch processing large datasets. Implement database-level constraints and consider counter caches for frequently counted associations. Tools like Bullet gem help identify N+1 queries during development. For complex queries, sometimes raw SQL or database views are more efficient than Active Record methods.

```ruby
# N+1 problem
posts = Post.all
posts.each { |post| puts post.user.name }  # Executes N+1 queries

# Solution with includes
posts = Post.includes(:user)
posts.each { |post| puts post.user.name }  # Executes 2 queries total

# Other optimizations
class Post < ApplicationRecord
  belongs_to :user, counter_cache: true
  scope :published, -> { where(published: true) }
  scope :with_comments, -> { joins(:comments).distinct }
end

# Batch processing
User.find_each(batch_size: 1000) do |user|
  user.update_last_login
end

# Select specific columns
User.select(:id, :name, :email).where(active: true)
```

---

### Q:13 What is the difference between `Class` and `Module` in Ruby?

**Approach:** Compare inheritance capabilities and usage patterns between classes and modules.

Classes in Ruby support inheritance and can be instantiated to create objects, while modules cannot be instantiated and don't support inheritance but can be included in classes as mixins. Classes use single inheritance (one superclass) but can include multiple modules. Modules are used for namespacing and sharing functionality across multiple classes. Classes define object blueprints with state and behavior, while modules provide shared behavior without state. You can check ancestry with `ancestors` method and use `prepend` or `include` to add module functionality to classes.

```ruby
# Module example
module Greetable
  def greet
    puts "Hello, I'm #{name}"
  end
end

# Class with inheritance
class Person
  include Greetable

  attr_accessor :name

  def initialize(name)
    @name = name
  end
end

class Employee < Person
  attr_accessor :company

  def initialize(name, company)
    super(name)
    @company = company
  end
end

# Usage
emp = Employee.new("John", "TechCorp")
emp.greet  # "Hello, I'm John"
puts Employee.ancestors  # Shows inheritance chain
```

---

### Q:14 How do you implement caching in Rails?

**Approach:** Explain different caching strategies and their appropriate use cases.

Rails provides multiple caching strategies: page caching (entire pages), action caching (controller actions), fragment caching (view portions), and low-level caching (arbitrary data). Fragment caching is most common, using `cache` helpers in views with cache keys that include model objects and timestamps. Rails.cache provides methods like `fetch`, `read`, `write` for low-level caching. Cache stores include memory, file system, Redis, and Memcached. Implement cache expiration strategies using touch: true in associations and consider Russian Doll caching for nested cache dependencies.

```ruby
# Fragment caching in views
<% cache @post do %>
  <h1><%= @post.title %></h1>
  <p><%= @post.content %></p>

  <% cache @post.comments do %>
    <% @post.comments.each do |comment| %>
      <% cache comment do %>
        <p><%= comment.body %></p>
      <% end %>
    <% end %>
  <% end %>
<% end %>

# Low-level caching in models/controllers
class Post < ApplicationRecord
  def expensive_calculation
    Rails.cache.fetch("post_#{id}_calculation", expires_in: 1.hour) do
      # Expensive operation here
      complex_computation
    end
  end
end

# Cache expiration
class Comment < ApplicationRecord
  belongs_to :post, touch: true  # Updates post's updated_at when comment changes
end
```

---

### Q:15 What are Rails engines and when would you use them?

**Approach:** Explain engines as miniature applications and their use cases for modularity.

Rails engines are miniature applications that can be mounted inside other Rails applications, providing a way to share functionality across multiple applications or organize large applications into smaller, manageable pieces. Engines have their own routes, controllers, models, views, and assets. They're useful for creating reusable components like admin panels, authentication systems, or business-specific modules. Engines can be packaged as gems for distribution or kept within the same repository for internal modularization. They help maintain separation of concerns and enable teams to work on different parts of an application independently.

```ruby
# Creating an engine
# rails plugin new blog_engine --mountable

# lib/blog_engine/engine.rb
module BlogEngine
  class Engine < ::Rails::Engine
    isolate_namespace BlogEngine

    config.generators do |g|
      g.test_framework :rspec
    end
  end
end

# Mounting in main application routes
Rails.application.routes.draw do
  mount BlogEngine::Engine, at: "/blog"
  # Other routes
end

# Engine routes (blog_engine/config/routes.rb)
BlogEngine::Engine.routes.draw do
  resources :posts
  root to: "posts#index"
end
```

---

### Q:16 How do you handle errors and exceptions in Rails?

**Approach:** Discuss rescue blocks, custom error pages, and exception handling strategies.

Rails error handling involves using rescue blocks in controllers, custom error pages for different HTTP status codes, and application-wide exception handling. Use `rescue_from` in ApplicationController to handle specific exceptions globally. Create custom error pages in public/ directory (404.html, 500.html) or use dynamic error pages with controllers. Log exceptions appropriately and consider using error tracking services like Bugsnag or Rollbar in production. Handle different exception types differently: validation errors should show user-friendly messages, while system errors might redirect to error pages.

```ruby
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with: :record_not_found
  rescue_from ActionController::ParameterMissing, with: :parameter_missing
  rescue_from StandardError, with: :internal_server_error

  private

  def record_not_found(exception)
    Rails.logger.warn "Record not found: #{exception.message}"
    render file: Rails.root.join('public', '404.html'), status: :not_found, layout: false
  end

  def parameter_missing(exception)
    render json: { error: "Missing parameter: #{exception.param}" }, status: :bad_request
  end

  def internal_server_error(exception)
    Rails.logger.error "Internal error: #{exception.message}"
    ExceptionNotifier.notify_exception(exception) if Rails.env.production?
    render file: Rails.root.join('public', '500.html'), status: :internal_server_error, layout: false
  end
end

# In specific controllers
class PostsController < ApplicationController
  def show
    @post = Post.find(params[:id])
  rescue ActiveRecord::RecordNotFound
    flash[:alert] = "Post not found"
    redirect_to posts_path
  end
end
```

---

### Q:17 What is the difference between `merge` and `joins` in Active Record?

**Approach:** Compare how these methods combine query conditions and their effects on the result set.

`joins` creates SQL JOIN clauses to combine tables but only affects the WHERE conditions and doesn't load associated records into memory. `merge` combines query scopes and conditions from associated models, allowing you to apply scopes defined in other models to your current query. Use `joins` when you need to filter records based on associated table conditions without loading the associations. Use `merge` when you want to apply existing scopes from associated models to create more readable and maintainable queries. `merge` is particularly useful with named scopes.

```ruby
class User < ApplicationRecord
  has_many :posts
  scope :active, -> { where(active: true) }
end

class Post < ApplicationRecord
  belongs_to :user
  scope :published, -> { where(published: true) }
  scope :recent, -> { where('created_at > ?', 1.week.ago) }
end

# Using joins - only filters, doesn't load user data
posts_with_active_users = Post.joins(:user).where(users: { active: true })

# Using merge - applies User's active scope
posts_with_active_users = Post.joins(:user).merge(User.active)

# More complex merge example
Post.joins(:user)
    .merge(User.active)
    .merge(Post.published)
    .merge(Post.recent)

# This is more readable than:
Post.joins(:user)
    .where(users: { active: true })
    .where(published: true)
    .where('posts.created_at > ?', 1.week.ago)
```

---

### Q:18 How do you implement file uploads in Rails?

**Approach:** Discuss Active Storage, image processing, and file validation strategies.

Rails file uploads are typically handled with Active Storage (Rails 5.2+), which provides a unified API for uploading files to cloud services or local storage. Active Storage creates attachment associations in models and handles file metadata, variants for image processing, and direct uploads. For image processing, integrate with ImageMagick or libvips through the image_processing gem. Implement validations for file size, content type, and dimensions. Consider security aspects like virus scanning and content type verification. For large files, use direct uploads to avoid server memory issues.

```ruby
# Model with Active Storage
class User < ApplicationRecord
  has_one_attached :avatar
  has_many_attached :documents

  validate :avatar_validation

  private

  def avatar_validation
    return unless avatar.attached?

    if avatar.byte_size > 5.megabytes
      errors.add(:avatar, 'File size must be less than 5MB')
    end

    unless avatar.content_type.in?(['image/jpeg', 'image/png', 'image/gif'])
      errors.add(:avatar, 'Must be a JPEG, PNG, or GIF image')
    end
  end
end

# Controller handling uploads
class UsersController < ApplicationController
  def update
    if @user.update(user_params)
      redirect_to @user, notice: 'Profile updated successfully'
    else
      render :edit
    end
  end

  private

  def user_params
    params.require(:user).permit(:name, :email, :avatar, documents: [])
  end
end

# View with file upload
<%= form_with model: @user do |form| %>
  <%= form.file_field :avatar, accept: 'image/*' %>
  <%= form.file_field :documents, multiple: true %>
  <%= form.submit %>
<% end %>

# Displaying images with variants
<%= image_tag @user.avatar.variant(resize_to_limit: [300, 300]) if @user.avatar.attached? %>
```

---

### Q:19 What are Rails serializers and how do you use them?

**Approach:** Explain JSON serialization patterns and popular serializer gems for API responses.

Rails serializers provide a way to customize JSON output for API responses, offering more control than the default `to_json` method. Popular gems include Active Model Serializers and jsonapi-serializer (formerly fast_jsonapi). Serializers define which attributes to include, handle associations, compute derived values, and format data consistently. They separate presentation logic from models and support conditional attributes, nested serialization, and different serialization contexts. This approach is essential for building clean APIs and controlling data exposure.

```ruby
# Using Active Model Serializers
class UserSerializer < ActiveModel::Serializer
  attributes :id, :name, :email, :created_at, :full_name

  has_many :posts, serializer: PostSerializer

  attribute :avatar_url, if: :has_avatar?

  def full_name
    "#{object.first_name} #{object.last_name}"
  end

  def avatar_url
    Rails.application.routes.url_helpers.url_for(object.avatar) if object.avatar.attached?
  end

  def has_avatar?
    object.avatar.attached?
  end
end

class PostSerializer < ActiveModel::Serializer
  attributes :id, :title, :content, :published_at

  belongs_to :user, serializer: UserSerializer
end

# Controller usage
class Api::UsersController < ApplicationController
  def show
    user = User.find(params[:id])
    render json: user, serializer: UserSerializer
  end

  def index
    users = User.includes(:posts)
    render json: users, each_serializer: UserSerializer
  end
end

# Custom serialization without gems
class User < ApplicationRecord
  def as_json(options = {})
    super(options.merge(
      only: [:id, :name, :email],
      methods: [:full_name],
      include: { posts: { only: [:id, :title, :published_at] } }
    ))
  end
end
```

---

### Q:20 How do you implement authorization in Rails?

**Approach:** Discuss role-based access control and popular authorization gems like Pundit or CanCanCan.

Authorization in Rails controls what authenticated users can do, typically implemented using gems like Pundit, CanCanCan, or custom solutions. Pundit uses policy classes that define permissions for each model, providing a clean separation of authorization logic. CanCanCan uses ability classes with a DSL to define rules. Both support role-based access control, resource-based permissions, and conditional authorization. Implementation involves checking permissions in controllers and views, handling unauthorized access gracefully, and organizing complex permission logic maintainably.

```ruby
# Using Pundit
class PostPolicy < ApplicationPolicy
  def show?
    record.published? || record.user == user || user.admin?
  end

  def create?
    user&.present?
  end

  def update?
    record.user == user || user.admin?
  end

  def destroy?
    record.user == user || user.admin?
  end

  class Scope < Scope
    def resolve
      if user&.admin?
        scope.all
      elsif user
        scope.where(published: true).or(scope.where(user: user))
      else
        scope.where(published: true)
      end
    end
  end
end

# Controller usage
class PostsController < ApplicationController
  before_action :authenticate_user!, except: [:index, :show]

  def index
    @posts = policy_scope(Post)
  end

  def show
    @post = Post.find(params[:id])
    authorize @post
  end

  def create
    @post = current_user.posts.build(post_params)
    authorize @post

    if @post.save
      redirect_to @post
    else
      render :new
    end
  end

  def update
    @post = Post.find(params[:id])
    authorize @post

    if @post.update(post_params)
      redirect_to @post
    else
      render :edit
    end
  end
end

# View usage
<% if policy(@post).update? %>
  <%= link_to 'Edit', edit_post_path(@post) %>
<% end %>

<% if policy(@post).destroy? %>
  <%= link_to 'Delete', @post, method: :delete, data: { confirm: 'Are you sure?' } %>
<% end %>
```

---

### Q:21 What is the difference between `present?` and `exists?` in Rails?

**Approach:** Compare memory usage and database query behavior between these two methods.

`present?` loads records into memory to check if the collection contains any elements, while `exists?` performs a database query (SELECT 1) without loading records, making it more memory-efficient for large datasets. `present?` is useful when you need to work with the records afterward, but `exists?` is better for simple presence checks. `exists?` can also accept conditions to check for specific records. Use `exists?` for conditional logic where you only need to know if records exist, and `present?` when you'll use the loaded records immediately after the check.

```ruby
# exists? - performs COUNT query, doesn't load records
if User.where(active: true).exists?
  puts "Active users found"
end

# More specific exists check
if User.exists?(email: 'admin@example.com')
  puts "Admin user exists"
end

# present? - loads records into memory
users = User.where(active: true)
if users.present?
  puts "Found #{users.count} active users"
  users.each { |user| puts user.name }  # Records already loaded
end

# Performance comparison
# Inefficient - loads all records just to check presence
if User.where(active: true).present?
  # Only checking existence, wasted memory
end

# Efficient - only checks existence
if User.where(active: true).exists?
  # Minimal database query
end

# When you need both check and data
users = User.where(active: true)
if users.present?
  # Work with loaded users
  process_users(users)
end
```

---

### Q:22 How do you handle database transactions in Rails?

**Approach:** Explain ACID properties and Rails transaction methods for data consistency.

Rails transactions ensure data consistency by wrapping multiple database operations in a single atomic unit that either completes entirely or rolls back completely. Use `ActiveRecord::Base.transaction` or model instance transactions to group operations. Transactions automatically rollback on exceptions and can be manually rolled back using `raise ActiveRecord::Rollback`. Nested transactions use savepoints in supported databases. Consider transaction isolation levels for concurrent access scenarios and be aware that transactions lock database resources, so keep them short and focused.

```ruby
# Basic transaction usage
def transfer_money(from_account, to_account, amount)
  ActiveRecord::Base.transaction do
    from_account.withdraw(amount)
    to_account.deposit(amount)

    # Both operations succeed or both fail
    TransactionLog.create!(
      from_account: from_account,
      to_account: to_account,
      amount: amount
    )
  end
rescue ActiveRecord::RecordInvalid => e
  Rails.logger.error "Transfer failed: #{e.message}"
  false
end

# Model-level transactions
class User < ApplicationRecord
  def deactivate_with_cleanup
    transaction do
      update!(active: false, deactivated_at: Time.current)
      posts.update_all(published: false)
      profile.destroy! if profile.present?

      # Manual rollback condition
      raise ActiveRecord::Rollback if some_condition_fails?
    end
  end
end

# Nested transactions with savepoints
User.transaction do
  user = User.create!(name: 'John')

  begin
    User.transaction(requires_new: true) do  # Creates savepoint
      profile = user.create_profile!(bio: 'Invalid bio that might fail')
      raise ActiveRecord::Rollback if profile.invalid?
    end
  rescue ActiveRecord::StatementInvalid
    # Handle savepoint rollback
    Rails.logger.warn "Profile creation failed, but user creation continues"
  end

  # User creation continues even if profile failed
end
```

---

### Q:23 What are Rails view helpers and how do you create custom ones?

**Approach:** Explain built-in helpers and demonstrate creating reusable view logic.

Rails view helpers are methods that assist in generating HTML and formatting data in views, promoting code reuse and keeping views clean. Built-in helpers include `link_to`, `form_with`, `image_tag`, and formatting helpers like `number_to_currency`. Custom helpers are created in app/helpers/ directory and are automatically available in views. They should contain presentation logic, not business logic, and can accept parameters to make them flexible. Helpers can be tested separately and shared across multiple views and layouts.

```ruby
  module ApplicationHelper
    def page_title(title = nil)
      if title.present?
        "#{title} | MyApp"
      else
        "MyApp - Welcome"
      end
    end

    def flash_class(level)
      case level.to_sym
      when :notice then "alert alert-info"
      when :success then "alert alert-success"
      when :error   then "alert alert-danger"
      when :alert   then "alert alert-warning"
      else "alert alert-info"
      end
    end

    def user_avatar(user, size: 'medium')
      sizes = { small: 32, medium: 64, large: 128 }
      dimension = sizes[size.to_sym] || sizes[:medium]

      if user.avatar.attached?
        image_tag user.avatar.variant(resize_to_limit: [dimension, dimension]),
                  alt: user.name,
                  class: "avatar avatar-#{size}"
      else
        image_tag "default_avatar.png",
                  width: dimension,
                  height: dimension,
                  alt: "Default Avatar",
                  class: "avatar avatar-#{size}"
      end
    end
  end
```
