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

## The new world

OK, let's try setting up our project again.

\#TODO: consider breaking this out of the main body..

```py
# app.py

import os

from datetime import datetime
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = os.environ["DB_URI"]
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
db = SQLAlchemy(app)


class RequestStore(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    created_at = db.Column(db.DateTime, nullable=False, default=datetime.utcnow)


@app.route("/")
def root():
    db.session.add(RequestStore())
    return "Hello, World"


if __name__ == "__main__":
    app.run()
```

Quick run-down of what this does. It's a "Hello, World" web application that counts the requests.

There are some things we should be aware of here:

- We use an environment variable called `DB_URI` to define our database.
- our dependencies are `flask`, `flask-sqlalchemy`, and whatever database driver we need.

To accomodate this, let's set up a Dockerfile:

```dockerfile
FROM python:3.7-alpine3.9

# defines where we're going to be storing our app code (or working from)
WORKDIR /app

# Base requirements
RUN pip install flask flask-sqlalchemy

# Our db driver
## dependency for the driver
RUN apk add postgresql-dev build-base
RUN pip install psycopg2-binary
## we remove as build-base as we only need it for compilation
RUN apk del build-base

# Add in our application code
ADD app.py .

# Document our environment variable
ENV DB_URI ''

# Run our web app
CMD python -m flask run --host 0.0.0.0
```

OK, we've got that set up. As a really quick test, we can try running with sqlite to verify
our docker is working correctly:

```sh
docker build -t hello-logged .
docker run --rm -it -p 5000:5000 -e 'DB_URI=sqlite:////tmp/test.db' hello-logged
```

You can verify in your browser by visiting `localhost:5000`.


### What's that command

#### build
`docker build -t hello-logged .`

This will build you a docker image, using your Dockerfile. It will run your commands, copy your files,
and give you the a bundle so you can easily execute your app.
`-t` tags the image, so you can launch it using a pretty name.

#### run
`docker run --rm -it -p 5000:5000 -e 'DB_URI=sqlite:////tmp/test.db' hello-logged`

`docker run` does what it says on the tin. It'll run an image as a container.

`--rm` will clean up resources after you stop running.

`-it` allows you to interact with the running container.

`-p 5000:5000` will expose port 5000 of the container to your machine.

`-e 'DB_URI=sqlite:////tmp/test.db'` sets an environment variable within the container.

`hello-logged` is the tag you defined during build.

### But what about my database?

Now we have the foundation of building and running a docker image, we can start using services.

Docker gives us a very useful tool for managing services, called `docker-compose`.

Let's define a config:

```yml
# docker-compose.yml

version: '3.7'
services:
  db:
    image: 'postgres:11-alpine'
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: db_name

  app:
    build: .
    environment:
      DB_URI: "postgresql://postgres:password@db/db_name"
      FLASK_DEBUG: 1
    ports:
      - '5000:5000'
    depends_on:
      - db
```

Let's go over what this means.

We're launching 2 services, named `db` and `app`.

`db` uses a Postgres 11 image, and sets up the username, password, and database.

`app` will build your Dockerfile, set up your ports, and link itself to `db` by depending on it.

This can be launched with the following command: `docker-compose up`.

You can now visit your browser on `localhost:5000`.

#### How do we get the DB_URI?

the construction of `DB_URI` is as follows:

`postgresql://`: This tells our app that we're dealing with a Postgres database.

`postgres:password`: This is our username and password, note that it's the same in the `db` service

`db`: This is the hostname of the database server. Note how it's the same as the `db` service.

`db_name`: this is the database name, defined in the `db` service.


### Adapt for development

You've probably noticed that changing your files locally doesn't get reflected up to Docker.
Let's fix that.

This is as simple as adding 2 lines to your compose file:

```yml
services:
  app:
    volumes:
      - '.:/app'
```

By adding this to your `app` service, any modifications you make on your machine will
be copied over your docker container.
