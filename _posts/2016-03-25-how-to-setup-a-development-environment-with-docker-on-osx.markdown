---
layout: post
title: How to setup a development environment with Docker on OSX
date: 2016-03-25 15:08:41
comments: true
---

Setting up a new project can be an absolute nightmare. If there are many interlocking parts, the codebase will make assumptions about how your system is set up. New developers can take days to get a proper development environment going. Even scarier, many companies rely on "special" production boxes to deploy their code. These servers have been hand-configured over the years and the loss of them could be catastrophic for getting the app running properly. Orchestration management tools like Ansible, Salt, Chef and Puppet were introduced to solve this problem in Production. However, this doesn't solve the fundamental problem that the developer's environments don't always match what is deployed to Production. This leads to frustrating problems where a piece of code works on one person's machine but breaks when deployed. To solve this, you need to ensure that you have a consistent environment and this is what Docker allows you to do. For those who are unfamiliar, Docker is a container system that allows you to guarentee a consistent environment whether you're developing on your local machine or deploying to Production.

I've heard a lot about Docker over the past couple years but I've never really made an effort to get a proper dev environment running until now.

Getting Docker setup on OSX is a little challenging. Docker was meant to work on Linux (although an officially supported Mac version is in [beta](https://beta.docker.com/)). Fortunately, a project called [boot2docker](https://github.com/boot2docker/boot2docker) exists to give us a lightweight Linux container to use on OSX. This enables us to develop our applications on Docker with Macs and deply the same instance to Production.

However, boot2docker has one significant downside. Since it relies on VirtualBox, a virtualization machine, it uses a system called vboxsf to sync your local files into your docker instance. vboxsf is slow, sometimes buggy and breaks the internals of file system watchers so live reload functionality will not work. To solve this, and to create a pleasant Docker install and usage experience, we're going to use a project calls [docker-osx-dev](https://github.com/brikis98/docker-osx-dev). Under the hood, docker-osx-dev uses rsync to make sure that your files match between your dev environment and your Linux container.

Let's get started. Note, I'll be showing my prompt in the examples. Type everything AFTER the dollar sign.

This tutorial assumes you have Homebrew installed. If you don't run the following command first:

```bash
# Only needed if you don't have Homebrew
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
The following three commands will download docker-osx-dev, make it executable and run the installer.

```bash
$ curl -o /usr/local/bin/docker-osx-dev https://raw.githubusercontent.com/brikis98/docker-osx-dev/master/src/docker-osx-dev
$ chmod +x /usr/local/bin/docker-osx-dev
docker-osx-dev install
```

docker-osx-dev will set you up with Docker, boot2docker and other dependencies needed to get started on your machine. Note that at the end of the install there will be a line to source files. Make sure you copy and run that command so your environmental variables are setup properly.

The heart of what docker-osx-dev does is set up syncing between your local file system and the Docker container. The easiest way to get started is to start the watcher and then bring up a container. First create a new folder you'll use for testing. I'll set this up in my home directory.

```bash
$ mkdir ~/docker_test
$ cd ~/docker_test
```

Now you can run docker-osx-dev in your test folder. This will start the Boot2Docker VM and start file watching on the folder that it lives in. You can also pass a specific folder with the ```-s``` command. Leave it running.

```bash
$ docker-osx-dev
```

Now in a separate tab, cd back into the test directory and run the following command. We're going to create a test file to ensure that the syncing is working properly and then we are going to run the Docker container. This will start up a small Alpine Linux container and give us shell access. Note this command was taken directly from the docker-osx-dev [README](https://github.com/brikis98/docker-osx-dev#readme)

```bash
$ cd ~/docker_test
$ echo "Hello from the local" > docker.txt
$ docker run -v $(pwd):/src -it --rm gliderlabs/alpine:3.1 sh
```

Now that you're in the Docker container you can verify everything worked by checking whether our test file has been synced (note that your prompt has changed. Type everything after the $ sign)

```bash
/ $ cat /src/docker.txt
Hello from the local
```

Let's break down the command above.

```docker run``` is used to run an existing container image. This can be a container that lives on our filesystem or one that lives on Docker Hub, similiar to Github for containers. You can also host an internal hub reigstry if you'd like. The ```-it``` argument told docker to create a new terminal for us to manage the container, ```--rm``` removes the container on exit. The heart of what we're looking for is done using ```-v```. Here, we're telling Docker to map a volume on our local box to the Docker container. The command above sets up a sync between the current directory and /src on the container. docker-osx-dev takes care of the actual syncing for us. Make sure to kill the tab running docker-osx-dev once you're done before we move on to the next steps.

Now that we know Docker is set up for us, how do we use this in a local development environment? To demonstrate, we're going to use the Docker training app to run a small Python server using Flask. However, instead of using a pre-built image, we're going to build an image ourselves and run it.

First, download the training app from the Github repo and start docker-osx-dev to make sure the file sync is set up. Then open a new tab and cd back into the github repo directory.

```bash
$ git clone git@github.com:docker-training/webapp.git ~/webapp
$ cd ~/webapp/webapp
$ docker-osx-dev
```

```bash
# In another tab
$ cd ~/webapp
```

We're going to make a small modification to the Flask file. By default, Flask does not auto-reload files on change. Let's make a quick edit to our webapp/app.py to enable reload. modify the last line to the following:

```python
app.run(host='0.0.0.0', port=port, debug=True)
```

Now let's take a look at our Dockerfile. A Dockerfile is a series of instructions to Docker that tells it how to build an image. You can see it below:

```bash
$ cat Dockerfile

FROM ubuntu:14.04
MAINTAINER Docker Education Team <education@docker.com>
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y -q python-all python-pip
ADD ./webapp/requirements.txt /tmp/requirements.txt
RUN pip install -qr /tmp/requirements.txt
ADD ./webapp /opt/webapp/
WORKDIR /opt/webapp
EXPOSE 5000
CMD ["python", "app.py"]
```

We're not going to break down everything above but we can highlight the main points.
FROM is almost always the first line in a Dockerfile. It tells Docker where to pull the base version. Here we are telling it to start with a standard Ubuntu 14.04 distro. Next, we setup our environment to run Python. The RUN command will execute on the Docker container. Once the container is setup to run Python, we'll use the ADD command to copy files from our local machine to the Docker container. This copies in the requirements.txt and then runs pip so all the dependencies are in place. We're going to then ADD the entire webapp folder to ```/opt/webapp``` where it will live on the container. WORKDIR will change the current working directory and EXPOSE will open the port specified. Finally, CMD will concatenate and execute the string specified when the container is first run.

To package this up into it's own image, we'll use the ```docker build``` command. From the root of the Github project run the following:

```bash
cd ~/webapp
docker build .
```

If this is the first time running docker build it might take a couple minutes. At the end, you should see a line that says "Successfully built XX" where XX refers to the unique id given to your image. In my case it was 3b4532063fd7 but yours will be different. Make sure to substitute that in the commands below. Now we're going to run the image we built with the following command. The key to enabling Docker development is to map the local directory where the files lives to the directory where they will be run from on the docker container. We're also going to open port 5000 on the local and map that to the Docker port 5000 so we can see changes.

```bash
docker run -v ~/webapp/webapp:/opt/webapp -d -p 5000:5000 -it 3b4532063fd7
```

If all goes well, you should be able to go to http://dockerhost:5000 in your local browser and see "Hello world!" Note that docker-osx-dev set up dockerhost in our /etc/hosts to make it easy to access running containers.

Now the real magic begins. docker-osx-dev is running in the background and syncing the files. We set up a volume mount from the development directory to ```/opt/webapp```, where the files are running our of on the Docker container. Open up the app.py on your local machine once again and edit line 8 so it looks like the following:

```python
provider = str(os.environ.get('PROVIDER', 'from my local machine'))
```

Save the file. If all goes well, you should see the changes reflected when you go to http://dockerhost:5000. The changes were synced to ```/opt/webapp``` by docker-osx-dev and Flask saw there were updates and reloaded the dev server.

We've barely scratched the surface of what Docker can do. Docker really shows it's strengths when you link different containers together to really have a unified development and production environment. Hopefully the instructions above allow you to get started working with Docker and shipping reliable applications.
