
spring jpa 
https://youtu.be/wuX2ESOy-Ts?si=bxO9_ku8-OWUsIlN
getting started with jpa 
https://www.youtube.com/watch?v=wuX2ESOy-Ts


learn flyway
https://youtu.be/AMopB9C2bH8?si=ZEhN-YhwJiHR-4F6
flyway in intellij
model first
https://youtu.be/AMopB9C2bH8?si=EdWU7MfNV7L3xNX9
https://blog.jetbrains.com/idea/2025/02/database-migrations-in-the-real-world/

learn hibernate and jpa 
hibernate ->
 
about records data oriented programming
pattern matrching 
https://youtu.be/jS_bIWWW6hA?si=PIdnXzuzhZDq4eOx


add 

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-flyway</artifactId>
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
      <artifactId>spring-boot-starter-data-jpa-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-flyway-test</artifactId>
      <scope>test</scope>
    </dependency>

    jdbc:postgresql://HOST:PORT/DATABASE_NAME
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/loginauth2
    username: aysha          # whatever you set when creating the DB/user
    password: sha12          # whatever you set
    driver-class-name: org.postgresql.Driver


     docker-compose.yml
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
docker compose up: 
Starts your entire multi-container stack in the foreground, streaming logs directly to your terminal.

docker compose up -d:
 Runs your containers in detached mode (in the background), freeing up your terminal window.
 
 docker compose up --build: 
 Forces Docker to rebuild custom container images from your Dockerfile before launching the stack.docker compose down: Stops and completely removes running containers, internal networks, and configurations created by the stack.

 docker compose down -v: 
 Tears down the stack and deletes all associated data volumes for a clean reset.


 db
 https://www.baeldung.com/jpa-persisting-enums-in-jpa


 # psql -U aysha -d loginauth2
psql (16.14 (Debian 16.14-1.pgdg13+1))
Type "help" for help.

loginauth2=# 
loginauth2=# \du
                             List of roles
 Role name |                         Attributes                         
-----------+------------------------------------------------------------
 aysha     | Superuser, Create role, Create DB, Replication, Bypass RLS

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

loginauth2=# 
