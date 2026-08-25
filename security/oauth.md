[https://auth0.com/docs/authenticate/protocols/oauth](https://auth0.com/docs/authenticate/protocols/oauth)

[https://auth0.com/docs/secure/tokens/access-tokens](https://auth0.com/docs/secure/tokens/access-tokens)

Json web tokens

[https://www.jwt.io/introduction#what-is-json-web-token](https://www.jwt.io/introduction#what-is-json-web-token)

Learn about authentication B2B

++[https://auth0.com/docs/get-started/architecture-scenarios/business-to-business/authorization](https://auth0.com/docs/get-started/architecture-scenarios/business-to-business/authorization)++

Oauth vulnerabilities 

++[https://portswigger.net/web-security/oauth](https://portswigger.net/web-security/oauth)++

Completed Lab: Authentication bypass via OAuth implicit flow

++[https://portswigger.net/web-security/oauth/lab-oauth-authentication-bypass-via-oauth-implicit-flow](https://portswigger.net/web-security/oauth/lab-oauth-authentication-bypass-via-oauth-implicit-flow)++

Completed Lab: Forced OAuth profile linking

++[https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking](https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking)++

Completed Lab: OAuth account hijacking via redirect_uri

++[https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri](https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri)++

You validate a JWT to make sure the token makes sense, adheres to the expected standards, contains the right data.

You verify a JWT to make sure the token hasn't been altered maliciously and comes from a trusted source.  

Jwts what are they and the attacks involved 

++[https://portswigger.net/web-security/jwt#what-are-jwts](https://portswigger.net/web-security/jwt#what-are-jwts)++

# **Authorization Code Flow with Proof Key for Code Exchange (PKCE)**

[https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce)

add auth0 jwt to springboot api 

oauth for java 

[https://www.youtube.com/live/z2Bt971k1EE?si=gDlLxp1k-fZxWWBY](https://www.youtube.com/live/z2Bt971k1EE?si=gDlLxp1k-fZxWWBY)

using thiss in the project!!

[https://auth0.com/docs/quickstart/backend/java-spring-security5](https://auth0.com/docs/quickstart/backend/java-spring-security5)

[https://auth0.com/blog/introducing-auth0-cli/](https://auth0.com/blog/introducing-auth0-cli/)

adding docker 

[https://www.postgresql.org/docs/current/](https://www.postgresql.org/docs/current/)

[https://docs.spring.io/spring-boot/reference/features/dev-services.html](https://docs.spring.io/spring-boot/reference/features/dev-services.html)

[https://www.geeksforgeeks.org/springboot/spring-boot-creating-docker-image-using-gradle/](https://www.geeksforgeeks.org/springboot/spring-boot-creating-docker-image-using-gradle/)

[https://docs.docker.com/build/building/best-practices/#dockerignore-file](https://docs.docker.com/build/building/best-practices/#dockerignore-file)

[https://docs.spring.io/spring-boot/reference/packaging/container-images/dockerfiles.html](https://docs.spring.io/spring-boot/reference/packaging/container-images/dockerfiles.html)

[https://docs.docker.com/compose/](https://docs.docker.com/compose/)

[https://www.youtube.com/watch?v=DQdB7wFEygo](https://www.youtube.com/watch?v=DQdB7wFEygo)

[https://docs.docker.com/reference/compose-file/](https://docs.docker.com/reference/compose-file/)

addd postrgres jpa spring boot

[https://docs.spring.io/spring-data/jpa/reference/](https://docs.spring.io/spring-data/jpa/reference/)

[https://start.spring.io](https://start.spring.io)

[https://docs.spring.io/spring-boot/reference/data/sql.html#data.sql.datasource](https://docs.spring.io/spring-boot/reference/data/sql.html#data.sql.datasource)

[https://docs.spring.io/spring-boot/appendix/application-properties/index.html#appendix.application-properties.data](https://docs.spring.io/spring-boot/appendix/application-properties/index.html#appendix.application-properties.data)

[https://docs.spring.io/spring-boot/reference/features/external-config.html](https://docs.spring.io/spring-boot/reference/features/external-config.html)

[https://docs.spring.io/spring-boot/how-to/properties-and-configuration.html](https://docs.spring.io/spring-boot/how-to/properties-and-configuration.html)

[https://docs.docker.com/compose/how-tos/networking/](https://docs.docker.com/compose/how-tos/networking/)

flyway

[https://documentation.red-gate.com/fd](https://documentation.red-gate.com/fd)

 docker exec -it  orgprojects-db sh       

# psql -U adminpr -d orgprojects

psql (16.14 (Debian 16.14-1.pgdg13+1))

Type "help" for help.

orgprojects-# \du

```
                         List of roles
```

 Role name |                         Attributes                         

-----------+------------------------------------------------------------

 adminpr   | Superuser, Create role, Create DB, Replication, Bypass RLS

orgprojects-# \l

```
                                                  List of databases

Name     |  Owner  | Encoding | Locale Provider |  Collate   |   Ctype    | ICU Locale | ICU Rules |  Access privileges  
```

-------------+---------+----------+-----------------+------------+------------+------------+-----------+---------------------

 orgprojects | adminpr | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | 

 postgres    | adminpr | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | 

 template0   | adminpr | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/adminpr         +

```
         |         |          |                 |            |            |            |           | adminpr=CTc/adminpr
```

 template1   | adminpr | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/adminpr         +

```
         |         |          |                 |            |            |            |           | adminpr=CTc/adminpr
```

(4 rows)

orgprojects-# 

[https://www.postgresql.org/docs/current/sql-createtable.html](https://www.postgresql.org/docs/current/sql-createtable.html)

[https://docs.spring.io/spring-boot/how-to/data-initialization.html](https://docs.spring.io/spring-boot/how-to/data-initialization.html)  
[https://docs.spring.io/spring-boot/reference/data/sql.html#data.sql.jpa-and-spring-data](https://docs.spring.io/spring-boot/reference/data/sql.html#data.sql.jpa-and-spring-data)

validate access token  
[https://auth0.com/docs/secure/tokens/access-tokens/validate-access-tokens](https://auth0.com/docs/secure/tokens/access-tokens/validate-access-tokens)

export TOKEN='token'

curl -s -w "\nHTTP %{http_code}\n" [http://localhost:8081/api/private](http://localhost:8081/api/private) \

-H "Authorization: Bearer $TOKEN"

curl -s -w "\nHTTP %{http_code}\n" -X POST [http://localhost:8081/api/me/sync](http://localhost:8081/api/me/sync) \

-H "Authorization: Bearer $TOKEN"