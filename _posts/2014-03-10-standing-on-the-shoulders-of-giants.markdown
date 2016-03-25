---
layout: post
title: "Standing on the Shoulders of Giants"
date: 2014-03-10 17:40:41
comments: true
---

Programming today is completely different than programming 30 years ago. Even using Ruby is world’s different than using lower-level programming languages. Take a look at old FORTRAN and Assembly manuals and you quickly begin to appreciate the syntactic beauty of the Ruby programming language.

Another way programming is different today is that we have access to so many more tools. These tools allow us to rely on “solved problems” and create incredibly useful applications.

One of the ways that we have access to these tools is through API’s. An API, or Application Program Interface, is a way for users to interact with your application programmatically. While most average consumers are used to working with graphical user interfaces, an API is a way for programmers to use their application in their own programs. We can then use another company’s API to build our own program.

One API that particularly intrigued me was Twilio. Twilio is a company that offers an API to send text messages and make/receive phone calls. This is a perfect example of standing on the shoulders of giants. Imagine if you had to implement that functionality yourself. It could take months to develop that functionality and only THEN would you be able to start developing your application. Twilio has made an incredibly challenging programming problem almost trivial.

I started with Twilio by signing up for an account on their website. After verifying my own phone number, I was given another phone number that would be used for the application. Before actually working with API, I wondered whether there was a Ruby gem available. Luckily enough, there was, which made interacting with the API incredibly easy. Anytime you need to work with an API, always check whether a Ruby wrapper has been written. Again, standing on the shoulders of giants. Rely on the ingenuity of others to further your own creativity.

After signing up for the account, you receive two authentication tokens to authorize the API and tie it to your account. To get started, I needed to require the twilio-ruby gem so I quickly ran

```bash
$ gem install twilio-ruby
```
at the command line. After installation, all I had to do was require the gem at the beginning of my file and I was good to go. If I was building a Sinatra or Rails application, I would have added the gem to my Gemfile and used Bundler to manage the gem dependencies.

The tricky part of using an API to manage your access while hiding your authorization tokens from others. There are several methods for doing this. Rails 4.1, which has not been officially released yet, will implement a new method that allows you to create a secrets.yml file. Other methods include loading the keys as environment variables directly on your production server. Whatever you choose, you want to be absolutely certain that you never put your API keys on Github or any other public repository. This would enable others to use your account illicitly.

Since I wasn’t using Rails and simply wanted to create a Ruby file to interact with the API, I placed the API keys directly in my application. Don’t worry, it won’t be going on Github. The twilio-ruby API gives you access to a set of modules and classes. You can set up your application by creating an instance variable and setting it to a new instance the class below.

```ruby
@client = Twilio::REST::Client.new(account_sid, auth_token)
```
Notice that the Twilio::REST::Client took two arguments on initialization; these are the two API keys provided to you by Twilio. Once you had your new instance variable set up, it is super easy to start texting people.

```ruby
@client.account.messages.create(
 :from => "+13473826253",
 :to => "+19175555555",
 :body => "Hey there! It’s me from the Flation School!"
)
```
The “from” phone number is the number provided by Twilio that relates to your account. The “to” phone number can be any number that receives a text, although as long as you’re using the trial version of the API, you can only text numbers that you have verified.

Twilio can also make outgoing phone calls, receive phone calls and receive text messages. It can perform almost any action relating to phones that you would need for your application. Please keep in mind that Twilio charges a nominal fee for its API based on usage. Consult its website for more details.

Twilio, and other programs like it, allow programmers today to accomplish incredible things in a fraction of the time it would have taken previously. Entire businesses that never existed have been built using Twilio’s technology. As I learn more and more about programming, I am in awe of the capabilities that are available to us. I couldn’t be more excited for the applications to come.
