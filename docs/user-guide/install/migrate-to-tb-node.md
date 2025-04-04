---
layout: docwithnav
assignees:
- ashvayka
title: Migrate to new ThingsBoard image (Linux or Mac OS)
description: Migrate to new ThingsBoard image (Linux or Mac OS)

---
* TOC
{:toc}

This guide will assist you in migrating from the `thingsboard/tb-postgres` to the `thingsboard/tb-node` image.

## Prerequisites

{% include templates/install/docker-install.md %}

{% include templates/install/docker-install-note.md %}

## Step 1. Create Backup of Database (Optional)

Before making any changes, it is hice to take a backup of the database to protect your data. In the event that something goes wrong during the migration, a backup allows you to restore the database to its previous state, preventing potential data loss.

Run the following command to create a backup of the database:

``` bash
docker exec -it mytb sh -c pg_dump -U thingsboard -h localhost -d thingsboard > ~/thingsboard_database_backup.sql
```
{: .copy-code}

Where:
- `mytb` - is the container name of your ThingsBoard instance. Change this if you are using a different container name.
- `~/thingsboard_database_backup.sql` - is the path where the backup file will be saved. Modify this path if necessary.

## Step 2. Apply Changes to PostgreSQL

Create a role that will be used with the new PostgreSQL container:
```bash
docker exec -i mytb psql -U thingsboard -c "CREATE ROLE postgres WITH SUPERUSER CREATEDB CREATEROLE REPLICATION BYPASSRLS LOGIN PASSWORD 'postgres';"
```
{: .copy-code}

Next, change the owner of ThingsBoard database to the `postgres` user:
```bash

```
{: .copy-code}

## Step 3. Change the Ownership of the Database Directory

The PostgreSQL container will use the `postgres` user to access the database directory. The following command applies the required permissions to the database directory:
```bash
sudo chown -R 999:999 ~/.mytb-data/db
```
{: .copy-code}

## Step 4. Allow Connection to PostgreSQL

The PostgreSQL database in the `thingsboard/tb-postgres` image is configured to listen only on a Unix socket by default. To enable TCP connections, follow these steps:
Open the PostgreSQL configuration file:
```bash
sudo nano ~/.mytb-data/db/postgresql.conf
```
{: .copy-code}
Press **Ctrl + W** to search required line and type: 
```text
#listen_addresses = 'localhost'
```
{: .copy-code}
press **Enter**. 
Change it to 
```text
listen_addresses = '*'
```
{: .copy-code}
press **Ctrl + O** to save and **Ctrl + X** to exit.


To allow connections from another container as the `postgres` user, open the `pg_hba.conf` file:
```bash
sudo nano ~/.mytb-data/db/pg_hba.conf
```
{: .copy-code}
press **Ctrl + Page Down** and add the following line:
```text
host    all             postgres        0.0.0.0/0                    md5
```
{: .copy-code}
Press **Ctrl + O** to save and **Ctrl + X** to exit.

## Step 5. Create Logs and Configuration Directories

The new `thingsboard/tb-node` image requires dedicated logs and configuration directories. The following commands will create the necessary folders and download configuration files:
```bash
mkdir -p ~/.mytb-node-conf 
mkdir -p ~/.mytb-node-logs 
curl -L -o ~/.mytb-node-conf/thingsboard.conf https://raw.githubusercontent.com/thingsboard/thingsboard/refs/tags/v3.9.1/docker/tb-node/conf/thingsboard.conf
curl -L -o ~/.mytb-node-conf/logback.xml https://raw.githubusercontent.com/thingsboard/thingsboard/refs/tags/v3.9.1/docker/tb-node/conf/logback.xml
sudo chown -R 799:799 ~/.mytb-node-conf
sudo chown -R 799:799 ~/.mytb-node-logs
```
{: .copy-code}

## Step 6. Change Docker Compose

Apply the following changes to your `docker-compose.yml` file according to this example:
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
- `~/.mytb-data/db`
- `~/.mytb-node-logs`
- `~/.mytb-node-conf` 

with directories you're planning to use in `docker-compose.yml`.

## Step 7. Running ThingsBoard

{% assign serviceName = "tb-node" %}
{% include templates/install/docker/docker-compose-up-and-ui-credentials.md %}

## Detaching, stop and start commands

{% assign serviceFullName = "ThingsBoard" %}

You can detach from session terminal using `Ctrl-p` `Ctrl-q` key sequence - the container will keep running in the background.

In case of any issues you can examine service logs for errors.
For example to see {{serviceFullName}} container logs execute the following command:

```
docker compose logs -f my{{serviceName}}
```
{: .copy-code}

To stop the container:

```
docker compose stop my{{serviceName}}
```
{: .copy-code}

To start the container:

```
docker compose start my{{serviceName}}
```
{: .copy-code}

## Troubleshooting

### DNS issues

{% include templates/troubleshooting/dns-issues.md %}

### Second migrate option

If this guide was not useful for you, please visit the alternative [guide](/docs/user-guide/install/migrate-to-tb-node-backup-restore/).