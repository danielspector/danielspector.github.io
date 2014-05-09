---
layout: post
title: Summing Consecutive Integers in Ruby
date: 2014-05-09  0:24:32
---

Let's solve a quick math problem using Ruby.

Given a positive integer, can that number be expressed as a sum of consecutive integers? For example, the number 3 would meet our criteria because 1 + 2 = 3. On the other hand, 4 would not. No combination of the integers below 4 can combine to equal 4 (1+2, 2+3, 1+2+3). Let's solve this in Ruby.

First, as always, let's write out a quick spec. The below examples use the RSpec test framework. Note that RSpec recently changed its syntax so the expectation ```be_true``` should now be written as ```be_truthy```. 

```ruby
describe "Sum of Consecutive Numbers" do
  it "returns true when the number can be summmed consecutively" do
    expect(consecutive_sum?(3)).to be_truthy
    expect(consecutive_sum?(10)).to be_truthy
    expect(consecutive_sum?(31)).to be_truthy
  end

  it "returns false when the number cannot be summmed consecutively" do
    expect(consecutive_sum?(4)).to be_falsy
    expect(consecutive_sum?(8)).to be_falsy
    expect(consecutive_sum?(32)).to be_falsy
  end
end
```

Our tests fail as expected and we can now move on to the implementation.

For this example we're not going to break out our logic into several classes as we want this to be as functional and simple as possible. Let's think through the problem before we implement it in code.

The first thing we should realize is that we can automatically disregard half the numbers that we'll encounter. If we start with 100, we can divide by 2 and throw out the larger half because 50 + 51 will always be greater than 100. Conversly, we can express in mathematical terms that n/2 + n/2+1 will always be larger than n.

Let's start by writing a method that will iterate through each number starting from 1 to n/2

```ruby
def consecutive_sum?(num)
  1.upto(num/2) do |i|
  end
end
```

We're going to need to keep track of our sum as we continue interating. Let's make that a local variable and give it an initial value of i since once an iteration is finished it means that we won't be considering that number again

```ruby
def consecutive_sum?(num)
  1.upto(num/2) do |i|
    sum = s
  end
end
```

Now for our logic. We're going to start a loop that will continue as long as the sum is less than our original number. Once the sum is greater than our original number, we will break out of the loop and return ```false```. We're going to increment our sum variable by ```i += 1``` and will explicitly return ```true``` if we've our sum is exactly equal to our original number. If we iterate through each number from 1 to n/2 and our sum has never matched our original number, we will return ```false```.

```ruby
def consecutive_sum?(num)
  1.upto(num/2) do |i|
    sum = i
    while sum < num
      i += 1
      sum += i
      return true if sum == num
    end
  end
  false
end
```

When we rerun our tests they pass! One improvement that we might want to make is wrapping our method with a quick check to ensure that we're being passed a positive integer greater than zero. Let's raise two ArgumentErrors. The first will be triggered if an a non-integer is passed to the method and the second will be triggered when a number less than zero is passed to the method. Let's spec out that behavior before we implement it.

```ruby
it "raises an error when passed a non-integer" do
  expect{consecutive_sum?("hello")}.to raise_error(ArgumentError, "Input must   be an integer")
end

it "raises an error when passed a negative" do
  expect{consecutive_sum?(-2)}.to raise_error(ArgumentError, "Input must be     larger than 0")
end
```

Note that I'm wrapping our expect statement in a lambda so RSpec will be able to catch the error rather than exiting.

After our tests fail as expected, we can implement the logic. We're going to wrap these two checks in a seperate method and call it from our main function.

```ruby
def check_input(num)
  raise ArgumentError, "Input must be an integer" unless num.is_a? Integer
  raise ArgumentError, "Input must be larger than 0" unless num > 0
end
```

And now all we have to do is call the error-checker from our original method.

```ruby
def consecutive_sum?(num)
  check_input(num)
  1.upto(num/2) do |i|
    sum = i
    while sum < num
      i += 1
      sum += i
      return true if sum == num
    end
  end
  false
end
```

And we're done! We've test-driven a small application that returns true or false if a number is the sum of consecutive integers. You can find the completed code on [Github](https://github.com/danielspector/consecutive_sum)

As always, I would love to hear your feedback. Drop me a note on Twitter or send me an email. Thanks for reading!