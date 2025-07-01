# Roadmap in "Docker only" souce

**Author:** Andrea Davanzo

**Last Updated:** 2025-07-01

This guide walks you through creating a simple Docker environment from the ground up. You will learn how to:

* Create a persistent Docker volume for data storage
* Set up a dedicated Docker network for container communication
* Run a PostgreSQL database container accessible from both other containers and your host machine
* Run a web server container connected to the database through the custom network

By the end, you’ll understand how to manually orchestrate multiple containers, manage networking, and persist data — all without relying on Docker Compose.
This hands-on approach helps build a solid foundation before moving on to more advanced tools.

## Setup the Docker environment

### Step 1: Create a Docker Volume

Volumes in Docker allow you to persist data outside of containers, so it remains safe even if containers are removed or recreated.

To create a named volume, run this command:

```bash
docker volume create roadmap-volume
```

This creates a volume called `roadmap-volume` that we will later use to store PostgreSQL data persistently.

You can list all volumes anytime with:

```bash
docker volume ls
```

### Step 2: Create a Dedicated Docker Network

Docker networks allow containers to communicate with each other securely and isolate them from other Docker containers.

Create a custom network called `roadmap-net` with this command:

```bash
docker network create roadmap-net
```

You can verify the network was created by listing all Docker networks:

```bash
docker network ls
```

Got it! Here’s the rewritten Step 3 including creating the Dockerfile for PostgreSQL 17 and building the container:

### Step 3: Create a Dockerfile and Run PostgreSQL 17 Container

#### 3.1 Create a Dockerfile for PostgreSQL

Create a file named `Dockerfile-db` with the following content:

```Dockerfile
FROM postgres:17

EXPOSE 5432
```

This uses the official PostgreSQL 17 image.

---

#### 3.2 Build the PostgreSQL Docker image

Run this command in the folder containing `Dockerfile-db`:

```bash
docker build --no-cache -t roadmap-db -f Dockerfile-db .
```

#### 3.3 Run the PostgreSQL container

Use the built image to start a container connected to the `roadmap-net` network, with persistent volume and exposed port:

```bash
docker run -d \
  --name roadmap-db \
  --network roadmap-net \
  -e POSTGRES_PASSWORD=roadmap \
  -v roadmap-volume:/var/lib/postgresql/data \
  -p 5432:5432 \
  roadmap-db
```

Now PostgreSQL 17 is running with password `roadmap`, data persisted, accessible from your host and other containers.


You can test the PostgreSQL container with one of the following methods:

**Option 1: From Host Machine (psql client installed)**

Run:

```bash
psql -h localhost -U postgres -p 5432 -W
```


**Option 2: Using Docker (psql via temporary container)**

If your host doesn’t have `psql`, run this:

```bash
docker run -it --rm \
  --network roadmap-net \
  postgres:17 \
  /bin/bash
```
then type

```bash
psql -h roadmap-db -U postgres -W
```

When prompted, enter the password: `roadmap`.


If it works, you’ll see something like this:

```
psql (17.x)
Type "help" for help.

postgres=#
```

When connected on PostgreSQL create a database

```sql
CREATE DATABASE roadmap
  WITH ENCODING 'UTF8'
  LC_COLLATE='C.UTF-8'
  LC_CTYPE='C.UTF-8'
  TEMPLATE=template0;
```


### Step 4: Clone the roadmap and roadmap-devres repositories

#### 4.1 Clone the roadmap repository

```bash
git clone git@github.com:DMPRoadmap/roadmap.git roadmap-web
```

#### 4.2 Clone the roadmap-devres repository

```bash
git clone git@github.com:DMPRoadmap/roadmap-devres.git roadmap-devres
```

#### 4.3 Copy developer-specific files into `roadmap-web`

```bash
cp roadmap-devres/docker-only/web/Dockerfile-web roadmap-web/
cp roadmap-devres/docker-only/web/.env roadmap-web/
cp roadmap-devres/docker-only/web/config/credentials.yml.enc roadmap-web/config/
cp roadmap-devres/docker-only/web/config/database.yml roadmap-web/config/
cp roadmap-devres/docker-only/web/config/master.key roadmap-web/config/
```

This prepares the web app directory with all necessary Docker and configuration files.


### Step 5: Build and Run the Webserver Container

#### 5.1 Build the Docker image

Make sure you’re inside the `roadmap-web` directory, then run:

```bash
docker build --no-cache -t roadmap-web -f Dockerfile-web .
```

#### 5.2 Run the container

Run the container on the custom network and expose port 3000:

```bash
docker run -d \
  --name roadmap-web \
  --network roadmap-net \
  -p 3000:3000 \
  roadmap-web
```

You can access it at [http://localhost:3000](http://localhost:3000) and see a migration error

now Enter the container
Use the container name or ID:

```bash
docker exec -it roadmap-web /bin/bash
```
and run the create and migration script

!!! warning The following operations need to be done once

```bash
rails db:setup
```
Output

```bash
Copying Bootstrap glyphicons to the public directory ...
Copying TinyMCE skins to the public directory ...
Running via Spring preloader in process 93
Database 'roadmap' already exists
```

```bash
root@9840432b7e33:/usr/src/app# rails db:migrate
```
Output 

```bash
Running via Spring preloader in process 114
```

Exit the container

Now reload your Rails app — the error should be gone.

Happy coding!

---

## Recommended Command for Developer Workflow

```bash
docker run -it \
  --rm \
  --network roadmap-net \
  -v "$PWD":/usr/src/app \
  -w /usr/src/app \
  roadmap-web \
  /bin/bash
```


**Breakdown:**

* `-it`: Run interactively with a TTY.
* `--rm`: Automatically remove the container after exit (optional but helpful for dev).
* `--network roadmap-net`: Connect to the same Docker network (for DB access).
* `-v "$PWD":/usr/src/app`: Mount your local Rails project into the container.
* `-w /usr/src/app`: Set the working directory to your project root.
* `roadmap-web`: The image name (make sure it’s built).
* `/bin/bash`: The shell to enter.

---

### After entering the container:

You'll be in a shell at `/usr/src/app` with your **local code mounted**, so any file changes on your host will **instantly reflect inside the container**.

You can now run things like:

```bash
bundle install
yarn install
bin/rails server -b 0.0.0.0
```

Or migrations:

```bash
bin/rails db:migrate
```


