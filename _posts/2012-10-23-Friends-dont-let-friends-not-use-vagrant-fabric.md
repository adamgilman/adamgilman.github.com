---
layout: post
title: Friend's Don't Let Friends NOT use Vagrant and Fabric
categories : [blog]
---
You may have heard people talk about [Vagrant](http://vagrantup.com) or [Fabric](http://docs.fabfile.org/en/1.4.3/) before and how cool they are but, never actually got around to trying them out or understand WHY you should be using it.

Well, take a few minutes here and change how you develop with Python, in a good way. (Or any language for that matter)

Have you ever had the following problems?
* Need a different version of a package than you currently have
* Have specific version dependencies for a project
* Need to work with MySQL on OSX 
* You deploy on Linux but, develop on OSX
* Have components of your project than don't run on OSX

[virtualenv](http://www.virtualenv.org/en/latest/) and [virtualenvwrapper](http://www.doughellmann.com/projects/virtualenvwrapper/) help with some of these problems but, not all of them. That's not to say you shouldn't use virtualenv but, it's not a panacea for all your Python woe's.

For me, a big problem was the lack of parity in my development enviroment (OSX) and my deployment environments (Ubuntu) and bumping into MySQL library hell on OSX every once and a while. These are not unsolveable problems but, they chew up time and wasting time doing package configuration and dependency debugging isn't productive. I'd rather actual write code and produce products.

So just like [virtualenv](http://www.virtualenv.org/en/latest/) is a sandbox for your Python packages for a project, [Vagrant](http://vagrantup.com) is a sandbox for everything! I personally use [Vagrant](http://vagrantup.com) like [virtualenvwrapper](http://www.doughellmann.com/projects/virtualenvwrapper/) to swap back and forth between projects, just without the cool command line tricks.

Let's start an example project that does some web crawling and storing the results in redis (note: we aren't actually going to do any coding just development environment setup). We also won't cover the installation of fabric and vagrant some both are very straightfoward and their docs cover the basics quite well.

I've created an empty project on github and checked it out to a local (on my Mac) directory

	null-406c8f3f8469:vagrant-fabric-demo adamgilman$ ls -lh
	total 8
	-rw-r--r--  1 adamgilman  staff    39B Oct 23 17:08 README.md

Nothing but, the empty readme.

You could simply type *vagrant init* at the command line to create a new Vagrantfile but, I have a slimmed down version that I use to get started which I'll copy into my project directory

	null-406c8f3f8469:vagrant-fabric-demo adamgilman$ cat Vagrantfile 
	Vagrant::Config.run do |config|
	  config.vm.box = "lucid32"
	end

That's it. 3 line's and we have a brand new, fresh, no cruft or crumbs laden Virtual Machine ready to do. So let's start it up

	null-406c8f3f8469:vagrant-fabric-demo adamgilman$ vagrant up
	[default] Importing base box 'lucid32'...
	[default] The guest additions on this VM do not match the install version of
	VirtualBox! This may cause things such as forwarded ports, shared
	folders, and more to not work properly. If any of those things fail on
	this machine, please update the guest additions and repackage the
	box.

	Guest Additions Version: 4.2.0
	VirtualBox Version: 4.2.1
	[default] Matching MAC address for NAT networking...
	[default] Clearing any previously set forwarded ports...
	[default] Fixed port collision for 22 => 2222. Now on port 2200.
	[default] Forwarding ports...
	[default] -- 22 => 2200 (adapter 1)
	[default] Creating shared folders metadata...
	[default] Clearing any previously set network interfaces...
	[default] Booting VM...
	[default] Waiting for VM to boot. This can take a few minutes.
	[default] VM booted and ready for use!
	[default] Mounting shared folders...
	[default] -- v-root: /vagrant

I haven't upgraded my VirtualBox Guest utils recently so I get that error, and I have another Vagrant virtual box running (I usually have 2 or 3) so I have a port collision on 2222 but, it doesn't really matter. Vagrant handles it fine and I rarely ever have to worry about it, which is exactly what you want from a tool. 

I have a new machine up and running ready to be SSH'd into

	null-406c8f3f8469:vagrant-fabric-demo adamgilman$ vagrant ssh
	Linux lucid32 2.6.32-38-generic #83-Ubuntu SMP Wed Jan 4 11:13:04 UTC 2012 i686 GNU/Linux
	Ubuntu 10.04.4 LTS

	Welcome to Ubuntu!
	 * Documentation:  https://help.ubuntu.com/
	New release 'precise' available.
	Run 'do-release-upgrade' to upgrade to it.

	Welcome to your Vagrant-built virtual machine.
	Last login: Fri Sep 14 07:26:29 2012 from 10.0.2.2
	vagrant@lucid32:~$ uname -n
	lucid32

Now this is my favorite part about Vagrant, if you've never used it.

	vagrant@lucid32:~$ cd /vagrant/
	vagrant@lucid32:/vagrant$ ls -lh
	total 8.0K
	-rw-r--r-- 1 vagrant vagrant 39 2012-10-23 18:08 README.md
	-rw-r--r-- 1 vagrant vagrant 63 2012-10-23 18:12 Vagrantfile

In the /vagrant directory of my VM is my LOCAL filesystem on my Mac and it's a shared folder between the two systems. 

This means I can use my local (on my Mac) IDE's but, execute and test my code on an Ubuntu machine. The best of both worlds.

Now, I could start building out this Vagrant VM from the command line but, one of the nice things about Vagrant is the fact that I can destroy and rebuild them pretty cheaply but, I'd lose all my fun packages I've installed. What we really want is a reliable and repeatable way to build out machines. 

Enter [Fabric](http://docs.fabfile.org/en/1.4.3/). Fabric is a remote deployment tool for Python which is similar to [puppet](http://puppetlabs.com) or [Chef](http://www.opscode.com/chef/) but, it's a bit slimmer and it's also Python. You can use any of the above tools to configure your Vagrant or production machines, it's your choice. I'm a Fabric man myself. Some people like boxers, some like briefs... Some like boxer briefs!

I have a very basic fabfile that I use to start all my projects, just like my basic VagrantFile. This fabfile has a quick dollop of Vagrant specific code and my stubs for pip and apt-get which are the main tools I use for building out boxes which will be practially universal for everything I work on.

<script src="https://gist.github.com/3939877.js"> </script>

From the command line I can test out my fabfile quickly with 

	null-406c8f3f8469:vagrant-fabric-demo adamgilman$ fab vagrant:2200 uname
	[localhost] local: vagrant ssh-config | grep IdentityFile
	[127.0.0.1:2200] Executing task 'uname'
	[127.0.0.1:2200] run: uname -n
	[127.0.0.1:2200] out: lucid32


	Done.
	Disconnecting from 127.0.0.1:2200... done.

The only thing you have to watch out for in my fabfile is the requirement for the port number of the vagrant VM you are connecting to. I do this because I sometimes will be running 2 or 3 different machines for different purposes at the same time. I want to make sure I'm connecting to the right machine so I force the port as a parameter.

Now all I need to do is add onto the "build" function in my fabfile

	aptget(['build-essential', 'python-dev', 'redis-server'])
	mysqlserver("root-password")
	aptget('libmysqlclient-dev')

	pip(['MySQL-python', 'sqlalchemy', 'redis', 'requests'])

That's it... I now have a VM with MySQL, MySQL Python Tools, SQLAlchemy, redis-server, redis-py, and the requests library.

The best part about it is, I can destroy the VM, keep my local code, rebuilt the VM and start from scratch, as many times as I like!

	vagrant destroy && vagrant up && fab vagrant:2200 build

Same box... Fresh and clean.

When I'm ready to move to staging or to production? Add a new desitintion in fabric and fire away. 