---
layout: docwithnav
assignees:
- ashvayka
title: Migrate to new ThingsBoard image (Linux or Mac OS)
description: Migrate to new ThingsBoard image (Linux or Mac OS)

---
* TOC
{:toc}
This guide will help you migrate from `thingsboard/tb-postgres` to `thingsboard/tb-node` image.

## Prerequisites

{% include templates/install/docker-install.md %}

{% include templates/install/docker-install-note.md %}

## Step 1. Create backup of the database

To migrate to the new image, you need to create a backup to move the database to a separate Postgres container.

``` bash
docker exec -it mytb sh -c pg_dump -U thingsboard -h localhost -d thingsboard > ~/thingsboard_database_backup.sql
```
{: .copy-code}

## Step 2. Create a temporary database container

A temporary Postgres container is required to restore the ThingsBoard database, which will be used with the `thingsboard/tb-node` image.

```bash
docker run -d \
  --restart always \
  --name postgres \
  -p 5432:5432 \
  -e POSTGRES_DB=thingsboard \
  -e POSTGRES_PASSWORD=postgres \
  -v ~/.mytbce-data/:/var/lib/postgresql/data \
  postgres:15
```
{: .copy-code}

## Step 3. Create a temporary ThingsBoard role

The new Postgres container does not have a ThingsBoard role required to restore the database. Use the following command to create it manually.

```bash
docker exec -it postgres psql -U postgres -c "CREATE ROLE thingsboard WITH LOGIN SUPERUSER;"
```
{: .copy-code}

## Step 4. Restore the ThingsBoard database

```bash
docker exec -i postgres psql -U postgres -d thingsboard < ~/thingsboard_database_backup.sql
```
{: .copy-code}

## Step 5. Change the database owner from ThingsBoard to Postgres

This is necessary for correctly upgrading to a newer platform version.

```bash
docker exec -it postgres psql -U postgres -d thingsboard -c "REASSIGN OWNED BY thingsboard TO postgres;"
```
{: .copy-code}

## Step 6. Delete the Thingsboard role

The thingsboard role is no longer required for database functionality, as the owner of the ThingsBoard database is now Postgres.

```bash
docker exec -it postgres psql -U postgres -c "DROP ROLE thingsboard;"
```
{: .copy-code}

## Step 7. Create the First Launch File

This file indicates that ThingsBoard has been installed.

```bash
sudo touch ~/.mytbce-data/.firstlaunch && sudo chown 799:799 ~/.mytbce-data/.firstlaunch 
```
{: .copy-code}

## Step 8. Create the docker-compose.yml

```yml
services:
  mytb-node:
    restart: always
    image: "thingsboard/tb-node:3.9.1" 
    depends_on:
      my-postgres:
        condition: service_healthy
    ports:
      - "8080:8080"
      - "1883:1883"
      - "7070:7070"
      - "5683-5688:5683-5688/udp"
    environment:
      TB_QUEUE_TYPE: in-memory
      SPRING_DATASOURCE_URL: jdbc:postgresql://my-postgres:5432/thingsboard
      DATABASE_TS_TYPE: sql
      SPRING_DRIVER_CLASS_NAME: org.postgresql.Driver
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
    volumes:
      - ~/.mytb-node-logs:/var/log/thingsboard
      - ~/.mytb-node-conf:/config
  my-postgres:
    restart: always
    image: "postgres:12"
    ports:
    - "5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    environment:
      POSTGRES_DB: thingsboard
      POSTGRES_PASSWORD: postgres
    volumes:
      - ~/.mytb-data/db:/var/lib/postgresql/data
```
{: .copy-code}

Where:

- `8080:8080` - connects local port 8080 to the exposed internal HTTP port 8080.
- `1883:1883` - connects local port 1883 to the exposed internal MQTT port 1883.
- `7070:7070` - connects local port 7070 to the exposed internal Edge RPC port 7070.
- `5683-5688:5683-5688/udp` - connects local UDP ports 5683-5688 to the exposed internal COAP and LwM2M ports.
- `~/.mytb-node-logs:/var/log/thingsboard` - mounts the host's directory `~/.mytb-node-logs` to the ThingsBoard logs directory.
- `~/.mytb-node-conf:/config` - mounts the host's directory `~/.mytb-node-conf` to the ThingsBoard configuration directory.
- `mytb-node` - friendly local name of this service.
- `restart: always` - automatically starts ThingsBoard in case of system reboot and restarts in case of failure.
- `image: thingsboard/tb-node` - Docker image for ThingsBoard.
- `5432` - exposed port for connecting to the PostgreSQL database.
- `my-postgres` - friendly local name for PostgreSQL.
- `restart: always` - utomatically starts PostgreSQL in case of system reboot and restarts in case of failure.
- `image: postgres:12` - Docker image for PostgreSQL.
- `healthcheck` - health check for the PostgreSQL service using the `pg_isready` command.
- `POSTGRES_DB: thingsboard` - database name for ThingsBoard.
- `POSTGRES_PASSWORD: postgres` - password for the PostgreSQL user.
- `~/.mytb-data/db:/var/lib/postgresql/data` - mounts the host's directory `~/.mytb-data/db` to the PostgreSQL data directory.

**NOTE**: Replace directories:
- `~/.mytb-data`
- `~/.mytb-node-logs`
- `~/.mytb-node-conf` 

with directories you're planning to use in `docker-compose.yml`.

## Step 9. Running ThingsBoard

```bash
docker compose up
```
{: .copy-code}