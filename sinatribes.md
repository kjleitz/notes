# Sinatribes notes

## Overview

Each user has an account, which corresponds to a tribe

Each tribe has a chieftain

Tribes have:

- land
- population
	- chieftain
	- warriors
	- hunters
	- farmers
	- priesthood
- infrastructure
- technology
- farms
- mines?
- natural resources

You can interact with your own tribe:

- collect taxes
- build infrastructure
- build buildings
- buy land
- mine resources
- see happiness
- change tribe religion
- change tribe governance
- change name of tribe
- split tribe into two tribes if sustainable?

Tribes can interact with other tribes:

- trade
- raid
- make alliance through marriage?
- sabotage?
- learn from?

You can send messages between tribes:

- messenger person
- option to kill the messenger person?
- lie to other tribes to alter your perceived stats?

## Models (v1)

### User

- has a username
- has an email
- has a password
- has a tribe
- has many messages (tribe instead?)
- has many conversations (tribe instead?)

### Tribe

- belongs to a user
- has many resources
- has a population (maybe just a hash?)
- has many buildings
- has money
- has land
- has a religion

### Religion

- has a name
- has many tribes
- has a modifier

### Resource

- belongs to a tribe
- has a name
- has a modifier

### Population

- belongs to a tribe
- has a chieftain
- has warrior count
- has farmer count
- has priest count

### Building

- belongs to a tribe
- has a name
- has a modifier

### Message

- belongs to a conversation
- belongs to a tribe
- has a boolean "read" value
- has timestamps
- use as rough idea (rails, may not apply):

```ruby
class CreateMessages < ActiveRecord::Migration
  def change
    create_table :messages do |t|
      t.text :body
      t.references :conversation, index: true
      t.references :user, index: true
      t.boolean :read, :default => false
      t.timestamps
    end
  end
end

...

class Message < ActiveRecord::Base
  belongs_to :conversation
  belongs_to :user
  
  validates_presence_of :body, :conversation_id, :user_id
  
  def message_time
    created_at.strftime(“%m/%d/%y at %l:%M %p”)
  end
end
```

### Conversation

- belongs to a user (sender)
- belongs to a user (receiver)
- use as rough idea (rails, may not all apply):

```ruby
class CreateConversations < ActiveRecord::Migration
  def change
    create_table :conversations do |t|
      t.integer :sender_id
      t.integer :recipient_id
      t.timestamps
    end
  end
end

...

class Conversation < ActiveRecord::Base
  belongs_to :sender, :foreign_key => :sender_id, class_name: ‘User’
  belongs_to :receiver, :foreign_key => :recipient_id, class_name: ‘User’
  has_many :messages, dependent: :destroy
  
  validates_uniqueness_of :sender_id, :scope => :recipient_id
  
  scope :between, -> (sender_id,recipient_id) do
    where(“(conversations.sender_id = ? AND conversations.recipient_id =?) OR (conversations.sender_id = ? AND conversations.recipient_id =?)”, sender_id,recipient_id, recipient_id, sender_id)
  end
end
```

## Models (v2)

### User

- username
- email
- password / password_digest
- **has many** tribes

### Religion

A tribe cannot attack/raid other tribes of the same religion.

- name
- **has many** tribes

### Tribe

- name
- land
- money
- technology
- **belongs to** user
- **belongs to** religion
- **has one** population
- **has one** warriors, **through** population
- **has one** farmers, **through** population
- **has one** priests, **through** population
- **has many** messengers
- **has many** tribe_buildings
- **has many** buildings, **through** tribe_buildings
- **has many** resources
- inventory: a JSON string of resource names and amounts. SUCH A BAD WAY OF DOING THIS. But what else could you do? Maybe multiple indentical associations through tribe_resources join table, then count?

### TribeBuilding

- **belongs to** tribe
- **belongs to** building

### Building

- name
- price
- resource_amount
- resource_name
- **has many** tribe_buildings
- **has many** tribes, **through** tribe_buildings

### Resource

- name
- **belongs to** tribe
- **belongs_to** gift

### Population

- warriors
- farmers
- priests
- **belongs to** tribe

### Messenger

- message
- timestamps
- rejected (false)
- **has one** gift
- **belongs to** home, **foreign key** home\_id, **class name** 'Tribe'
- **belongs to** destination, **foreign key** destination\_id, **class name** 'Tribe'

### Gift

- money (0)
- warriors (0)
- accepted (false)
- **has_one** resource
- **belongs to** messenger

## Improvements

- Resource names should be symbols for better lookup - does that actually achieve better lookup with activerecord?
- need a better way to get wood
- lay out gift cell better
- db queries probably not scalable
- go through the code and review for ways to reduce clutter/extra code/inefficiency
- area for telling you if you have unaccepted gifts on the management page
- mobile friendly
- cap on resources
- balance game so resource cap isn't really necessary
- remember 10000 row limit
- pretty sure join tables don't contribute to rows? seems that way at least, maybe not
- deleted tribes should only remove links to tribe/tribe name in messenger activity rather than replacing a whole row

Visible

- fixed resource panel to right
- infrastructure third column delete, actions inside name column 
- new building row on top
- money above
- tips section fixed 
- move active tribe link to leftmost in nav
- little house fonticon house