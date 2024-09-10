##for using postgres in backend:

login to postgres with super user and create app_user and create the database: CREATE ROLE app_user WITH LOGIN PASSWORD 'app_user'; CREATE DATABASE app_user OWNER app_user;

here replace app_user with the application name. by doing this we are create the user with the same name as the database and the username and password for the user is also same as the database name. this is for simplicity so that the database name, the user which can access the database and password of that user is same. but in prod, make it more unque for security.
