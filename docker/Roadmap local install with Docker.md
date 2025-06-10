# Roadmap local install with Docker

## Steps to clone repos
Clone Docker

```cmd
> git clone git@github.com:DMPRoadmap/roadmap-devres.git
> cd roadmap-devres/docker
```

Clone Roadmap

```cmd
> git clone git@github.com:DMPRoadmap/roadmap.git
```

Create a .env file based on .env_example:

- Change '<PLACE_HOLDER>' in line 2 with a password for 'POSTGRES_PASSWORD=<PLACE_HOLDER>'
- Change database name if necessary
- Mac user will need to comment line 15 and uncomment line 16 #ALPINE_SUFFIX=-alpine

## Steps to build Docker instance

```cmd
1. Build the container
    > docker compose --env-file .env build --no-cache
2. Run command line in the container
    > docker compose --env-file .env run server /bin/bash  
3. Bundle install
    > bundle install
4. Yarn install
    > yarn install
5.  Credentials
    > rails credentials:edit
    Add these 
        # Recaptcha credentials
        recaptcha:
        site_key: 11111
        secret_key: 22222

        dragonfly_secret: "my_dragonfly_secret"

        devise_pepper: "111122223333444455555"
6. Load DB
    > rails db:setup
    > rails db:migrate
7. Clear the assets
    > rails assets:clobber
8. Precompile assets
    > DISABLE_DATABASE=true DISABLE_SPRING=true bundle exec rails assets:precompile
9. Yarn build
    > DISABLE_DATABASE=true DISABLE_SPRING=true yarn build
10. Yarn build css
    > DISABLE_DATABASE=true DISABLE_SPRING=true yarn build:css
11. Start the container
    > docker compose --env-file .env up
12. Open browser 
    localhost:2001
```