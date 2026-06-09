# Load Balancer

Simple guide to setup a load balancer based on nginx.

## Backend Server
First we need to create two backend-server:

```bash
cd /etc/nginx/sites-available
sudo vi backend_server
```

```nginx
server {
        listen 127.0.0.1:8080;
        server_name _;

        root /var/www/backend/srv1;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}

server {
        listen 127.0.0.1:8081;
        server_name _;

        root /var/www/backend/srv2;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

enable the new servers and validate configuration:

```bash
sudo ln -s /etc/nginx/sites-available/backend_server backend_server
sudo nginx -t
```

restart nginx

```bash
sudo systemctl reload nginx
```

to verify the two servers are running:

```bash
sudo netstat -anp | grep -E "8080|8081"
```

## simple html
the new webserver need an `index.html`

```bash
cd /var/www/
sudo mkdir -p backend/srv1
sudo mkdir -p backend/srv2
```

### srv1

`index.html` of server one:

```bash
sudo vi backend/srv1/index.html
```

```html
<html>
    <head>
        <title>Welcome!</title>
    </head>
    <body>
        <h1>Hello World! I am backend-server 1!</h1>
    </body>
</html>
```

### srv2

`index.html` of server two:

```bash
sudo vi backend/srv2/index.html
```

```html
<html>
    <head>
        <title>Welcome!</title>
    </head>
    <body>
        <h1>Hello World! I am backend-server 2!</h1>
    </body>
</html>
```

## Load Balancer

lets create a new server as load balancer.

```bash
cd /etc/nginx/sites-available
sudo vi loadbalancer
```

```nginx
upstream my_backend_server {
        # default round-robin; other options: least_conn; ip_hash;
        # or using custom hash based on e.g. cookies: hash $cookie_<cookie-name> consistent;
        server 127.0.0.1:8080;
        server 127.0.0.1:8081;
}

server {
    listen 80;

    server_name *mydomain*;

    location / {
        proxy_pass http://my_backend_server;
    }
}


```

enable the new configuration & verify:

```bash
sudo ln -f -s /etc/nginx/sites-available/loadbalancer *mydomain*
sudo nginx -t
```

restart nginx:

```bash
sudo systemctl reload nginx
```

## Test

load the website and refresh it serveral times...

### simulate server crash

simply change the listen port of one the backend-server (e.g. srv2 listen on 8082). Restart nginx. Retry the test.

### simulate server failure

before `location` add directive: `return 500;`. Restart nginx. Retry the test.

## Handle server failure

edit load balancer configuration:

```nginx
[...]
    location / {
        proxy_pass http://my_backend_server;
        proxy_next_upstream timeout http_500;
    }
[...]
```

validate configuration, restart nginx and repeat the server failure test.