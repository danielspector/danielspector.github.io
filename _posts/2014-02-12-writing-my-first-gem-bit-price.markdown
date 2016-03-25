---
layout: post
title: "Writing my first gem - Bit Price"
date: 2014-02-12 17:40:41
comments: true
---

At the Flatiron School we’ve been taught that the structures and syntax of programming are just that, structures. Just like a building is only as valuable as the people you put in it and what you use it for, the structures of programming are only as valuable as the programs you make with them.

Over the last couple days we covered diverse topics such as web scraping and RubyGems. I decided to implement both of these skills in one project in order to reinforce what we’ve covered in lectures.

Web scraping is a broad topic that I’ll definitely be writing about more thoroughly in future posts, but for now let’s just define it as “gathering data from a website and bringing it into your program.” Anything that can be viewed through a browser can (theoretically) be scraped. In the Ruby community, most web scraping is done through a gem called Nokogiri.

A RubyGem, or gem for short, is a packaged piece of code that can be passed around to different computers. If I write a ruby file that I want to share with the world, I want to make it as convenient as possible for others to access and utilize my code. There’s a fantastic project called RubyGems that allows people to easily publish their gems for others to use. In this post, I’m going to explore how I published my first gem and what improvements I still need to make.

First, before publishing a gem we’re going to assume that you have a working Ruby file that you want to share with the world. The gem can be as simple as printing “Hello World” to the screen or complicated as a Ruby on Rails project. For this project, I decided to write a small scraper that uses Nokogiri to grab the Bitcoin buy and sell prices from Coinbase’s website.

## Bit_Price: My First Gem
Below is the actual Ruby code that is run when you execute the gem I created. The file is called bit_price.rb and is packaged up with the gem. If you’ve never seen Ruby code (or any code at all) this might look a little unfamiliar to you but that’s OK, you don’t have to understand this code to learn how to create a gem.

```ruby
  require "nokogiri"
  require "open-uri"
  def bit_price
   page = Nokogiri::HTML(open("https://coinbase.com/charts"))
   prices = page.css("div.page-header h2.pull-right strong").text.split("$").reject(&:empty?)
   puts
   puts "The Current Bitcoin Prices Are:"
   puts "Buy Price: $#{prices[0]}"
   puts "Sell Price: $#{prices [1]}"
   puts
   puts "Prices provided by www.coinbase.com/charts"
  end
```

The Nokigiri gem scrapes the prices from the Coinbase site using their HTML tags and CSS selectors. For a quick recap, every website has “tags” which tells your browser how to render the page and CSS selectors which allow you to style your content. Coinbase actually names their CSS selectors for buy and sell prices with identical names, which meant that when I tried to get the text of the name I was given both the buy and sell prices. To get around this, I had to split the text into an array using the “$” as the delimiter and delete the blank string that it added to the array.

If none of that made sense to you, that’s OK. Just keep in mind that your gem can do anything that you want it to, mine just happens to grab two numbers from a website and prints them to a screen.

Getting Started with RubyGems
To get started with RubyGems, I did a Google search and landed on the official instruction page for RubyGems. The guide is very helpfully written and made it easy to push up the gem.

The first thing I had to was reorganize my code. I called my main folder “bitcoin_price”. The RubyGems website expects that you will have a folder at the root of your project (the main folder where your project files are) called lib/ where your the Ruby file you just created will live. Additionally, you will need a .gemspec file in your main project directory with the Gem name so let’s create that now. To create a blank file just run:

```bash
$ touch bit_price.gemspec
```

Once you’ve reorganized your files, your structure should look something like this:

```bash
% bitcoin_price

├── bit_price.gemspec
└── lib
 └── bit_price.rb
 ```
Next you should open up the blank .gemspec file in your favorite text editor (I use Sublime Text) and edit it with the following code, substituting your information where appropriate.

```ruby
Gem::Specification.new do |s|
 s.name = "bit_price"
 s.version = "0.0.2"
 s.executables << "bit_price"
 s.date = "2014-02-12"
 s.summary = "Bitcoin Prices from Your Terminal"
 s.description = "Get the Bitcoin price easily from your terminal"
 s.authors = ["Daniel Spector"]
 s.email = 'danielyspector@gmail.com'
 s.files = ["lib/bit_price.rb"]
 s.homepage =
 "http://rubygems.org/gems/bit_price"
 s.license = 'MIT'
 s.add_runtime_dependency "nokogiri", [">=0"]
end
```
One thing to note is the version name. If you previously pushed your gem up to RubyGems with an identical version number and then make changes, it will reject your update. Each push has to have a unique version number. You'll notice that there’s an executables file right under version. We’ll cover that in a bit. Mostly you want to fill out the information for your gem including your personal information such as author and email. You’ll also see that we’re adding a runtime dependency for Nokogiri. This will install Nokogiri on the user’s computer if it’s not installed already.

