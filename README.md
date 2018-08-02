# ripple-vagrant
Development version of the Ripple Stack deployed via Vagrant (for cross-platform usability) and the docker-compose provisioner (for simplicity and minimal configuration)

This repository contains a Vagrantfile and a Docker Compose file which together automate the setup of a working Ripple Stack.

I've started here with some explanation of why this development setup has been chosen, but if you want to just get on with installation, [click here to jump to the Getting Started section](#getting-started)

## Why use Vagrant and Docker?
Docker on its own is not completely at home on all platforms, Windows in particular being a problem.

In order to make the development experience the same on all platforms, we've wrapped the development environment in a headless virtual machine (which is actually how the Ripple stack developer documentation suggests to do it anyway). Virtual machines work on all platforms and the Ubuntu guest that is created then gives us a consistent and reproducible development environment on all platforms.

Inside this VM we use [Docker](https://www.docker.com/what-docker) and [docker-compose](https://docs.docker.com/compose/) to orchestrate the creation of all of the Ripple Stack components inside the VM, and these are all preconfigured to be able to communicate with each other. This process has reduce the setup complexity of the Ripple Stack from a 25-page document down to a few lines of commands.

### What is Vagrant?
Vagrant is an automation layer that allows programmatic setup of a Virtual Machine, using any provider (eg Virtualbox, QEMU, VMWare etc) and on any platform. It means we can state which 'base box' we want to use (in this case it's Ubuntu 16.04, but it could be anything) and then specify what setup needs to be applied to that base box, eg networking, forwarded ports, shared drives, etc. If you look at our [Vagrantfile](Vagrantfile) you can see fairly easily what is going on, it's a nice clean syntax.

### What is Docker?
Docker is a tool which greatly simplifies the use of Linux Containers, which you can envisage as a 'super-lightweight virtual machine' which interfaces directly at Linux kernel level, doing away with much of the inefficiency of traditional Virtual Machines. Because they are so lightweight and performant, the technology has  enabled individual software products like web servers or databases to be packaged and distributed in Docker containers, which have all the right dependencies pre-packaged in the container. This is great when you're setting up a complex environment and, for example, your stack has components which need *different* Java versions, or Ruby versions, or whatever. Each Docker container would contain it's *own* correct Java or Ruby version, and there is no dependency conflict.

### What is Docker Compose?
Docker Compose is another tool which comes with Docker, which enables programmatic setup and networking of multiple Docker containers. In this case we're using Docker Compose to provision the Ubuntu VM once Vagrant has done creating it. Docker Compose creates an internal network inside our Virtual Machine, and automatically plugs all the Docker containers into it.

#### Why not just do all the setup in the Vagrantfile and let Vagrant do it all?
Good question. You totally could do this and it would also work. Vagrant can be instructed to set up each of the Docker containers in the Vagrantfile. However, Docker Compose is a much more portable tool which can be used in other contexts, so having a working `docker-compose.yml` for the Ripple Stack enables the exact same code to be used again for  configuration of a Docker-only setup, for Boot2Docker, even for live deployments if necessary and appropriate.

# Getting Started

**Prerequisites/assumptions:*
* I am assuming a Unixy platform (MacOS or Linux) in the commands below, but the steps are the same for all platforms.
* You should have some familiarity with the command line in your platform
* You should have a Virtual Machine provider installed already (eg Virtualbox, VMware etc)
* You will need a decent bandwidth Internet connection to download Vagrant, the virtual machine base box, Docker images, and all the NPM modules that are needed (this happens under the hood as each Docker container sets itself up)

1. Install Vagrant. Instructions for this vary depending on your platform, so please [go to the Vagrant website](https://www.vagrantup.com/downloads.html) for details

1. Navigate to wherever you are going to keep your development files, and create a folder to contain all the code for the Ripple Stack.
  `mkdir ripple`
  `cd ripple`

1. `git clone` this repository to get a copy of the files.
`git clone https://github.com/pacharanero/ripple-stack-vagrant-docker.git`

1. `git clone` the QEWD Microservices middleware component
1. `git clone` the Pulsetile React frontend component
1. `git clone` the EtherCIS Clinical Data Repository component

You should now have four subdirectories in your development directory, one for each of the Ripple Stack components, and one for this repo. The reason for separating them like this, is so that you could do local development work on any of those components and still preserve the Git history of each separate repo. If we put them all in one folder we can get Git chaos.

1. go into the ripple-stack-vagrant-docker repository

`cd ripple-stack-vagrant-docker`

1. Start Vagrant

`vagrant up`

That's it for setup. There will be reams of command line output scrolling past now, which is Vagrant creating the VM, downloading the base Ubuntu box, and setting up the VM. Then Docker Compose takes over to set up the numerous Docker containers, again there will be lots of command line output as this is done. Docker will automatically download all the containers it needs.

## Monitoring and Troubleshooting
To get inside your Vagrant Virtual Machine to see what's going on, type `vagrant ssh` into the command line (NB: you must be in the same directory that the Vagrantfile is in for this to work)

You can check that the Docker containers were all created and are on the correct ports by typing `docker-compose ps` or `docker ps` which lists all the running Docker containers in the VM. You should see an output a bit like this, showing the services, each with a named container, and port mapping etc.

![](images/docker-ps-docker-compose-ps.png)

To see the logs of **all** the different services all together, with nice colour highlighting for each service and timestamps, you can use `docker-compose logs -f -t` to connect to the logging output. (the switches `-f` and `-t` make the output 'follow' and 'timestamped' respectively. To disconnect without stopping the containers, just use Ctrl+C. You should see something like the following output:

![](images/docker-compose-logs-example.png)
