---
layout: post
title: "Folderize Me Capt'n"
date: 2014-02-24 17:40:41
---

All groups will eventually agree on a code, a set of rules that they can use to communicate with each other. To outsiders, these codes may look inscrutable, but to those on the inside its almost a way of life. Rubyists around the world have largely agreed to a file structure when creating Ruby programs and scripts. When I first encountered the file structure I was lost and confused, wandering around from file to file not sure of what I was looking like. After spending some time working in and creating ruby file structures, I’m more comfortable than ever. It’s my hope that after reading you’ll feel the same.

The standard Ruby file structure looks something like this:

```bash
% project_root
├── bin
│ └── generate
├── config
| └── environment.rb
├── lib
| └── project_file.rb
├── spec
| └── spec_helper.rb
└── .rspec
└── Gemfile
└── Gemfile.lock
└── README.md
└── Rakefile
```
The file structure exists as a way to organize our code and keep it DRY. DRY stands for “Don’t Repeat Yourself” which means we want to avoid duplication like the plague. The file structure allows each folder to have a specific job, making debugging easier. Let’s explore each of folders above and hope to tease out some more meaning.

Bin — The bin directory is usually reserved for executable files. Ad you can see above, the bin directory has one file in our example, generate, which we would make executable. Most Rubyists will create an executable file that actually runs their code, rather than having the code run on its own. This keeps the execution of our code separate from the formulation of our code, allowing us to debug when we find problems.

Config — The config directory is used for configuration files. You can have as few or as many config files as you’d like. Most of the time its best to keep all of your configuration in one file which we called environment.rb. In here we “require” all of our files which lets each part of the program talk to one another other. Just placing files in the same folder is not enough, we need to explicitly tell our files that they can talk to each other. Additionally, we would include any libraries we need as well as set up our database structure. You can think of the Config folder as your Ruby version of Grand Central Station — A central meeeting point so everyone can get to where they need to get.

Lib — This is where the heart of our program will live. The lib folder will often have many subdirectories, each relating to a different part of the program. Additionally, while not pictured above, there may additional folders in the lib directory which relate to abstracted code from our program such as modules. In Rubyland, these would usually go in a directory called “concerns”.

Spec — This is where all of our tests will live. I’m using the RSpec testing framework but you can use any framework that you’re comfortable with. The spec folder will often contain dozens of testing files, each testing a separate part of your code. Most testing files are named after the file that they are testing along with _spec at the end of the filename. For example, if we were testing a file from our lib directory called artist.rb, we would probably call our matching test file artist_spec.rb. We generate the spec_helper.rb file that you see above by running

```bash
$ rspec --init
```
in our project root, which also creates a .rspec file. These two files allow us to configure RSpec and tweak to our needs so we can test our files. Most importantly, we need to make sure that our testing files know about all the other files in the file structure so we would make sure to require the environment.rb file we discussed earlier at the top of our spec_helper.rb file.

Gemfile &amp; Gemfile.lock — Gems are a really convenient way for Rubyists to pass around and share code. As I discussed in a previous blog post, Gems are often indispensable to the function of ourprogram and we want to make sure that we are including the gems we want and managing them properly. A really clever program called Bundler allows us to manage the gems we’ll be needing in our program. To get set up with a Gemfile in your program run

```bash
$ bundle init
```
which will create the Gemfile that you see in the directory above. Once you’ve added the gems that you need in your program, run

```bash
$ bundle
```
and the Gemfile.lock file will be created which makes sure that your program is running the correct gems at all times. Any additional gems required can be added to the Gemfile. It is very important to run the bundle command as seen above every time you make a change to your Gemfile.

README.md — This is optional for your program, but highly recommended if you will be sharing your code with others. Many programmers spend so much time with their own programs that they have an intuitive sense of how it operates. However, others who stumble on to your code may feel quite differently. It’s important to properly document the functionality of your program so it will be accessible to as many people as possible.

Rakefile — Finally, we arrive at our Rakefile, which allows us to to run custom tasks for our program. While a full discussion of Rake is beyond the scope of this post, be aware that many Ruby programs rely on Rake tasks to run and set a lot of their functionality. These Rake tasks are set in the Rakefile.

An Easier Way — Folderize
Wow! That’s a lot of folders! When getting started on a Ruby program, it takes a long time to create all the directories and files, making sure that they talk to each other. Luckily, there’s another way. To automate the process, I created a gem called Folderize which creates the directory structure discussed above. To install it, go to your terminal and type

```bash
$ gem install folderize
```
Once the gem is installed, make a new directory that your Ruby project will live in. Instead of spending time creating the folders and files listed above, just run:

```bash
$ folderize
```
After a brief pause, you should have a complete working file structure like you see above. Happy coding!

Please keep in mind that the above is a general guide to creating a Ruby file structure and should not be regarded as a canonical source. There can be an infinite amount of variation from what’s listed above but by using Folderize you will be off to a great start.

The code for the gem can be found at Github. Pull requests are welcome. As always, I appreciate any feedback you may have about the blog post or the gem itself. You can reach me on Twitter @danielspecs