We’re going to create one more file which will make it much more convenient for people to access our gem. We need to create an executable file in a bin/ folder simply called “bit_price” that will allow people to run the code from the command line. For my project I created the following structure:

```bash
% bitcoin_price

├── bin
│ └── bit_price
├── bit_price.gemspec
└── lib
 └── bit_price.rb
 ```
In bin/bit_price I created a simple file that called our bit_price.rb file and executed it.

```ruby
#!/usr/bin/env ruby
require "bit_price"
bit_price
```
Notice that I named my method bit_price in my ```bit_price.rb``` file which has to match the executable call shown above. This will be equivalent to people running

```bash
$ ruby bit_price.rb
```
on their command line.

Now we’re all set up and we’re ready to test our gem!

To setup my gem and allow me to test it, I ran the following two commands at the root of my project directory.

```bash
$ gem build bit_price.gemspec
$ gem install ./bit_price-0.0.2.gem
```
Note that the version number has to match what you put in your bit_price.gemspec file.

Now you should be able to run your file simply by typing ```$ bit_price```.
If that doesn’t work, you might have to enter irb, require the gem, and then run it.

```bash
$ irb
>> require 'bit_price'
=> true
>> bit_price
```
Assuming you get the output you’re looking for, you should be ready to distribute your gem.

## Distributing Your Gem: Getting your gems up on RubyGems
Now we’re going to get our gems up on RubyGems so anyone can find and download it. First, navigate to the RubyGems and sign up. Remember the handle and password that you enter — it will be important in a minute. Once you sign up, open a new tab in your browser and go the following link:

https://rubygems.org/api/v1/api_key.yaml

You’ll be brought to an authentication screen. Type in the handle/username that you set up before. This should automatically download a file called api_key.yaml file to your computer. You’re going to need to move this file to your .gem folder and rename it to CREDENTIALS with the following commands. Assuming your download defaulted to your Downloads folder, run the following:

```bash
$ mv ~/Downloads/api_key.yaml ~/.gem
$ cd ~/.gem
$ mv api_key.yaml CREDENTIALS
```
Once you’ve run those commands you should be all set up to push your gem to RubyGems! Simply run the command below and you should see the following output.

```bash
$ gem push bit_price-0.0.2.gem
```
which should (hopefully) result in:

```bash
Pushing gem to RubyGems.org…
Successfully registered gem: bit_price(0.0.2)
```
Take a small break. You’ve accomplished a lot. Once you come back, your gem should be available for everyone to see on the RubyGems website. A simple way to ensure that you’ve successfully published your gem is to type:

```bash
$ gem list -r bit_price
*** REMOTE GEMS ***
bit_price (0.0.2)
```
Once you’ve confirmed that your gem has been successfully uploaded, anyone with the RubyGems package manager installed (it comes standard with Ruby) should be able to run

```bash
$ gem install bit_price
```
and then

```bash
$ bit_price
```
to run the program. Now all anyone has to do is to type your method name and see the output of of your program from anywhere in their terminal.

```bash
$ bit_price
The Current Bitcoin Prices Are:
Buy Price: $676.69
Sell Price: $674.66
Prices provided by www.coinbase.com/charts
```

Great job! Your gem is now live for anyone to download. Make sure to put your code up on Github so others can comment and improve on your work. The power of open source is incredible.

## A To-Do list for this gem
A couple quick caveats and some things to consider when writing your own gem.

It has no tests and you should always test your code in a separate spec file before you publish. I like RSpec, it’s nearly magical.
It depends on Nokogiri to run. Since I already have Nokogiri installed, I had no issues with the program but other people might have issues installing Nokogiri when they run your program. If anyone has any feedback on this I would love to hear it.
Make sure your gem name isn’t taken already before you try to publish! Your push to RubyGems will not work if the name is taken.
You should add a license and a README to the file so you protect yourself and be able to guide new users with how to run your program.
I had a lot of fun learning gems and I hope you found this useful. You can find the completed source code at https://github.com/danielspector/bitcoin_price

As always I would love your feedback on the post. Follow me on Twitter [@danielspecs](https://www.twitter.com/danielspecs).

A huge thanks to Sam Schlinkert for reviewing this post and fixing my sometimes inelegant choice of words.

Thanks for reading!
