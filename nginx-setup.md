```
events {
    worker_connections 1024;  # Adjust the number of worker connections based on your needs
}

http {
server {
  listen 8083;
  server_name localhost;
  server_name 127.0.0.1;
  client_max_body_size 1G;

  location /static/ {
      root /volume/build;
  }
  location /media/ {
    root /volume/media;
  }
    location / {
    proxy_pass http://videocaption:8000;
  }
}

}
```

this nginx config allows to serve static files using /static/ path. so when a browser requests for http://localhost:8083/static/ it will get files from this folder: `/volume/build`. `server` block is there to represent a server.

1. `listen` tells nginx to listen on this port for requests for this server.
2. `server_name` tells nginx the host name of the request that it should associate with this server block. For any other hostname, create another server block inside http with its seperate `server_name`.
3. `location` represents url paths. Here, for / , you are telling nginx to follow this configuration `{ proxy_pass http://videocaption:8000; }`.
4. `proxy_pass` is used to tell nginx where to forward the request to. Here, it will formward the request to `http://videocaption:8000`.  `proxy_pass` is very important for nginx as proxy server.
5. root is used to set the root directory in location block (or for all locations if used in server block itself). In /static location, if user requests for `http://localhost:8083/static/my.js`, nginx will send the following file `/volume/build/my.js`
6. using the `location` block and `root` attribute you can easily, we can use nginx to server static files easily.
7. `upstream` block allows you to provide multiple server statements. and then you can use the upstream name in hostname to proxy pass and nginx will load balance workload between different servers on that upstream block.



**Note: host name is the string that comes after `http://` and ends on either port or the path. For example, hostname in `http://videocaption:8000` is videocaption.**
