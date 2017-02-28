if you choose a number not on the list it errors out
you can pass a param to a `#times` block and it will be the index (instead of `reply_counter` in `Tweet#set_replies`)
you can call `#set_replies` in `initialize`
i'm having a visceral hate reaction to that counter instance variable
If you select a tweet and check replies, then you go back and select the same tweet and check replies, it increases the number shown by 10-15 tweets each time
if you check out kanyewest, then enter "more", then select 9 ("With love and respect, Kanye West" - the number will probably change at some point), it displays 14 tweets and then errors out at 15. Says

```
lib/screet/reply.rb:16:in `content': undefined method `text' for nil:NilClass (NoMethodError)
```

which probably means it counts one past where it should have, and tries to call `text` on that `nil` iteration
happens with his #2 tweet if you ask for replies too, on the 14th reply this time
also happens with his #15 tweet, on the 15th reply again

should reduce the number of times you send a request to twitter--I'd make a scraper class, grab the user page you want, store it--this is your only actual HTTP request until you want to view a specific tweet's replies. Build all the tweets on the page into `@tweets` on `User`, as individual `Tweet`s, but only scrape the original doc, don't `open` another URL. All the info for an individual tweet will be there in the main page. Then, when a user requests replies, you can build the replies by going to the tweet's specific URL (which you can set on a specific `Tweet` by using a selector on a specific tweet to get the link off of the time (why is the _time_ the permalink to a tweet, twitter? makes no sense):

```ruby
doc.get_some_tweet.css(".content .time a").href
```

Or do it your way, grabbing the tweet id and building the URL.

A tweet should be initialized with its specific node of the original doc tree, I think. Like if you take a User, and do

```ruby
def build_tweets
  doc.css("div.tweet").each do |tweet_node|
    @tweets << Tweet.new(tweet_node)
  end
end
```

Then, `Tweet` would have (I think) exactly the same methods as before and still work fine, but you wouldn't make multiple calls to Twitter! And `@url` would be set based on what I mentioned earlier:

```
class Tweet
  attr_reader :doc
  attr_accessor :user

  def initialize(tweet_node)
    @doc = tweet_node
    @url = set_url
  end
  
  def set_url
    @url = doc.css(".content .time a").href
  end
end
```

And in your `User` class you can modify that `#build_tweets` method from earlier to establish the `user` for the tweet when you add it, taking care of the only other thing it was missing, I think:

```ruby
class User
  attr_reader :tweets, :doc

  def initialize(user_name)
    html = open("https://twitter.com/#{user_name}")
    @doc = Nokogiri::HTML(html)
    @tweets = []
  end

  def build_tweets
    doc.css("div.tweet").each do |tweet_node|
      new_tweet = Tweet.new(tweet_node)
      new_tweet.user = self
      tweets << new_tweet
    end
  end
end
```

Or you could be fancy:

```ruby
class User
  attr_reader :tweets, :doc

  def initialize(user_name)
    html = open("https://twitter.com/#{user_name}")
    @doc = Nokogiri::HTML(html)
    @tweets = []
  end

  def build_tweets
    doc.css("div.tweet").each do |tweet_node|
      tweets << Tweet.new(tweet_node).tap { |t| t.user = self }
    end
  end
end
```