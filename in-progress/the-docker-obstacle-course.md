# The Docker Obstacle Course

Docker is fantastic.
It lets you quickly spin up your code with all accompanying services, and removes most of the “it works on my machine” nonsense.
If you do it right, it even lets you simulate a production environment locally, so you know things will “Just Work”.

Not only is it great for development, it also greatly simplifies deploying your applications to production.

It is my most used and loved tool.

## The old world

Let's say you have a python web application.

If you wanted to run this locally, a common way was to install all of the dependencies on your host machine, then install your needed python version (often needing to compile it), set up a virtual environment (not a virtual machine), pip install your requirements, then maybe by some miracle it will work first time.

Odds on, you'll also have a database, such as MySQL or PostgreSQL.

The common way people would go about using these in development was:

- Install a database to your local machine
- Go through the hassle of setting up all the configs so your user can access the root user
- Pray that the setup instructions you wrote work for everyone else in the team (they won't)
- Move back to using SQLite because now it's all on fire

## What is docker

Docker is a tool that helps you manage containers.

A container is a program bundled with all the parts it requires.

When I say "all the parts", I mean everything. In most cases, it amounts to everything but the kernel in a normal Linux distribution.

These are then isolated from the host machine, so the container can't tamper with the host, and the host doesn't inadvertently break anything within the container.

### This sounds familiar..

Let's make something clear. Docker isn't a new concept. Nor are containers.

Something people did was use virtual machines. You could spin them up, install everything the same way you would a production server, and it'd #JustWork.
You could even use the same provisioning scripts that your production environment used.

This would give them a segregated area to run the software without causing any collisions with their development machine.

This method is fairly costly though, as you're running operating systems next to each other.

Containers run under your host OS, and relies on cheaper techniques to enforce isolation.
In most cases, the performance penalty of running your software in a container is negligible, and there are event linux distributions that work on the premise of everything being in a container.
