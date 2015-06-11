# Rails Notes

## DB CURD

### Hash
* Hash Recipe: ```variable = { key: value }```
* Read recipe: ```variable[:key] = value``` or ```variable.key = value```

### Create
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

### Read
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
### Update
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

## Models
```ruby
# app/modles/tweet.rb
class Tweet < ActiveRecord::Base

end
```

### Validations
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

  validates :status, presence: true,
                     uniqueness: true,
                     numericality: true,
                     length: { minimum: 0, maximum: 2000 },
                     format: { with: /.*/ },
                     acceptance: true,
                     confirmation: true
end

>> t = Tweet.new
=> #<Tweet id: nil, status: nil, zombie: nil>
>> t.save
=> false
>> t.errors.messages
=> {status:["can't be blank"]}
>> t.errors[:status][0]
=> "can't be blank"
```

### Relationships
```ruby
class Zombie < ActiveRecord::Base
  has_many :tweets
end

class Tweet < ActiveRecord::Base
  belongs_to :zombie
end

>> ash = Zombie.find(1)
=> #<Zombie id: 1, name: "Ash", graveyard: "Glen Haven Memorial Cemetery">
>> t = Tweet.create(status: "Your eyelids taste like bacon.", zombie: ash)
=> #<Tweet id: 5, status: "Your eyelids taste like bacon.", zombie_id: 1>
>> ash.tweets.count
=> 3
>> ash.tweets
=> [#<Tweet id: 1, status: "Where can I get a good bite to eat?", zombie_id: 1>,
    #<Tweet id: 4, status: "OMG, my fingers turned green. #FML", zombie_id: 1>,
    #<Tweet id: 5, status: "Your eyelids taste like bacon.", zombie_id: 1>]

>> t = Tweet.find(5)
=> #<Tweet id: 5, status: "Your eyelids taste like bacon.", zombie_id: 1>
>> t.zombie
=> #<Zombie id: 1, name: "Ash", graveyard: "Glen Haven Memorial Cemetery">
>> t.zombie.name
=> "Ash"
```










