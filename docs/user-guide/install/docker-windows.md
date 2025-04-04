---
layout: docwithnav
assignees:
- ashvayka
title: Installing ThingsBoard using Docker (Windows)
description: Installing ThingsBoard IoT Platform using Docker (Windows)

---

{% include templates/live-demo-banner.md %}

* TOC
{:toc}
This guide will help you to install and start ThingsBoard Community Edition (CE) using on Windows.
If you are looking for a cluster installation instruction, please visit [cluster setup page](/docs/user-guide/install/cluster-setup/).  

If you are using the `thingsboard/tb-postgres` image, please check the following [guide](/docs/user-guide/install/migrate-to-tb-node) to migrate to the new image.

## Prerequisites

- [Install Docker Toolbox for Windows](https://docker-docs.uclv.cu/toolbox/toolbox_install_windows/)


## Step 1. Choose ThingsBoard queue service

{% include templates/install/install-queue.md %}

{% capture contenttogglespecqueue %}
In Memory <small>(built-in and default)</small>%,%inmemory%,%templates/install/windows-docker-queue-in-memory.md%br%
Kafka <small>(recommended for on-prem, production installations)</small>%,%kafka%,%templates/install/windows-docker-queue-kafka.md%br%
AWS SQS <small>(managed service from AWS)</small>%,%aws-sqs%,%templates/install/windows-docker-queue-aws-sqs.md%br%
Google Pub/Sub <small>(managed service from Google)</small>%,%pubsub%,%templates/install/windows-docker-queue-pub-sub.md%br%
Azure Service Bus <small>(managed service from Azure)</small>%,%service-bus%,%templates/install/windows-docker-queue-service-bus.md%br%
RabbitMQ <small>(for small on-prem installations)</small>%,%rabbitmq%,%templates/install/windows-docker-queue-rabbitmq.md%br%
Confluent Cloud <small>(Event Streaming Platform based on Kafka)</small>%,%confluent-cloud%,%templates/install/windows-docker-queue-confluent-cloud.md{% endcapture %}

{% include content-toggle.liquid content-toggle-id="ubuntuThingsboardQueue" toggle-spec=contenttogglespecqueue %} 

Where: 

- `8080:8080` - connects local port 8080 to the exposed internal HTTP port 8080.
- `1883:1883` - connects local port 1883 to the exposed internal MQTT port 1883.
- `7070:7070` - connects local port 7070 to the exposed internal Edge RPC port 7070.
- `5683-5688:5683-5688/udp` - connects local UDP ports 5683-5688 to the exposed internal COAP and LwM2M ports.
- `mytb-node-logs:/var/log/thingsboard` - maps the `mytb-node-logs` volume to the `/var/log/thingsboard` directory inside the container.
- `mytb-node-conf:/config` - maps the `mytb-node-conf` volume to the `/config` directory inside the container.
- `mytb-data/db:/var/lib/postgresql/data` - maps the `mytb-data` volume to the `/var/lib/postgresql/data` directory inside the container.
- `mytb-node` - friendly local name of this machine.
- `restart: always` - automatically start ThingsBoard in case of system reboot and restarts in case of failure.
- `thingsboard/tb-node` - Docker image.
- `my-postgres` - friendly local name for PostgreSQL.
- `5432` - exposed port for connecting to the PostgreSQL database.
- `healthcheck` - health check for the PostgreSQL service using the `pg_isready` command.
- `POSTGRES_DB: thingsboard` - database name for ThingsBoard.
- `POSTGRES_PASSWORD: postgres` - password for the PostgreSQL user.

## Step 2. Setup config and folders

Windows users should use docker managed volume for ThingsBoard Database. 
Create docker volume (for ex. `mytb-data`) before executing docker run command:
Open "Docker Quickstart Terminal". Execute the following command to create docker volumes:

```text
docker volume create mytb-data
docker volume create mytb-node-logs
docker volume create mytb-node-conf
```
{: .copy-code}

{% assign serviceName = "tb" %}
{% include templates/install/docker/docker-compose-up.md %}

After executing this command you can open `http://{your-host-ip}:8080` in you browser (for ex. `http://localhost:8080`). You should see ThingsBoard login page.
Use the following default credentials:

- **System Administrator**: sysadmin@thingsboard.org / sysadmin
- **Tenant Administrator**: tenant@thingsboard.org / tenant
- **Customer User**: customer@thingsboard.org / customer
    
You can always change passwords for each account in account profile page.

## Detaching, stop and start commands

{% assign serviceFullName = "ThingsBoard" %}
{% include templates/install/docker/detaching-stop-start-commands.md %}

## Upgrading

In order to update to the latest image, open "Docker Quickstart Terminal" and execute the following commands:

```
$ docker pull thingsboard/tb-postgres
$ docker compose stop
$ docker run -it -v mytb-data:/data --rm thingsboard/tb-postgres upgrade-tb.sh
$ docker compose rm mytb
$ docker compose up
```

{% capture dockerComposeStandalone %}
If you still rely on Docker Compose as docker-compose (with a hyphen) here is the list of the above commands:
<br>**$ docker pull thingsboard/tb-postgres**
<br>**$ docker-compose stop**
<br>**$ docker run -it -v mytb-data:/data --rm thingsboard/tb-postgres upgrade-tb.sh**
<br>**$ docker-compose rm mytb**
<br>**$ docker-compose up**
{% endcapture %}
{% include templates/info-banner.md content=dockerComposeStandalone %}


**NOTE**: if you use different database change image name in all commands from `thingsboard/tb-postgres` to `thingsboard/tb-cassandra` or `thingsboard/tb` correspondingly.
 
**NOTE**: replace volume `mytb-data` with volume used during container creation. 

## Troubleshooting

### DNS issues

{% include templates/troubleshooting/dns-issues-windows.md %}

## Next steps

{% assign currentGuide = "InstallationGuides" %}{% include templates/guides-banner.md %}
