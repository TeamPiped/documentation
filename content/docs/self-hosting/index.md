---
title: "Self-Hosting"
weight: 4
summary: How can I Self-Host Piped?
---

## Docker-Compose Caddy AIO script

First, install `git`, `docker` and `docker-compose`.

Run `git clone https://github.com/TeamPiped/Piped-Docker`.

Then, run `cd Piped-Docker`.

Then, run `./configure-instance.sh` and fill in the hostnames when asked. Choose `caddy` as the reverse proxy when asked.

Now, create A records to your server's public IP with the hostnames you had filled in above.

Finally, run `docker-compose up -d` and you're done!

Consider joining the federation protocol at https://github.com/TeamPiped/piped-federation#how-to-join

## Docker-Compose Nginx AIO script

Note: This setup requires you to have your own reverse proxy in addition to the one provide, and requires you to configure TLS manually.

First, install `git`, `docker` and `docker-compose`.

Run `git clone https://github.com/TeamPiped/Piped-Docker`.

Then, run `cd Piped-Docker`.

Then, run `./configure-instance.sh` and fill in the hostnames when asked.  Choose `nginx` as the reverse proxy when asked.

Now, create A records to your server's public IP with the hostnames you had filled in above.

Run `docker-compose up -d`.

Forward traffic to 127.0.0.1:8080 with your reverse proxy, **along with the `Host` header**.

For example, in nginx, you would do the following:
```
server {
    listen 80;
    server_name hostname; # For all 3 hostnames

    location / {
       proxy_pass http://127.0.0.1:8080;
       proxy_set_header Host $host;
    }
}
```

Finally, configure your TLS certificates if you need to!

Consider joining the federation protocol at https://github.com/TeamPiped/piped-federation#how-to-join

# Manually updating

Run `docker run --rm -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --run-once piped-frontend piped-backend piped-proxy varnish nginx caddy postgres watchtower`

## Docker-Compose with Nginx

First download the files required to run Piped.

```
mkdir piped && cd piped
wget https://raw.githubusercontent.com/TeamPiped/Piped-Backend/master/config.properties
wget https://raw.githubusercontent.com/TeamPiped/Piped-Backend/master/docker-compose.yml
```

Create two A records - one for the proxy and one for the api.
Note: Each running instance of the proxy should have it's own record to maximize performance.

For example:
A pipedapi.kavin.rocks
A pipedproxy-bom.kavin.rocks

Now, edit your `config.properties` file to reflect the changes.

Now, run piped with the following command:

```
docker-compose up -d
```

Now, find your nginx user's and group's id.

You can do this by running the following command:

```
cat /etc/passwd
```

Then look for a line which starts with `www-data` or `nginx`, here is an example of that:

```
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Now, you have the user and group id - `33:33`.

Now, run the proxy with the following command, while replacing the user parameter with what you just found:

```
docker run -d --network=host -v "/var/run/ytproxy/:/app/socket" --user 33:33 --restart unless-stopped 1337kavin/ytproxy:latest
```

You can now use watchtower to enable automatic container updates (optional):

```
docker run -d \
    --name watchtower \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower
```

Now, create an nginx snipper like so:

`/etc/nginx/snippets/ytproxy.conf`

```
add_header Access-Control-Allow-Origin *;
add_header Access-Control-Allow-Headers *;
if ($request_method = OPTIONS ) {
   return 200;
}
proxy_buffering on;
proxy_set_header Host $arg_host;
proxy_ssl_server_name on;
proxy_set_header X-Forwarded-For "";
proxy_set_header CF-Connecting-IP "";
proxy_hide_header "alt-svc";
sendfile on;
sendfile_max_chunk 512k;
tcp_nopush on;
aio threads=default;
aio_write on;
directio 2m;
proxy_hide_header Cache-Control;
proxy_hide_header etag;
proxy_http_version 1.1;
proxy_set_header Connection keep-alive;
proxy_max_temp_file_size 0;
access_log off;
proxy_pass http://unix:/var/run/ytproxy/http-proxy.sock;
```

Now, create a site configuration file:

`/etc/nginx/sites-available/piped.conf`

```
server {
    listen 80;
    listen 443 ssl http2;
    ssl_certificate /etc/ssl/certs/kavin.rocks.pem;
    ssl_certificate_key /etc/ssl/private/kavin.rocks.key;
    ssl_early_data on;
    server_name pipedapi.kavin.rocks; # Change this depending on what domain you are using

    location / {
       proxy_pass http://127.0.0.1:8080;
    }
}

server {
    listen 80;
    listen 443 ssl http2;
    ssl_certificate /etc/ssl/certs/kavin.rocks.pem;
    ssl_certificate_key /etc/ssl/private/kavin.rocks.key;
    ssl_early_data on;
    server_name pipedproxy-bom.kavin.rocks; # Change this depending on what domain you are using

    location ~ (/videoplayback|/api/v4/|/api/manifest/) {
       include snippets/ytproxy.conf;
       add_header Cache-Control private always;
       proxy_hide_header Access-Control-Allow-Origin;
    }

    location / {
       include snippets/ytproxy.conf;
       add_header Cache-Control "public, max-age=604800";
       proxy_hide_header Access-Control-Allow-Origin;
    }
}
```

Finally, reload the nginx service and you are done!
