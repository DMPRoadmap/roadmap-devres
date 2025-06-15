# Roadmap: local install with Docker

## 1. Clone the repositories

### Clone the Docker configuration repo

```cmd
git clone git@github.com:DMPRoadmap/roadmap-devres.git
cd roadmap-devres/docker
```

### Clone the Roadmap application repo

```cmd
git clone git@github.com:DMPRoadmap/roadmap.git
```
## 2. Configure environment variables

Create a `.env` file in the `roadmap-devres/docker` directory based on the provided example:

- In the 'Credentials' section, set your desired value for each key.
- Mac users will need to set the value of the `ALPINE_SUFFIX` key to `-alpine`.

## 3. Build and set up Docker environment

1. Build the container:
    ```bash
    docker compose build --no-cache
    ```
2. Run the container and enter it:
    ```bash
    docker compose run server /bin/bash
    ```  
3. Install Ruby and Javascript dependencies:
    ```bash
    bundle install; yarn install;
    ```
4. Create the database, load schema and seed data, then run any pending migrations:
    ```bash
    rails db:setup; rails db:migrate;
    ```
5. Clear and then precompile the assets:
    ```bash
    rails assets:clobber; rails assets:precompile
    ```   
6. Exit the container:
7. Serve up the application services:
    ```bash
    docker compose up
    ```
8. Open browser and go to: 
    ```http://localhost:3000```
