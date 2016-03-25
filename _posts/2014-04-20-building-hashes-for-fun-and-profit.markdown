---
layout: post
title: "Building Hashes for Fun and Profit"
date: 2014-04-20 17:40:41
comments: true
---

We’re wrapping up the semester at the Flatiron School and the school organized mock technical interviews for us so we could get some practice in a live coding environment before going on actual interviews. My mock interviewer was [Ed Weng](https://twitter.com/wengzilla), a great developer and all-around super nice guy. I wanted to discuss the programming challenge that Ed presented to me so others can learn and comment on it.

NOTE TO CURRENT FLATIRON STUDENTS: I will be discussing the details of the mock interview from Ed. If you have not yet had your interview and do not want to know the problem that’s discussed stop reading now.

The problem that I was tasked with completing was correctly parsing a YAML file. Ed sent me a link to a git repository with a YAML file and a README. The instructions were to parse the YAML file and construct an API-like interface so its contents could be easily accessed. These would be accessible using Ruby’s bracket notation as well as a JavaScript object-like dot-notation. I would need to define methods that would allow me to parse the data using the following method calls.

```bash
data.product.first.sku
```

The YAML file that Ed provided looked like a shipping invoice and contained attributes that would need to be accessed.

```yaml
invoice: 34843
date   : 2001-01-23
billto: &id001
    given  : Chris
    family : Dumars
    address:
        lines: |
            "458 Walkman Dr.
            Suite #292"
        city    : Royal Oak
        state   : MI
        postal  : 48046
shipto: *id001
product:
    - sku         : BL394D
      quantity    : 4
      description : Basketball
      price       : 450.00
    - sku         : BL4438H
      quantity    : 1
      description : Super Hoop
      price       : 2392.00
tax  : 251.42
total: 4443.52
comments: >
    "Late afternoon is best.
    Backup contact is Nancy
    Billsmer @ 338-4338."
```
The first hurdle to jump was accessing the YAML file. I had to require YAML at the top of my program and then call YAML.load to actually parse the file. Since I was not passing straight YAML into the YAML.load method I needed to call File.open in order to grab the contents of the file. At the beginning, my file looked like this:

```ruby
require 'yaml'

data = YAML.load(File.open('shipping.yml'))
```

Next, I needed a way to access the actual data. YAML.load provides a Ruby-interface for the data contained in the YAML file. At Ed’s suggestion, I moved everything into a class to get all the benefits of object orientation. I initialized the class with an instance variable and set a reader for that variable so I could call it in my program.

```ruby
require 'yaml'

class ShippingHash
  attr_reader :hash

  def initialize(hash)
    @hash = hash
  end
end

data = YAML.load(File.open('shipping.yml'))
```

The one downside to this approach, and the obstacle that I still haven’t overcome is that in order to get to the actual data you need to make an explicit call to the reader at the end of the method. This will be discussed further below.

Now that I had a class, it was time to implement the dot-notation calls for the ShippingHash object. To do this, I used method_missing. Method_missing is a part of BasicObject, the base class for all Ruby-classes. When a method call is made, Ruby uses a complex method-lookup chain in order to find the method that you are referring to. If it can’t find the method, it makes a call to method_missing right before letting you know that no method was found. If you definte a set of parameters using method_missing, you can give Ruby instructions on what to do if the method that was called is not found.

Method_missing always takes an argument, which refers to the value that was called but was not found. Leveraging that, I created a method_missing definition that would always make a call to a key in the hash that we created above.

```ruby
def method_missing(x)
  @hash["#{x}"]
end
```

There are several obvious problems with this approach. The first is that ANY method not found in this class will try to call that method as a key on our hash. To fix this, we need to call super on our method_missing to allow it to continue up the method-lookup chain and return a ```NoMethod Error```. For now, I’m ok with leaving this issue as I develop the solution but I’ll come back to it in a little bit.

The second problem that we have is that once we’ve accessed the hash for the first time, we lose the ability to make further calls to that hash. For instance the following code will return what we want:

```bash
data.product
```
However, if we try to chain additional method calls to keys further down the nested hash, the errors that will pop up will tell us that those methods don’t exist for the Hash class. Our method_missing stops at the first call. To solve this, I created new ShippingHash instances for every method_missing call. Our method now looked like:

```ruby
def method_missing(x)
  ShippingHash.new(@hash["#{x}"])
end
```

Now we’re getting somewhere. We’re able to make method calls on instances of the shipping class. Assuming the class was properly instantiated and the methods that we are chaining are keys in the hash that we created, we should be able to return an instance of the ShippingHash class containing the data that we’re looking for. However, we’ll still run into a roadblock. Above, we were tasked with solving the problem to this method call:

```bash
data.product.first.sku
```

If you look carefully, ```.first``` is not a key in the hash, it is calling the first item in the array that the receiver returns. If we look closely at the YAML file we can see that the product key is actually an array of nested hashes. To solve this, we will need to redefine the first method within the scope of our class to return new instances of the ShippingHash class.

```ruby
def first
  ShippingHash.new(@hash[0])
end
```

Excellent, we’re almost there. The last problem we need to tackle is that we only want to call method_missing if we meet certain conditions. Otherwise, we want it to continue up the method-lookup chain.

What condition are we trying to meet? We only want to call method_missing if we called a key in the hash that we created. If this were a single-level hash, we would only need to check whether the method call passed to method_missing was a key in the hash. Otherwise, we would want it to continue up the method lookup chain.

```ruby
def method_missing(x)
  if @hash.keys.include?("#{x}")
    ShippingHash.new(@hash["#{x}"])
  else
    super
  end
end
```

However, the hash we’re dealing with is a nested hash so we need a way to access all the keys in the hash. Suprisingly, Ruby does not provide a convenient way to grab all the nested keys in a hash so I reopened the Hash class and wrote a method to put all the keys into a new array

The method creates a blank array to hold the keys of the hash. It then loops through each key. If the key that’s being looped through is actually a hash itself, it makes a recursive call to the method that stops when it reaches a regular key and pushes that key into the newly created array.

```ruby
class Hash
  def nested_keys
    keys_array = []
    keys.each do |key|
      if self[key].is_a?(Hash)
        keys_array << self[key].nested_keys << key
      else
        keys_array << keys
      end
    end
    keys_array.flatten
  end
end
```

Now that we have a way to grab all of the keys of the hash, we can use it in our method_missing to only call our function if it matches the pattern that we specified.


Now, the following command will return an instance of the ShippingHash class containing the data we want.

```bash
data.product.first.sku
```
The problem that remains is that if we want to return the actual data we need to append the reader (.hash) to the end of the method call. To me, this seems unintuitive and I would love to be able to automatically append the reader to the end of a any method call to method_missing but I haven’t had any luck in figuring out how to do this. Alternatively, another approach might be to metaprogram new instance methods rather than using method_missing, which would give us greater flexibility in defining our method calls.

The final program, with a little cleaning up, looks like this:

```ruby
require 'yaml'

class ShippingHash
  attr_reader :hash
  
  def initialize(hash)
    @hash = hash
  end

  def method_missing(x)
    @hash.nested_keys.include?("#{x}") ? ShippingHash.new(@hash["#{x}"]) : super
  end

  def first
    ShippingHash.new(@hash[0])
  end

end

class Hash
  def nested_keys
    keys_array = []
    keys.each do |key|
      if self[key].is_a?(Hash)
        keys_array << self[key].nested_keys
      else
        keys_array << key
      end
      keys_array.flatten
    end
  end
end

data = ShippingHash.new(YAML.load(File.open('shipping.yaml')))
puts data.product.first.sku.hash
```

Please note that I did not actually implement all of this during the mock technical interview. I got stuck defining the recursive function to grab all of the keys of a hash and put that off for a later time while I implemented the rest of the program.

I learned a lot from my mock interview and I feel much more prepared as I begin interviewing. I want to thank Ed for his time and the Flatiron School for arranging the interview.

As always, I would love to hear any feedback as well as any advice you can provide on the code above. Feel free to hit me up on Twitter [@danielspecs](https://www.twitter.com/danielspecs).

Thanks for reading!
