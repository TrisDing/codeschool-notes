# Rails Notes

### 1. DB CURD

#### Hash
* Hash Recipe: ```variable = { key: value }```
* Read recipe: ```variable[:key] = value``` or ```variable.key = value```

#### Create
```ruby
t = Tweet.new
t.status = "I <3 brains."
t.save

t = Tweet.new(
 status: "I <3 brains",
 zombie: "Jim"
)
t.save

Tweet.create(status: "I <3 brains", zombie: "Jim")
```
#### Read
```ruby
Tweet.find(2)
Tweet.find(3, 4, 5)
Tweet.first
Tweet.last
Tweet.all
Tweet.count
Tweet.order(:zombie)
Tweet.limit(10)
Tweet.where(zombie: "ash")
Tweet.where(zombie: "ash").order(:status).limit(10)
Tweet.where(zombie: "ash").first
```
#### Update
```ruby
t = Tweet.find(3)
t.zombie = "EyeballChomper"
t.save

t = Tweet.find(2)
t.attributes = {
 status: "Can I munch your eyeballs?",
 zombie: "EyeballChomper"
}
t.save

t = Tweet.find(2)
t.update(
 status: "Can I munch your eyeballs?",
 zombie: "EyeballChomper"
)
```
### Delete
```ruby
t = Tweet.find(2)
t.destroy

Tweet.find(2).destroy

Tweet.destroy_all
```

### 2. Models
```ruby
# app/modles/tweet.rb
class Tweet < ActiveRecord::Base
end
```

#### Validations
```ruby
class Tweet < ActiveRecord::Base
  validates_presence_of     :status
  validates_numericality_of :fingers
  validates_uniqueness_of   :toothmarks
  validates_confirmation_of :password
  validates_acceptance_of   :zombification
  validates_length_of       :password, minimum: 3
  validates_format_of       :email, with: /regex/i
  validates_inclusion_of    :age, in: 21..99
  validates_exclusion_of    :age, in: 0...21, message: "Sorry you must be over 21"

  # multiple validations
  validates :status, presence: true,
                     uniqueness: true,
                     numericality: true,
                     length: { minimum: 0, maximum: 2000 },
                     format: { with: /.*/ },
                     acceptance: true,
                     confirmation: true
end
```
#### Relationships
```ruby
class Zombie < ActiveRecord::Base
  has_many :tweets
end

class Tweet < ActiveRecord::Base
  belongs_to :zombie
end
```

### 3. Views
```ruby
index.html.erb       # list all tweets
create.html.erb      # create a tweet
show.html.erb        # view a tweet
edit.html.erb        # edit a tweet
delete.html.erb      # delete a tweet
application.html.erb # The main layout
```

#### All Links For Tweets
| Action          | Code                      | The URL        |
| --------------- | ------------------------- | -------------- |
| List all tweets | tweets_path               | /tweets        |
| New tweet form  | new_tweet_path            | /tweets/new    |
| Show a tweet    | tweet                     | /tweets/1      |
| Edit a tweet    | edit_tweet_path(tweet)    | /tweets/1/edit |
| Delete a tweet  | tweet, :method => :delete | /tweets/1      |

Link Recipe: ```<%= link_to text_to_show, code %>```

### 4. Controllers
```ruby
# /app/controllers/tweets_controller.rb
class ArticlesController < ApplicationController
  def index     # List all tweets
  def edit      # Show a single tweet
  def new       # Show a new tweet form
  def create    # Create a new tweet
  def update    # Update a tweet
  def destroy   # Delete a tweet
  def new       # Delete a tweet
end
```

#### Scaffolding
```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def edit
    @article = Article.find(params[:id])
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render 'new'
    end
  end

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article
    else
      render 'edit'
    end
  end

  def destroy
    @article = Article.find(params[:id])
    @article.destroy

    redirect_to articles_path
  end

  private

  def article_params
    params.require(:article).permit(:title, :text)
  end

end
```
Instance Variable: grants view access to variables with @

#### Strong Params
Strong Params Required only when: CREATING or UPDATING with MULTIPLE Attributes
```ruby
# /tweets?tweet[status]=I’m dead
# params = { tweet: {status: "I’m dead" }}
@tweet = Tweet.create(params.require(:tweet).permit(:status))

@tweet = Tweet.create(params[:tweet])
params.require(:tweet).permit([:status, :location])

# Alternately
@tweet = Tweet.create(params[:tweet])
```

#### Respond with HTML, JSON, XML
```ruby
def show
  @tweet = Tweet.find(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @tweet }
    format.xml { render xml: @tweet }
  end
end
```

#### Redirect and Flash
```ruby
# /app/controllers/tweets_controller.rb
def edit
  @tweet = Tweet.find(params[:id])
  if session[:zombie_id] != @tweet.zombie_id
    flash[:notice] ="Sorry, you can’t edit this tweet"
    redirect_to(tweets_path)

    # Alternately
    redirect_to(tweets_path, :notice =>"Sorry, you can’t edit this tweet")
  end
end

# /app/views/layouts/application.html.erb
<% if flash[:notice] %>
  <div id="notice"><%= flash[:notice] %></div>
<% end %>
 <%= yield %>
```

#### Before Actions
```ruby
before_action :get_tweet, only => [:edit, :update, :destroy]
before_action :check_auth , :only => [:edit, :update, :destroy]

def get_tweet
  @tweet = Tweet.find(params[:id])
end

def check_auth
  if session[:zombie_id] != @tweet.zombie_id
    flash[:notice] = "Sorry, you can’t edit this tweet"
    redirect_to tweets_path
  end
end
```

### 4. Routing
```ruby
# /config/routes.rb
ZombieTwitter::Application.routes.draw do
end
```

#### Resources
|        Prefix | URI Pattern                         | Controller#Action |
| ------------- | ----------------------------------- | ----------------- |
|     articles  | GET    /articles(.:format)          | articles#index    |
|               | POST   /articles(.:format)          | articles#create   |
|  new_article  | GET    /articles/new(.:format)      | articles#new      |
| edit_article  | GET    /articles/:id/edit(.:format) | articles#edit     |
|      article  | GET    /articles/:id(.:format)      | articles#show     |
|               | PATCH  /articles/:id(.:format)      | articles#update   |
|               | PUT    /articles/:id(.:format)      | articles#update   |
|               | DELETE /articles/:id(.:format)      | articles#destroy  |

#### Custom & Named Routes
```ruby
get '/all' => 'tweets#index', as: 'all_tweets'
<%= link_to "All Tweets", all_tweets_path %>
```

#### Redirect
```ruby
get '/all' => redirect('/tweets')
```

#### Root Route
```ruby
root_to: "tweets#index"
<%= link_to "All Tweets",root_path %>
```

#### Route Parameters
```ruby
get '/local_tweets/:zipcode' => 'tweets#index', as: 'local_tweets'
<%= link_to "Tweets in 32828", local_tweets_path(32828) %>
```



