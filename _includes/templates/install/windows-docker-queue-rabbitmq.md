#### RabbitMQ Installation

For installing RabbitMQ use this [instruction](https://www.rabbitmq.com/install-windows.html).

Create docker compose file for ThingsBoard queue service:

```text
docker-compose.yml
```
{: .copy-code}

Add the following line to the yml file. Don't forget to replace "YOUR_USERNAME" and "YOUR_PASSWORD" with your **real user credentials**, "localhost" and "5672" with your **real RabbitMQ host and port**:

```yml
services:
  mytb:
    restart: always
    image: "thingsboard/tb-node:{{ site.release.ce_ver }}"
    depends_on:
      my-postgres:
        condition: service_healthy
    ports:
      - "8080:9090"
      - "1883:1883"
      - "7070:7070"
      - "5683-5688:5683-5688/udp"
    environment:
      TB_QUEUE_TYPE: rabbitmq
      TB_QUEUE_RABBIT_MQ_USERNAME: YOUR_USERNAME
      TB_QUEUE_RABBIT_MQ_PASSWORD: YOUR_PASSWORD
      TB_QUEUE_RABBIT_MQ_HOST: localhost
      TB_QUEUE_RABBIT_MQ_PORT: 5672
      SPRING_DATASOURCE_URL: jdbc:postgresql://my-postgres:5432/thingsboard
      DATABASE_TS_TYPE: sql
      SPRING_DRIVER_CLASS_NAME: org.postgresql.Driver
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
    volumes:
      - mytb-node-logs:/var/log/thingsboard
      - mytb-node-conf:/config
  my-postgres:
    restart: always
    image: "postgres:16"
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
      - mytb-data/db:/var/lib/postgresql/data
volumes:
  mytb-data:
    external: true
  mytb-node-logs:
    external: true
  mytb-node-conf:
    external: true
```
{: .copy-code}
