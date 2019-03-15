# The Docker Obstacle Course

Docker is fantastic.
It lets you quickly spin up your code with all accompanying services, and removes most of the “it works on my machine” nonsense.
If you do it right, it even lets you simulate a production environment locally, so you know things will “Just Work”.

Not only is it great for development, it also greatly simplifies deploying your applications to production.

It is my most used and loved tool.

## The old world

Let's say you have a python web application.

If you wanted to run this locally, a common way was to install all of the dependencies on your host machine, then install your python version, set up a virtual environment (not a virtual machine), pip install your requirements, then maybe by some miracle it will work first time.

Odds on, you'll also have a database, such as MySQL or PostgreSQL.

The common way people would go about using these in development was:

- Install a database to their local machine
- Go through the hassle of setting up all the configs so your user can access the root user
- Pray that the setup instructions they wrote work for everyone else in the team (they won't)
- Move back to using SQLite without considering that their dev is now nothing like the production environment.

