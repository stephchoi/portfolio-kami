---
layout: post
title: Bloccit
thumbnail-path: "img/bloccit_home.png"
short-description: A Rails application that allows users to create forums and comment on them.

---

{:.center}
![]({{ site.baseurl }}/img/bloccit_home.png)

## Explanation

Bloccit is a forum based app made with Ruby on Rails modeled after Reddit. It allows an individual user to make posts, vote on, share and save links and comments. And like with Reddit, with Bloccit users can favorite individual posts and up/downvote them depending on whether they liked them or found them relevant to the topic.

## Features
#### Seeding Data

As I created this website in order to familiarize myself with Ruby models, a lot of different aspects that could have been easily supplemented with Ruby gems I ended up creating by myself.

![]({{ site.baseurl }}/img/bloccit_topic.png)

For example, for the random seeded data to create various topics, posts, comments, and users, I made a file that created random strings like so:

{% highlight ruby %}
def self.random_word
     letters = ('a'..'z').to_a
     letters.shuffle!
     letters[0,rand(3..8)].join
end

def self.random_name
    first_name = random_word.capitalize
    last_name = random_word.capitalize
    "#{first_name} #{last_name}"
end

def self.random_email
    "#{random_word}@#{random_word}.#{random_word}"
end
{% endhighlight %}

I created a `random_word` by shuffling an array of the alphabet and stringing together a random number of letters from the array. Two `random_word`s could be capitalized to create someone's name or they could be combined with "@" to form an email as shown above. Or a handful of words joined form a much larger `random_sentence` and a couple of sentences form a `random_paragraph`.

These are then used to create most of the content shown on the application posts and topics.

{% highlight ruby %}
5.times do
    User.create!(
        name: RandomData.random_name,
        email: RandomData.random_email,
        password: RandomData.random_sentence
    )
end
users = User.all

15.times do
    Topic.create!(
        name:        RandomData.random_sentence,
        description: RandomData.random_paragraph
    )
end
topics = Topic.all

50.times do
   post = Post.create!(
        user: users.sample,
        topic: topics.sample,
        title:  RandomData.random_sentence,
        body:   RandomData.random_paragraph
    )

   post.update_attribute(:created_at, rand(10.minutes..1.year).ago)
   rand(1..5).times { post.votes.create!(value: [-1, 1].sample, user: users.sample) }
end
{% endhighlight %}

#### Voting
![]({{ site.baseurl }}/img/bloccit_post_page.png)

Besides filling the site with data, a key feature in making the website seem more like Reddit is the voting and ranking system. Bloccit posts when first created have a ranking of zero and based upon whether other users like the post or don't the ranking can go up or down.

{% highlight ruby %}
def update_vote(new_value)
   @post = Post.find(params[:post_id])
   @vote = @post.votes.where(user_id: current_user.id).first

   if @vote
       @vote.update_attribute(:value, new_value)
   else
       @vote = current_user.votes.create(value: new_value, post: @post)
   end
end
{% endhighlight %}

Whenever a user clicks the up arrow, the ranking or `post.value` increases by 1 and whenever a user clicks the down arrow, it decreases by 1. So the more people that like a post, the higher the ranking of the post. It's an easy way to sift through popular content on the topic of choice.

#### User Authentication

![]({{ site.baseurl }}/img/bloccit_user.png)

Bloccit does not use any gems, like Devise or AuthLogic, to authenticate users. Instead the user model and authentication system for signing up and signing in users was created through Rails, using BCrypt to encrypt the passwords in the database.

{% highlight ruby %}
validates :name, length: { minimum: 1, maximum: 100 }, presence: true
validates :password, presence: true, length: { minimum: 6 }, if: "password_digest.nil?"
validates :password, length: { minimum: 6 }, allow_blank: true
validates :email,
          presence: true,
          uniqueness: { case_sensitive: false },
          length: { minimum: 3, maximum: 254 }

has_secure_password
{% endhighlight %}

The user model requires a name, an email, and a password, stored in a `password_digest` attribute which is required when using the `has_secure_password` method provided by BCrypt. The one-directional hashing algorithm turns the plaintext string into an unrecognizable string of characters to provide a layer of security for the user accounts should a hacker gain access to the database.

## Reflections

As a way to learn the ins and outs of Ruby and Rails, I think this application was very thorough. It had several different models with different relations to each other, some that I generated through Rails itself and others that I created models, views, controllers and migrations separately. Had I used user authentication gems and data seeding gems I might not have truly understood what I was actually using and how they worked.

However, at the same time, this application probably wouldn't be great to use publically.
Because the user authentication system is completely self-made, it's impossible to tell whether an email actually exists or not, so there's a possibility of having users spam the forums. I don't have a system for sending out a password reset email  so if a user forgets their password they won't be able to access their account at all.

There's always room for improvement in any project in any case. I'll just have to go through all of these concerns later on if I decide to develop the application more.
