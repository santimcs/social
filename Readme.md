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