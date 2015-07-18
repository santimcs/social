cd\ruby

rails new social

cd social

git init

rails generate resource User name email

rake db:migrate

rails g model Subscription leader:references follower:references

rake db:migrate

## app/models/subscription.rb
  belongs_to :leader, class_name: 'User'

  belongs_to :follower, class_name: 'User'

## app/models/user.rb
  has_many :subscriptions, foreign_key: :follower_id, dependent: :destroy

  has_many :leaders, through: :subscriptions

  def following?(leader)

      leaders.include? leader

  end

  def follow!(leader)

      if leader != self && !following?(leader)

         leaders << leader

      end

  rails console

  alice = User.create name: "Alice"

  bob = User.create name: "Bob"

  alice.follow! bob

  alice.following? bob

  bob.following? alice

## app/models/user.rb        

  has_many :reverse_subscriptions, foreign_key: :leader_id, class_name: 'Subscription', dependent: :destroy

  has_many :followers, through: :reverse_subscriptions

rails console

alice = User.find(1)

bob = User.find(2)

alice.followers

alice.followers.to_a

rails g resource Post title body:text url type user:references

rake db:migrate

rails g resource TextPost --parent=Post --migration=false

rails g resource ImagePost --parent=Post --migration=false

## app/models/post.rb 
validates :user_id, presence: true

validates :type, presence: true

## app/models/image_post.rb
validates :url, presence: true

## app/models/text_post.rb
validates :body, presence: true

## app/models/user.rb
  has_many :posts, dependent: :destroy

  has_many :text_posts, dependent: :destroy

  has_many :image_posts, dependent: :destroy

rails console

alice = User.find(1)

post1 = alice.text_posts.create(body: "First Post")

post2 = alice.image_posts.createurl: "http://i.imgur.com/Y7syDEa.jpg")

alice.posts.to_a

alice.text_posts.to_a

rails g resource Comment body:text post:references user:references
  
rake db:migrate

git add .

git commit -m "initial commit version"

## create repository social on github
git remote add origin https://github.com/santimcs/social.git

git push -u origin master

## Chapter 9 Authentication

## gemfile
gem 'bootstrap-sass'

bundle install

## app/assets/stylesheets/application.css
*= require bootstrap

## app/assets/javascripts/application.js
//= require bootstrap

## app/views/layouts/application.html.erb
<div class="container">

  <%= yield %>

</div>

## app/controllers/posts_controller.rb
def index
    @posts = Post.all
end

def show
    @post = Post.find(params[:id])
end

## create app/views/posts/index.html.erb
<div class="page-header">

  <h1>Home</h1>

</div>

<%= render @posts %>

## create app/views/text_posts/_text_post.html.erb
<div class="panel panel-heading">

  <div class="panel-heading">

    <h3 class="panel-title">

      <%= text_post.title %>

    </h3>

  </div>

  <div class="panel-body">
    <p>
      <em>
        By <%= text_post.user.name %>
      </em>
    </p>

    <%= text_post.body %>
  </div>
</div>

## create app/views/image_posts/_image_post.html.erb
<div class="panel panel-heading">

  <div class="panel-heading">

    <h3 class="panel-title">

      <%= image_post.title %>

    </h3>

  </div>

  rails server

  <div class="panel-body">
  
    <p>
      <em>
        By <%= image_post.user.name %>
      </em>
    </p>

    <%= image_tag image_post.url, class: "img-responsive" %>

    <%= image_post.body %>

  </div>

</div>
  
## create app/views/posts/show.html.erb
<div class="page-header">

  <h1>Post</h1>

</div>

<%= render @post %>

<%= link_to "Home", posts_path, class: "btn btn-default" %>

# modify gemfile
gem 'bcrypt', '~> 3.1.7'

bundle install

rails g migration AddPasswordDigestToUsers password_digest

## edit app/models/user.rb
has_secure_password

  validates :email, presence: true, uniqueness: true

## edit config/routes.rb
get 'signup', to: 'users#new', as: 'signup'

  root 'posts#index'  

##  app/views/users/new.html.erb

## app/views/layouts/application.html.erb

rails g controller Sessions

## edit config/routes.rb
  resources :sessions

## create app/views/sessions/new.html.erb

## edit app/controllers/sessions_controller.rb

## edit app/controllers/application _controller.rb
  private

  def current_user
    if session[:user_id]
       @current_user ||= User.find(session[:user_id])
    end
  end
  helper_method :current_user

## edit app/views/layouts/application.html.erb
  <div class="pull-right">
      <% if current_user %>
        <%= link_to 'Log Out', logout_path %>
      <% else %>
        <%= link_to 'Log In', login_path %> or
        <%= link_to 'Sign Up', signup_path %>
      <% end %>
    </div>

## edit app/controllers/application_controller.rb      
  def authenticate_user!
    redirect_to login_path unless current_user
  end

## eidt app/controllers/posts_controller.rb
  before_action :authenticate_user!

## edit app/models/user.rb
  def timeline_user_ids
     leader_ids + [id]
  end

## edit app/controllers/posts_controller.rb
  user_ids = current_user.timeline_user_ids
  
  @posts = Post.where(user_id: user_ids)
             .order("created_at DESC")

