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

  has_many :reverse_subscriptions, foreign_key: :leader_id, class_name: 'Subscription', dependent: :destroy
  has_many :followers, through: :reverse_subscriptions

  def following?(leader)
  	  leaders.include? leader
  end

  def follow!(leader)
  	  if leader != self && !following?(leader)
         leaders << leader
      end
  end

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

rails g resource Comment body:text post:references user:references  
rake db:migrate