**ThingsBoard includes In Memory Queue service and use it by default without extra settings.**

Create docker compose file for ThingsBoard queue service:

```text
nano docker-compose.yml
```
{: .copy-code}

Add the following lines to the yml file:

```yml
services:
  mytb-node:
    restart: always
    image: "thingsboard/tb-node:{{ site.release.ce_ver }}"
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
      - ~/.mytb-data/db:/var/lib/postgresql/data
```
{: .copy-code}
