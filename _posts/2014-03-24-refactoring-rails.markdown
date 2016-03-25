---
  layout: post
title: "Refactoring Rails"
date: 2014-03-24 17:40:41
comments: true
---

Recently at the Flatiron School we were tasked with creating a simplified eBay clone using our newfound knowledge of Ruby on Rails. My team and I diligently worked through the test spec provided to create a fully functioning auction app complete with user accounts, auctions and a whole host of validations.

However, when I looked back at our code, it was a total mess. We built our app to “work” but didn’t pay any attention to how maintainable the codebase would be down the road. We coupled our logic where it should have been decoupled, placed model logic in our controllers and a whole host of other Rails Sins. I wanted to work through one section of our codebase and refactor it so the code is more readable and maintainable for the future. Since we had a testing spec, I was able to refactor the code and not worry about breaking the application. This post was inspired by Ben Orenstein from Thoughtbot who speaks regularly on refactoring your code (and is the only person who is more obsessed with I am with aliasing their shell commands).

I searched my codebase for the most heinous example of bad code and found it in my Bids Controller. The Bids Controller creates new bids and associates them to existing auctions. Honestly, I hesitated sharing the code because its so awful. Ugh. Here goes nothing.

```ruby
def create
    @auction = Auction.find(params[:auction_id])
    @bid = @auction.bids.build(amount: params[:amount],  bidder_id: session[:user_id])
    if @auction.bids.length == 1 || (params[:amount].to_i > @auction.highest_bid.amount_in_dollars)
      if @bid.save
        flash[:notice] = "You are the current high bidder!"
        redirect_to auction_path(@auction)
      else
        render auction_path(@auction)
      end
    else
      if params[:amount].to_i == 0 || params[:amount].include?(",")
        flash[:notice] = "Amount is not a number"
        redirect_to "/auctions/#{@auction.id}"
      else
        flash[:notice] = "Amount is too low!"
        redirect_to "/auctions/#{@auction.id}"
      end
    end
  end
end
```
Wow, we have a lot of work ahead of us.

The first two lines are fairly standard to CRUD applications. I find the current auction by its ID and then associated a new bid for that auction. The first line is actually a really common in Rails. The URL (through the params hash) provides us the current ID so our Create controller action can find the proper Auction from the database. This is so common that its probably much smarter to set this instance variable before each of the controllers that we will need it instead of repeating the logic in every controller.

At the top of my controller I’m going to create a Rails macro that’s going to run before every controller action that we specify.

```ruby
class BidsController < ApplicationController
  before_action :set_auction, only: [:show, :update, :destroy]
  ...
  private

  def set_auction
    @auction = Auction.find(params[:auction_id])
  end
end
```
I was able to DRY up my code by finding the auction every time one of those three controllers actions are run. One line down.

I’m actually fairly comfortable with the second line of code. Most Rails developers would use mass assignment in this case but since I have my actions aliased in my model and there’s only two attributes to set, I don’t mind setting them manually here. Additionally, this is the only place in my bids controller where create new instances of auction so I don’t have any repeating code.

The rest of the method is one gigantic nested if statement. Heinous.

Let’s break this down line by line so we can get a feel for what we’re dealing with here.

```ruby
if @auction.bids.length == 1 || (params[:amount].to_i > @auction.highest_bid.amount_in_dollars)
```
This code will evaluate to true when this is either the first bid is made or the bid placed is highest that the current bid (through a quirk in the code the length of the bids array will be at least one because the instance variable is written in the build statement but not written to the database yet).

My first instinct is to put this logic in its own method. There’s just too much to keep track of in on statement of logic.

```ruby
def first_bid?
  @auction.bids.length == 1
end

def higher_bid?
  (params[:amount].to_i >  @auction.highest_bid.amount_in_dollars)
end
```  
Then we can refactor the first part of the code to

```ruby
if first_bid? || higher_bid?
```
Nice.

Let’s move down to our next piece of “validation” code

```ruby
if params[:amount].to_i == 0 || params[:amount].include?(“,”)
```

The code above checks whether the input is a string or whether there were invalid characters submitted. To be honest, this kind of validation sounds like it should go in the model. We want basic float validation without having to resort to figuring out every possible invalid combination. Luckily, Rails provides a really convenient macro that we can use in our model:

```ruby
validates_numericality_of :amount
```

This macro will prevent the bid from being saved if the amount submitted is not a number.

Looking over the remainder of our code, we create a new flash notice for our users on new lines. We can slim this down by passing that notice to the redirect so it can be rendered in our views.

```ruby
redirect_to auction_path(@auction), notice: "You are the current high bidder!"
```
We also seem to be redirecting to the same view multiple times. By pulling that information out of our the if statement, we can DRY up our codebase. Additionally, since we’re performing our own validation, we can comfortably call our save method directly in our controller. Finally, we can use some Rails magic to simplify some of our method calls. Our final controller codebase looks something like this (apologies for the odd formatting errors):

```ruby
  before_action :set_auction, only: [:show, :update, :destroy]
  
  def create
    @bid = @auction.bids.build(amount: params[:amount], bidder_id:                  session[:user_id])
    if first_bid? || higher_bid?
      @bid.save
      redirect_to @auction, notice: "You are the current high bidder!"
    else
      redirect_to "/auctions/#{@auction.id}", notice: "Amount is too low!"
    end
  end

  def first_bid?
    @auction.bids.length == 1
  end

  def higher_bid?
    (params[:amount].to_i > @auction.highest_bid.amount_in_dollars)
  end
```

  Our codebase is now decoupled and our logic is much easier to follow.

  The main reason why we were able to change our code confidently is because we had a passing test spec before we started. Writing tests for your code is so important because it gives you the flexibility to make your code as best as it could be. By practicing TDD, you can confidentially refactor your codebase and not worry about breaking your application.

  Happy refactoring!
