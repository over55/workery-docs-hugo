---
title: 'Setup via Docker Compose'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 2
---

## Step 1: Workspace

Create a `workery` folder on your machine.

```shell
mkdir ~/workery
```

Afterwards enter it.

```shell
cd ~/workery
```

## Step 2: Environment Variables

Create an empty environment variable file:

```shell
touch .env
```

Copy and paste the following sample environment variable into the `.env` file.

```text
# BACKEND CONFIGURATION
WORKERY_DB_NAME=workery_v2_db
WORKERY_DB_USER=golang
WORKERY_DB_PASSWORD=123password
WORKERY_AWS_S3_ACCESS_KEY=xxx
WORKERY_AWS_S3_SECRET_KEY=xxx
WORKERY_AWS_S3_ENDPOINT=https://xxx.digitaloceanspaces.com
WORKERY_AWS_S3_REGION=us-east-1
WORKERY_AWS_S3_BUCKET_NAME=xxx
WORKERY_APP_SIGNING_KEY=xxx
WORKERY_APP_HAS_AUTO_MIGRATIONS=true

# FRONTEND CONFIGURATION
# None
```

Afterwards please change the environment variables for your build.

## Step 3: Docker Compose

Create an empty docker compose file:

```shell
touch docker-compose.yml
```

Copy and paste the following into the `docker-compose.yml` file:

```yml
version: '3.8'
services:
  frontend:
    container_name: workery-frontend
    image: "over55/workery-frontend:selfhost-latest"
    stdin_open: true
    restart: unless-stopped
    links:
      - backend
      - postgresdb
      - bleve-server
      - redis
    depends_on:
      - backend
      - postgresdb
      - bleve-server
      - redis
    ports:
        - '10001:80'

  backend:
    container_name: workery_backend_app
    image: over55/workery-backend:latest
    restart: unless-stopped
    stdin_open: true
    environment:
        WORKERY_DB_HOST: postgresdb
        WORKERY_DB_PORT: 5432
        WORKERY_DB_USER: ${WORKERY_DB_USER}
        WORKERY_DB_PASSWORD: ${WORKERY_DB_PASSWORD}
        WORKERY_DB_NAME: ${WORKERY_DB_NAME}
        WORKERY_APP_IP: 0.0.0.0
        WORKERY_APP_PORT: 8000
        WORKERY_APP_REDIS_ADDRESS: redis:6379
        WORKERY_APP_SIGNING_KEY: ${WORKERY_APP_SIGNING_KEY}
        WORKERY_APP_HAS_AUTO_MIGRATIONS: ${WORKERY_APP_HAS_AUTO_MIGRATIONS}
        WORKERY_AWS_S3_ACCESS_KEY: ${WORKERY_AWS_S3_ACCESS_KEY}
        WORKERY_AWS_S3_SECRET_KEY: ${WORKERY_AWS_S3_SECRET_KEY}
        WORKERY_AWS_S3_ENDPOINT: ${WORKERY_AWS_S3_ENDPOINT}
        WORKERY_AWS_S3_REGION: ${WORKERY_AWS_S3_REGION}
        WORKERY_AWS_S3_BUCKET_NAME: ${WORKERY_AWS_S3_BUCKET_NAME}
        WORKERY_BLEVE_SERVER_ADDRESS: bleve-server:8001
    links:
      - postgresdb
      - bleve-server
      - redis
    depends_on:
      - postgresdb
      - bleve-server
      - redis
    ports:
      - "10000:8000"

  bleve-server:
      container_name: workery-bleve-server
      image: bmika/bleve-server:1.0 # Note: https://hub.docker.com/r/bmika/bleve-server
      environment:
        BLEVE_SERVER_ADDRESS: 0.0.0.0:8001
        BLEVE_SERVER_HOME_DIRECTORY_PATH: /db
      volumes:
        - bleve_data:/db

  postgresdb:
      container_name: workery-postgres
      image: postgres:14.3-alpine
      restart: unless-stopped
      environment:
          POSTGRES_PASSWORD: ${WORKERY_DB_PASSWORD}
          POSTGRES_USER: ${WORKERY_DB_USER}
          POSTGRES_DB: ${WORKERY_DB_NAME}
      volumes:
          # DEVELOPERS NOTE: We want persistance of our postgres server so here
          # is where we will save our data.
          - db_data:/var/lib/postgresql/data

  redis:
      container_name: workery-redis
      image: redis:7.0.0-alpine
      restart: unless-stopped
      volumes:
          - redis_data:/data
          - redis_config:/usr/local/etc/redis/redis.conf

  pgadmin:
      # HOWTO: Setup PgAdmin Docker via  https://hevodata.com/learn/pgadmin-docker/
      # How to access, in your browser run: http://127.0.0.1:8080
      container_name: workery-pgadmin4
      image: dpage/pgadmin4:6.11 # Note: https://hub.docker.com/r/dpage/pgadmin4
      restart: always
      logging:
        driver: "none" # We don't want to see any logs from this container.
      environment:
        PGADMIN_DEFAULT_EMAIL: admin@admin.com  # For developer purposes, keep it simple b/c it's local on the developers machine.
        PGADMIN_DEFAULT_PASSWORD: secret
        PGADMIN_LISTEN_PORT: 80
        GUNICORN_ACCESS_LOGFILE: /dev/null # We don't want to see any logs from this container.
      ports:
        - "8080:80"
      volumes:
        - pgadmin_data:/var/lib/pgadmin
      depends_on:
        - postgresdb

volumes:
    bleve_data:
    db_data:
    redis_data:
    redis_config:
    pgadmin_data:
```

## Step 4: Start the Server

Start the server.

  ```shell
  docker compose up -d
  ```

If everything correctly you should see a message saying: `workery_backend | 2022/09/03 04:17:35 Worker and server started at 0.0.0.0:8000`.

## Step 5: Initial Accounts
We will need to log into the running container for the *backend* so we can finish setting up our accounts.

```shell
docker compose run backend /app/workery-server version shell
```

Create your *organization* in the system:

```shell
docker compose run --rm backend /app/workery-server create_tenant \
    --name="Over 55 (London) Inc." \
    --schema_name="workery" \
    --email="info@o55.ca" \
    --telephone="123-456-7898" \
    --timezone="utc" \
    --address_country="Canada" \
    --address_region="Ontario" \
    --address_locality="London" \
    --area_served="Middlesex" \
    --available_language="en" \
    --post_office_box_number="" \
    --postal_code="N6H1B4" \
    --street_address="78 Riverside Dr" \
    --street_address_extra="" \
    --state=1
```

Create our *root administrator* (a.k.a. Executive) in the system. Please change the password `secret` to something else please!

```shell
docker compose run --rm backend /app/workery-server create_user \
  --tid=1 \
  --fname="Root" \
  --lname="Administrator" \
  --email="admin@admin.com" \
  --password="secret" \
  --role_id=1 \
  --state=1
```

In your favourite browser, go to [http://localhost:10001](http://localhost:10001). If you see a login page, congratulations you've setup `workery`. Please login with the *root administrator* user you created earlier.
