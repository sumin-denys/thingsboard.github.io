---
layout: docwithnav
assignees:
- ashvayka
title: Installing ThingsBoard using Docker (Linux or Mac OS)
description: Installing ThingsBoard IoT Platform using Docker (Linux or Mac OS)

---

{% include templates/live-demo-banner.md %}

* TOC
{:toc}

This guide will help you to install and start ThingsBoard Community Edition (CE) using Docker Compose on Linux or Mac OS.
If you are looking for a cluster installation instruction, please visit [cluster setup page](/docs/user-guide/install/cluster-setup/).  

If you are using the `thingsboard/tb-postgres` image, please check the following [guide](/docs/user-guide/install/migrate-to-tb-node) to migrate to the new image.

## Prerequisites

{% include templates/install/docker-install.md %}

{% include templates/install/docker-install-note.md %}

## Step 1. Choose ThingsBoard queue service

{% include templates/install/install-queue.md %}

{% capture contenttogglespecqueue %}
In Memory <small>(built-in and default)</small>%,%inmemory%,%templates/install/docker-queue-in-memory.md%br%
Kafka <small>(recommended for on-prem, production installations)</small>%,%kafka%,%templates/install/docker-queue-kafka.md%br%
AWS SQS <small>(managed service from AWS)</small>%,%aws-sqs%,%templates/install/docker-queue-aws-sqs.md%br%
Google Pub/Sub <small>(managed service from Google)</small>%,%pubsub%,%templates/install/docker-queue-pub-sub.md%br%
Azure Service Bus <small>(managed service from Azure)</small>%,%service-bus%,%templates/install/docker-queue-service-bus.md%br%
RabbitMQ <small>(for small on-prem installations)</small>%,%rabbitmq%,%templates/install/docker-queue-rabbitmq.md%br%
Confluent Cloud <small>(Event Streaming Platform based on Kafka)</small>%,%confluent-cloud%,%templates/install/docker-queue-confluent-cloud.md{% endcapture %}

{% include content-toggle.liquid content-toggle-id="ubuntuThingsboardQueue" toggle-spec=contenttogglespecqueue %} 

Where: 

- `8080:8080` - connects local port 8080 to the exposed internal HTTP port 8080.
- `1883:1883` - connects local port 1883 to the exposed internal MQTT port 1883.
- `7070:7070` - connects local port 7070 to the exposed internal Edge RPC port 7070.
- `5683-5688:5683-5688/udp` - connects local UDP ports 5683-5688 to the exposed internal COAP and LwM2M ports.
- `~/.mytb-node-logs:/var/log/thingsboard` - mounts the host's directory `~/.mytb-node-logs` to the ThingsBoard logs directory.
- `~/.mytb-node-conf:/config` - mounts the host's directory `~/.mytb-node-conf` to the ThingsBoard configuration directory.
- `~/.mytb-data/db:/var/lib/postgresql/data` - mounts the host's directory `~/.mytb-data/db` to the PostgreSQL data directory.
- `mytb-node` - friendly local name of this machine.
- `restart: always` - automatically start ThingsBoard in case of system reboot and restarts in case of failure.
- `thingsboard/tb-node` - Docker image.
- `my-postgres` - friendly local name for PostgreSQL.
- `5432` - exposed port for connecting to the PostgreSQL database.
- `healthcheck` - health check for the PostgreSQL service using the `pg_isready` command.
- `POSTGRES_DB: thingsboard` - database name for ThingsBoard.
- `POSTGRES_PASSWORD: postgres` - password for the PostgreSQL user.


## Step 2. Setup configs and folders

{% include templates/install/docker/docker-create-folders-sudo-explained.md %}

```
mkdir -p ~/.mytb-data && sudo chown -R 799:799 ~/.mytb-data
mkdir -p ~/.mytb-node-logs && sudo chown -R 799:799 ~/.mytb-node-logs
```
{: .copy-code}

The `thingsboard/tb-node` image requires the configuration files. Download the files to the corresponding directory and apply required permissions with the following commands:
```bash
mkdir -p ~/.mytb-node-conf
curl -L -o ~/.mytb-node-conf/thingsboard.conf https://raw.githubusercontent.com/thingsboard/thingsboard/refs/heads/master/docker/tb-node/conf/thingsboard.conf
curl -L -o ~/.mytb-node-conf/logback.xml https://raw.githubusercontent.com/thingsboard/thingsboard/refs/heads/master/docker/tb-node/conf/logback.xml
sudo chown -R 799:799 ~/.mytb-node-conf
```
{: .copy-code}

**NOTE**: Replace directories:
- `~/.mytb-data/db`
- `~/.mytb-node-logs`
- `~/.mytb-node-conf` 

with directories you're planning to use in `docker-compose.yml`.

## Step 3. Install ThingsBoard

Run the following command to install ThingsBoard:

```bash
docker compose run --rm \
  -e INSTALL_TB=true \
  -v ~/.mytb-node-logs:/var/log/thingsboard \
  -v ~/.mytb-node-conf:/config \
  mytb-node \
  sh -c "start-tb-node.sh"
```
{: .copy-code}

## Step 4. Run ThingsBoard

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

## Upgrading

In order to update to the latest image сhange your current version to the latest one, for example:

```text
image: "thingsboard/tb-node:3.9.0"  --> image: "thingsboard/tb-node:latest"
```

Run the following commands:

```bash
docker pull thingsboard/tb-node:latest
docker compose run --rm \
  -e UPGRADE_TB=true  \
  -e FROM_VERSION=3.9.0 \
  -v ~/.mytb-node-logs:/var/log/thingsboard \
  -v ~/.mytb-node-conf:/config \
  mytb-node \
  sh -c "start-tb-node.sh"
docker compose up
```
{: .copy-code}

## Troubleshooting

### DNS issues

{% include templates/troubleshooting/dns-issues.md %}

## Next steps

{% assign currentGuide = "InstallationGuides" %}{% include templates/guides-banner.md %}