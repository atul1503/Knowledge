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

## Basic Nginx Configuration Structure

### Events Block
The `events` block contains configuration for handling connections:
- `worker_connections 1024`: This sets the maximum number of connections that each worker process can handle at the same time. If you have 2 worker processes and worker_connections is 1024, nginx can handle 2048 connections total.

### HTTP Block
The `http` block contains all the web server configuration. All server blocks must be inside the http block.

### Server Block
A `server` block represents a virtual server (website) that nginx will serve. You can have multiple server blocks for different websites.

## Server Configuration Explained

1. **`listen`**: Tells nginx which port to listen on for incoming requests for this server. Example: `listen 8083;` means nginx will listen on port 8083.

2. **`server_name`**: Tells nginx the hostname of the request that it should associate with this server block. For any other hostname, create another server block inside http with its separate `server_name`. You can have multiple server_name entries in one server block.

3. **`client_max_body_size`**: Sets the maximum size of client request body (like file uploads). `1G` means 1 gigabyte maximum. Without this, nginx has a default limit of 1MB.

4. **`location`**: Represents URL paths and how nginx should handle requests to those paths.

## Location Blocks and Static File Serving

### Basic Location Syntax
```
location /path/ {
    # configuration for this path
}
```

### Root Directory
`root` is used to set the root directory in location block (or for all locations if used in server block itself). 

**Example**: In `/static/` location, if user requests `http://localhost:8083/static/my.js`, nginx will send the file `/volume/build/my.js` because:
- The location matches `/static/`
- The root is `/volume/build`
- Nginx removes `/static/` from the URL path and looks for the remaining path in the root directory

### Location Matching Types
```
# Exact match (highest priority)
location = /exact-path {
    # Only matches exactly /exact-path
}

# Prefix match (most common)
location /prefix/ {
    # Matches /prefix/anything
}

# Regex match (case sensitive)
location ~ \.(jpg|jpeg|png|gif)$ {
    # Matches any path ending with image extensions
}

# Regex match (case insensitive)
location ~* \.(jpg|jpeg|png|gif)$ {
    # Same as above but case insensitive
}
```

## Proxy Pass (Reverse Proxy)

5. **`proxy_pass`**: Used to tell nginx where to forward the request to. Here, it will forward the request to `http://videocaption:8000`. This is very important for nginx as a proxy server.

**Example**: If user visits `http://localhost:8083/api/users`, nginx will forward this request to `http://videocaption:8000/api/users`.

### Common Proxy Settings
```
location / {
    proxy_pass http://backend-server:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

**Explanation of proxy headers**:
- `Host $host`: Passes the original hostname to the backend
- `X-Real-IP $remote_addr`: Passes the client's real IP address
- `X-Forwarded-For $proxy_add_x_forwarded_for`: Passes the chain of proxy IP addresses
- `X-Forwarded-Proto $scheme`: Passes whether the request was HTTP or HTTPS

## Load Balancing with Upstream

7. **`upstream`** block allows you to provide multiple server statements and nginx will load balance workload between different servers in that upstream block:

```
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

### Load Balancing Methods
```
upstream backend {
    # Round-robin (default) - requests distributed evenly
    server backend1.example.com;
    server backend2.example.com;
    
    # Weighted round-robin
    server backend3.example.com weight=3;
    
    # Backup server (used only when others fail)
    server backend4.example.com backup;
}
```

## SSL/HTTPS Configuration

```
server {
    listen 443 ssl;
    server_name example.com;
    
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    location / {
        proxy_pass http://backend;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
```

**SSL directives explained**:
- `ssl_certificate`: Path to your SSL certificate file
- `ssl_certificate_key`: Path to your private key file
- `return 301`: Redirects HTTP requests to HTTPS permanently

## Common Nginx Directives

### Index Files
```
location / {
    root /var/www/html;
    index index.html index.htm;
}
```
`index` tells nginx which file to serve when a directory is requested.

### Try Files
```
location / {
    try_files $uri $uri/ /index.html;
}
```
`try_files` tries to serve files in order. If `$uri` (the requested path) doesn't exist, try `$uri/` (as a directory), then fallback to `/index.html`.

### Error Pages
```
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;

location = /404.html {
    root /var/www/errors;
}
```

## Performance Optimizations

### Gzip Compression
```
http {
    gzip on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
}
```

### Rate Limiting
```
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
    
    server {
        location /api/ {
            limit_req zone=mylimit burst=20 nodelay;
            proxy_pass http://backend;
        }
    }
}
```
This limits each IP address to 10 requests per second with a burst of 20 requests.

## Logging

### Access and Error Logs
```
http {
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    server {
        access_log /var/log/nginx/mysite.access.log;
        error_log /var/log/nginx/mysite.error.log;
    }
}
```

### Custom Log Format
```
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
}
```

## Security Considerations

### Hide Nginx Version
```
http {
    server_tokens off;
}
```

### Prevent Access to Hidden Files
```
location ~ /\. {
    deny all;
}
```

### Add Security Headers
```
server {
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

## Complete Example Configuration

```
events {
    worker_connections 1024;
}

http {
    # Basic settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    
    # MIME types
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;
    
    # Gzip compression
    gzip on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    
    # Upstream for load balancing
    upstream backend {
        server backend1:8000;
        server backend2:8000;
        server backend3:8000 backup;
    }
    
    # HTTP server (redirects to HTTPS)
    server {
        listen 80;
        server_name example.com www.example.com;
        return 301 https://$server_name$request_uri;
    }
    
    # HTTPS server
    server {
        listen 443 ssl;
        server_name example.com www.example.com;
        
        # SSL configuration
        ssl_certificate /path/to/certificate.crt;
        ssl_certificate_key /path/to/private.key;
        
        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        
        # File upload limit
        client_max_body_size 100M;
        
        # Static files
        location /static/ {
            root /var/www;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
        
        # Media files
        location /media/ {
            root /var/www;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
        
        # API with rate limiting
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        
        # Main application
        location / {
            try_files $uri $uri/ @fallback;
        }
        
        # Fallback to backend
        location @fallback {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        
        # Deny access to hidden files
        location ~ /\. {
            deny all;
        }
    }
}
```

**Note: Hostname is the string that comes after `http://` and ends before either port or the path. For example, hostname in `http://videocaption:8000` is videocaption.**