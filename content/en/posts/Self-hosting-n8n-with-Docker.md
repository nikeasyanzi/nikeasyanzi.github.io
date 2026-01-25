+++
date = '2025-09-04T16:10:56+08:00'
draft = 'False'
title = 'Self hosting local n8n service'
categories = ["self-hosting"]
tags = ["self-hosting", "n8n"]
ShowToc = true
TocOpen = true
comments = true
+++

# Steps

## 1. Create Docker Compose File

First, create a project directory to store n8n-related files. Run the following commands in the terminal:

```
mkdir n8n-docker
cd n8n-docker
```

In the project directory, create a `docker-compose.yml` file to define the Docker containers for n8n and PostgreSQL.

docker-compose.yml

```yaml
services:
  n8n:
    image: n8nio/n8n:nightly
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - ./n8n_storage:/home/node/.n8n
    depends_on:
      - postgres

  postgres:
    image: postgres:13.22-alpine3.22
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - ./postgres_storage:/var/lib/postgresql/data

volumes:
  n8n_storage:
  postgres_storage:
```

Explanation:

- Uses the n8n Docker image and maps port `5678` from the container to port `5678` on the host.
- `postgres` service: Uses the PostgreSQL Docker image and sets the username, password, and database name (stored in the `.env` file).
- `volumes`: Used to persist data for n8n and PostgreSQL respectively.
- GENERIC_TIMEZONE and TZ environment variables: Set the timezone to Taipei timezone.
- For more configuration details, refer to the [n8n official documentation](https://docs.n8n.io/hosting/installation/docker).

## 2. Create `.env` File

In the project directory, create a `.env` file to set the PostgreSQL username, password, and database name.
According to the [Docker official documentation](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/), the **.env** file will be treated as an environment variable file.

```bash
POSTGRES_USER=your_username
POSTGRES_PASSWORD=your_password
POSTGRES_DB=your_n8n_database
```

## 3. Start n8n Service

Run the following command in the terminal to start the n8n and PostgreSQL services. You should see the created containers running in the background.

```
docker compose up -d
```

![](https://i.imgur.com/pNVWBDo.png)

We can check the PostgreSQL logs:

```
docker logs n8n-postgres-1
```

![](https://i.imgur.com/WxdS2hT.png)

## 4. Open Browser and Access n8n Page

<http://localhost:5678>
![](https://i.imgur.com/J8ycuRo.png)

![](https://i.imgur.com/aYferHm.png)

# Reference

<https://darrenjon.com/docs/n8n/n8n-docker-setup/>
