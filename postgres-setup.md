##for using postgres in backend:

login to postgres with super user and create app_user and create the database: CREATE ROLE app_user WITH LOGIN PASSWORD 'app_user'; CREATE DATABASE app_user OWNER app_user;

here replace app_user with the application name. by doing this we are create the user with the same name as the database and the username and password for the user is also same as the database name. this is for simplicity so that the database name, the user which can access the database and password of that user is same. but in prod, make it more unque for security.

You should also create a sperate `schema` while logged in as the application user by:

```
CREATE SCHEMA my_schema;
```
and if you are using spring boot, add this in application.properties file:
```
spring.jpa.properties.hibernate.default_schema=my_schema
```

Basically all you schema for your database will be created in this schema.

This is required because sometimes, java complains that it doesnot have access to public schema. Instead of providing access to public schema create your own schema.
