# Spring Data JPA + Flyway + Postgres Notes

## Reference Videos/Articles

- Getting started with JPA: [https://youtu.be/wuX2ESOy-Ts](https://youtu.be/wuX2ESOy-Ts)
- Flyway (model-first, IntelliJ integration): [https://youtu.be/AMopB9C2bH8](https://youtu.be/AMopB9C2bH8)
- JetBrains blog — database migrations in the real world: [https://blog.jetbrains.com/idea/2025/02/database-migrations-in-the-real-world/](https://blog.jetbrains.com/idea/2025/02/database-migrations-in-the-real-world/)
- Hibernate + records / data-oriented programming / pattern matching: [https://youtu.be/jS_bIWWW6hA](https://youtu.be/jS_bIWWW6hA)
- Persisting enums in JPA (Baeldung): [https://www.baeldung.com/jpa-persisting-enums-in-jpa](https://www.baeldung.com/jpa-persisting-enums-in-jpa)

## Maven Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```



## application.yml , Datasource Config

JDBC URL shape: `jdbc:postgresql://HOST:PORT/DATABASE_NAME`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/loginauth2
    username: aysha
    password: sha12
    driver-class-name: org.postgresql.Driver
```



## docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:16
    container_name: loginauth2-db
    environment:
      POSTGRES_DB: loginauth2
      POSTGRES_USER: aysha
      POSTGRES_PASSWORD: sha12
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```



## Docker Compose Commands


| Command                     | What it does                                                                   |
| --------------------------- | ------------------------------------------------------------------------------ |
| `docker compose up`         | Starts the full stack in the foreground, streaming logs to the terminal        |
| `docker compose up -d`      | Starts the stack detached (background), frees up the terminal                  |
| `docker compose up --build` | Rebuilds custom images from the Dockerfile before starting                     |
| `docker compose down`       | Stops and removes containers, networks, and configs from the stack             |
| `docker compose down -v`    | Same as `down`, plus deletes the named volumes , full reset, including DB data |




## Verifying via psql

```
psql -U aysha -d loginauth2
```

Useful meta-commands:

- `\du` : list roles/users and their attributes
- `\dt` : list tables in the current schema

Example output confirming the schema is in place after migrations ran:

```
loginauth2=# \dt
               List of relations
 Schema |         Name          | Type  | Owner
--------+-----------------------+-------+-------
 public | flyway_schema_history | table | aysha
 public | permissions           | table | aysha
 public | role_permissions      | table | aysha
 public | roles                 | table | aysha
 public | user_roles            | table | aysha
 public | users                 | table | aysha
(6 rows)
```

`flyway_schema_history` is Flyway's own bookkeeping table , it tracks which migrations have run,
their checksums, and when. Don't touch it manually; if a migration needs to change, write a new
versioned migration rather than editing an applied one.