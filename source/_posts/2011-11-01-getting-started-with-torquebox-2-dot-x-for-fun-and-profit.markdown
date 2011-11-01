---
layout: post
title: "Getting Started With Torquebox 2.x for Fun and Profit"
date: 2011-11-01 16:30
comments: true
categories: Ruby jRuby EC2 CloudComputing
---
I am a big fan of jRuby and I have been aware of the [Torquebox Application Server](http://www.torquebox.org) for some time now but until recently never found the time to take it on a test drive on Vagrant and on Amazon's EC2. Because I wasn't familiar with the JBoss Application Server on what Torquebox is based on I ran into some problems and hopefully this blogpost will enable you to have a Rails 3.1 test application up and running a lot quicker than I did.

Getting started on Vagrant
--------------------------

Ryan Bates of the famous [Railscasts](http://www.railscasts.com) featured [Vagrant](http://vagrantup.com) on one of his last episodes. In essence it's a way to try out your production set-up for your application without the hassle to start up a server instance on any cloud provider. It is also possible to provision your instances with tools like chef but that's not what this blogpost is about.

For a quick overview what Vagrant offers I suggest you go watch the [screencast](http://railscasts.com/episodes/292-virtual-machines-with-vagrant?autoplay=true) first. I had problems to start up the box Ryan uses in his screencast, but with [another one](http://www.vagrantbox.es/26/) everything worked as expected. After downloading the box setting running vagrant ist pretty easy. Just go to your app's directory and type:

``` bash
vagrant init ubuntu-1104-server-i386
vagrant up
vagrant ssh
```

On the server
-------------
From this point on the steps to getting torquebox to run on EC2 and Vagrant are exactly the same, that's the point of Vagrant after all. On EC2 you have to start an Ubuntu instance of course. I usually use the [ alestic ](http://alestic.com/) AMIs.

First we update our sources:
``` bash
sudo apt-get update
```
I wanted to install jRuby with rvm so I had to install git and ruby first to run rvm ([Ruby Version Manager](http://beginrescueend.com/)). Additionally we have to install some kind of jdk to install jruby. I choose openjdk-6. Openjdk-7 should work as well I guess. See [ openjdk.org ](http://www.openjdk.org) for Installation instructions but actually it's just another package that you install with apt-get.
``` bash
sudo apt-get install git-core ruby openjdk-6-jre
```
With that we are all set to install rvm and jruby.

``` bash
bash < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)
```
After rvm is installed we have to reload our bashrc to be able to use rvm.
``` bash
source .bashrc
```
Now to install jruby
``` bash
rvm install jruby
```
After the installation is completed we want to use jruby and we will create a global gemset that torquebox will use (see the official [ Torquebox blog ](http://torquebox.org/news/2011/04/26/ashevillerb-preso/)).
```bash
rvm use jruby
rvm gemset create torquebox
rvm use jruby-1.6.5@torquebox --default
```
The default flag ensures we are using jruby with the torquebox gemset everytime we log into the machine.

Torquebox 2.x is available as a gem and we want to make use of this. Installing Torquebox consums a lot of system memory so we have to specify that the install has access to more memory than it has with the default settings.
```bash
jruby -J-Xmx1024m -S gem install torquebox-server --pre --source http://torquebox.org/2x/builds/LATEST/gem-repo
```
Instead of LATEST you could specify any build.
```bash
jruby -J-Xmx1024m -S gem install torquebox-server --pre --source http://torquebox.org/2x/builds/591/gem-repo
```
With LATEST you will just get the last CI-Build of Torquebox. You have to take this into account when using our Gemfile later on.

Well and that's about everything there is to installing torquebox. 

Preparing your application for deployment on Torquebox
-----------------------------

You have to take to ensure your application is deployable to Torquebox.
Torquebox provides templates for this (you can read more about that [here](http://torquebox.org/2x/builds/html-docs/web.html#rails)). The template resides in your torquebox-gem's folder. You can set a $TORQUEBOX_HOME variable temporarily to make it easier to access the template.
``` bash
TORQUEBOX_HOME=$(torquebox env torquebox_home)
rails new myapp -m $TORQUEBOX_HOME/share/rails/template.rb
```
To adapt an existing application please
refer to the [torquebox documentation](http://torquebox.org/2x/builds/html-docs/).

Deploying and running an application on Torquebox
---------------

Deploying a (torquebox ready) application on Torquebox couldn't be
simpler. Just change into your application's directory and run:

``` bash
  bundle install
  torquebox deploy
  torquebox run
```

I had several problems to get my test application to run with torquebox. Every single one was my own fault and was due to my lack of understanding of the underlining JBoss-Server or different behaviour of Rails on Ubuntu compared to MacOS.

First of all Ubuntu does not come with a Javascript-runtime which is needed to run a Rails 3.1 application. Ryan talks about that in his screencast but his workaround to install 'therubyracer'-gem won't work for a jruby-environment. There is actually a special javascript-gem for jruby, called 'therubyrhino'. Specify this in your app's Gemfile instead of 'therubyracer'.

``` ruby
  group :linux do
    gem 'therubyrhino'
  end
```

Accessing your application from localhost (Vagrant)
---------------

Vagrant provides a way to access your VirtualBox-Instances from your localhost. Ryan talks about it on his Railscasts-Episode. You can configure Vagrant to forward ports from your virtual-instance to localhost in your Vagrantfile, Vagrant creates with the vagrant init command in your app's directory.

JBoss and Torquebox are accessible through port 8080 as default so we have to specify port-forwarding for this port:
```
config.vm.forward_port "torquebox", 8080, 8080
```
This should make your vagrant instance accessible through the address localhost:8080. You can specify any port you want
```
config.vm.forward_port "torquebox", 8080, 3000
```
Would make port 8080 of your vagrant-instance accessible throug localhost:3000 in your browser.

Unfortunately this won't work for Torquebox. You have to bind Torquebox to 0.0.0.0 to make it accessible via the above-mentioned Vagrant feature.
```
torquebox deploy <your app directory>
torquebox run -b 0.0.0.0
```
Now everything should work as expected.

Deploying your application to EC2
-------------------------
The steps to get running on EC2 are nearly the same as on Vagrant. The only thing you have to remember is that port 8080 does not accept inbound connections on EC2's default security group. You have to open this port and you have to open port 22(ssh) to be able to log into the instance if your using a custom security group.

When running the application from EC2 you also have to bind it to
0.0.0.0. Binding to an Elastic-IP won't work.

Other major problems
----------------------
I don't know if I am the only one who was not aware of the fact that though on MacOS Ruby's require statement is NOT case sensitive but on Ubuntu it is. I tried to require the ruby-geocoder in my test-application and 
```ruby
require 'Geocoder'
```
was fine on my development machine but on Ubuntu hell broke lose because rails wasn't able to locate my geocoder gem though it was installed. On Ubuntu you really have to require case sensitive.

```ruby
require 'geocoder'
```
Keep that in mind and spare yourself hours of digging around.

