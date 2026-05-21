# Enable HTTP/2 in Nginx

following adaption of the nginx site config is needed (nginx version > 1.25.1):

```nginx
listen 443 ssl;
http2 on;
```
for older nginx-versions:

```nginx
listen 443 ssl http2;
```

verify via browser debug console.

# Enable HTTP/3 (QUIC) in Nginx

## verify nginx version
Nginx supports HTTP/3 starting with version 1.25.0.

```bash
sudo nginx -v
```

## enable http/3

```nginx
# Port 443 using UDP (HTTP/3)
listen 443 quic reuseport;
listen [::]:443 quic reuseport;

# inform browser, we support HTTP/3
add_header Alt-Svc 'h3=":443"; ma=86400';

http3 on; # optional. http/3 will enabled also without this setting
```
