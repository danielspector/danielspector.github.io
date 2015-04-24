---
layout: post
title: Preloading Data in Ember with Rails
date: 2015-04-23 11:41:46
---
**This is a subset of a talk I gave at Railsconf. You can find the slides to the full talk [here](https://speakerdeck.com/danielspector/crossing-the-bridge-connecting-rails-and-your-front-end-framework)**

Ember JS is a fantastic framework for working with, in the words its creators, large, ambitious applications. However, Ember suffers from the same problem that all client-side frameworks do. If you need to load a specific subset of your data that's not trivial to retrieve from the client, you're going to spend a lot of time on startup figuring out what data to display and making a series of API calls to your server. That's great if your users are on a super fast connection but for users on mobile or those who aren't lucky enough to have access to the fastest broadband possible, they're going to see loading bars and animated gif spinners. Not the best experience when someone is first coming to your page. To mitigate this, we'are able to use the power of Ember and Ember Data to preload our data so Ember will load up with the exact data that we need.

First, let's get setup with our Rails side. We're going to create a simple controller that's going to render out a list of Todos, similiar to TodoMVC. However, we're going to assume for a second that we have users who are currently signed in and we only want to show each user their specific todos.

Without this technique, the client-side would first have to figure out who is logged in, probably using a combination of cookies and API calls before them requesting the todos for a specific user. This is the simplest case. In large applications, you could be making tens of API calls before you're able to load your page.

We're going to solve this problem by loading the data that we need into Ember before it gets rendered out to the client. Let's setup a really quick Rails app.

```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  has_many :todos
end

# app/models/todo.rb

class Todo < ActiveRecord::Base
  belongs_to :user
end

# app/controllers/ember_controller.rb

class EmberController < ApplicationController
  def preload
    @todos = current_user.todos
  end
end

# config/routes.rb

Rails.application.routes.draw do
  get "/" => "ember#preload"
end
```

Here we have a simple model where a user has many todos. We set up a controller which we'll use to preload our data when we hit the root route. Now we need to figure out a way to preload the data into Ember. First, let's add a couple dependencies that will make working with Ember really easily.

```ruby

# Gemfile

gem 'active_model_serializers'
gem 'ember-cli-rails'
```

The first gem is ActiveModelSerializers which we're going to use to render out our JSON responses. AMS plays really nicely with Ember and we'll be using it to make sure that it renders the responses that Ember Data expects. ember-cli-rails is a gem that will make it really easy for us to use ember-cli in our Rails application.

Let's set up our serializer.

```
$ rails g serializer todo
```

```ruby
class TodoSerializer < ActiveModel::Serializer
  embed :ids, include: true
  attributes :id, :name
end
```


It's important that we have our root element set. This will allow Ember Data to process the response. What we're going to do is have Ember preload our data into the Ember Data store on startup. Client-side applications often need to make tens of API calls to show the right data on startup. We can do that processing on the server and just pass the data that we need straight to Ember. Doing this means we can avoid round trips back to the server and give our users a quicker experience.

```ruby
# app/controllers/ember_controller.rb

class EmberController < ApplicationController
  def preload
    @todos = current_user.todos
    preload! @todos, serializer: TodoSerializer
  end

  def preload!(data, opts = {})
    @preload ||= []
    data = prepare_data(data, opts)
    @preload << data unless data.nil?
  end

  def prepare_data(data, opts = {})
    data = data.to_a if data.respond_to? :to_ary
    data = [data] unless data.is_a? Array
    return if data.empty?
    options[:root] ||= data.first.class.to_s.underscore.pluralize
    options[:each_serializer] = options[:serializer] if options[:serializer
    ActiveModel::ArraySerializer.new(data, options)
  end
end
```

Let's take these two methods line by line. We're going to set up an array to hold our objects and prepare our data. We prepare our data to be passed to ActiveModelSerializer's Array Serializer. We need to ensure that we set the root element, which in our case will be "todos" and we need to set the proper serializer.

When we render our response we want it to look like the following:

```javascript
{
  "todos": [
    {
      "id": 1,
      "name": "Milk"
    },
    {
      "id": 2,
      "name": "Coffee"
    },
    {
      "id": 3,
      "name": "Cupcakes"
    }
  ]
}
```

Finally, we'll pass the array of arrays to our client-side via the window object in our layout. We'll call ```to_json``` on our array of arrays so it will be parsable by our Javascript.

```ruby
# app/views/layouts/application.html.haml
    = stylesheet_link_tag :frontend
    :javascript
      window.preloadEmberData = #{(@preload || []).to_json};
    = include_ember_script_tags :frontend
    %body
    = yield
```

Now lets generate our Ember application. First, we'll need to generate our config file to set up Ember-CLI to work with Rails. We're going to keep the Ember code in a folder called frontend and have it live at the root of our Rails application.

```
$ rails g ember-cli:init
```

```ruby
# config/initializer/ember.rb

EmberCLI.configure do |config|
  config.app :frontend, path: Rails.root.join('frontend').to_s
end
```

Now we can generate our Ember application. We'll skip git since Ember will live in our Rails application.

```
$ ember new frontend --skip-git

version: 0.2.3
installing
  create .bowerrc
  create .editorconfig
  create .ember-cli
  create .jshintrc
  create .travis.yml
  create Brocfile.js
  create README.md
  create app/app.js
  create app/components/.gitkeep
...
```
Ember will download all of the dependencies it needs using Bower and scaffold out a folder structure.

Once our fodlers are created, we'll generate a Todos resource. This will give us a model, a route and a template, along with the associated test files.

```
$ ember g resource todos
version: 0.2.3
installing
  create app/models/todo.js
installing
  create tests/unit/models/todo-test.js
installing
  create app/routes/todos.js
  create app/templates/todos.hbs
installing
  create tests/unit/routes/todos-test.js
```

The last two pieces that we'll need to make this work are an adapter and a serializer.

```
$ ember g adapter application
version: 0.2.3
installing
  create app/adapters/application.js
installing
  create tests/unit/adapters/application-test.js
$ ember g serializer application
version: 0.2.3
installing
  create app/serializers/application.js
installing
  create tests/unit/serializers/application-test.js
```

Ember Data uses adapters to specify how your client should communicate with the outside world. Ember ships with several adapters you can use. The FixtureAdapter is used for test data, the LSAdapter is used for localStorage and the RESTAdapter is used to commmunicate with JSON APIs. However, Ember ships with an extension of the RESTAdapter called the ActiveModelAdapter that is set up to work seamlessly with ActiveModelSerializers.

```javascript
// frontend/app/adapters/application.js

import DS from 'ember-data';
export default DS.ActiveModelAdapter.extend({
});
```

Now we'll set up a simple Todo model to hold our data.

```javascript
// frontend/app/models/todo.js

import DS from 'ember-data';
var Todo = DS.Model.extend({
  name: DS.attr('string')
});
export default Todo;
```

Once we have our model, we can set up our initializer. Ember, like Rails, we load any functions in its initializers folder before Ember is rendered on the client.

```javascript
// frontend/app/initializers/preload.js

export function initialize(container) {
  if (window.preloadEmberData) {
    var store = container.lookup('store:main');
    window.preloadEmberData.forEach(function(item) {
      store.pushPayload(item);
    });
} }

export default {
  name: 'preload',
  after: 'store',
  initialize: initialize
};
```

First, we're going to make sure that our store is intialized. Ember Data holds all of its data in stores that are globally accessible. Then we're going to iterate over each of arrays in our window object and push it into our store using ```pushPayload```. Ember Data will infer the model from the root of the JSON that we passed and save it in each model's respective object cache. Now all our the objects that we retrieved are already preloaded into our Ember application. Let's setup our router to map the root URL to our ```todos``` resource

```javascript
// frontend/app/router.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

export default Router.map(function() {
  this.resource('todos', { path: '/' }, function() {});
});
```

Then we'll set up our route. By convention, routes in Ember should be used to fetch the data that your controller and template will need to render.


```javascript
// frontend/app/routes/todos/index.js

export default Ember.Route.extend({
  model: function() {
    return this.store.all('todo')
  }
});
```

```html
// frontend/app/templates/todos/index.hbs
{% raw %}
<h2>Todo:</h2>
<ul>
  {{#each todo in model}}
    <li>{{todo.name}}</li>
  {{/each}}
</ul>
{% endraw %}
```

Often in Ember you'll find you prepre your controller by calling ```.find("model")``` which will make an API call to /model. However, we already have all the data we need and we can just call ```.all("model")``` to retrieve it out of the cache. Once we have that data we simply iterate over it in our tempalte using Handlebars and display it to our users.

There are enormous benefits to preloading your data like this. Often when you first load a site with client-side framework you'll see spinners and loading bars while the application loads all of its initial data. This can be a poor experience for your users, especially on mobile devices that might not be as fast. By preloading our data, it will be available to your users as soon as Ember loads.

I should note that this pattern isn't exclusive to Ember. You can preload your client using the window object with any framework. However, Ember Data gives us powerful abstractions that allows us to succintly and concisely present all the data that our client needs.

If you have any questions or want to discuss further, feel free to tweet at me [@danielspecs](http://www.twitter.com/danielspecs)

Thanks for reading!
