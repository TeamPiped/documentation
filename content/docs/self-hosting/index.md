---
title: "Self-Hosting"
weight: 4
summary: How can I Self-Host Piped?
---

# Self-Hosting

There are two simple ways to self-host Piped.

- [Bring your own reverse proxy](#docker-compose-nginx-aio-script) (**recommended**) - This is the recommended way to self-host Piped. You can use any reverse proxy you want, and must configure TLS certificates yourself.
- [Using Caddy](#docker-compose-caddy-aio-script) - This would use Caddy on port 80 and 443, and automatically configure TLS certificates for you. However, it would be difficult to host multiple services on the same server.

Because YouTube tends to ban IP addresses, it is recommended to either
- use the [IPv6 rotator made by the Invidious team](#ipv6-rotator-using-docker) - This is the recommended way because it's easier to set up and doesn't increase loading times.
- use a VPN or proxy between the Piped backend/proxy and YouTube.
to keep the instance running if you plan to make your instance public or to have a lot of users.

## Docker Compose Caddy AIO script

First, install `git`, `docker`, and the compose plugin for docker.

Run `git clone https://github.com/TeamPiped/Piped-Docker`.

Then, run `cd Piped-Docker`.

Then, run `./configure-instance.sh` and fill in the hostnames when asked. Choose `caddy` as the reverse proxy when asked.

Now, create `A` (and `AAAA`) records to your server's public IP with the hostnames you had filled in above.

Finally, run `docker compose up -d` and you're done!

Consider joining the federation protocol at https://github.com/TeamPiped/piped-federation#how-to-join

## Docker Compose Nginx AIO script

Note: This setup requires you to have your own reverse proxy in addition to the one provide, and requires you to configure TLS manually.

First, install `git`, `docker`, and the compose plugin for docker.

Run `git clone https://github.com/TeamPiped/Piped-Docker`.

Then, run `cd Piped-Docker`.

Then, run `./configure-instance.sh` and fill in the hostnames when asked.  Choose `nginx` as the reverse proxy when asked.

Now, create `A` (and `AAAA`) records to your server's public IP with the hostnames you had filled in above.

Run `docker compose up -d`.

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

Here's an example with Apache:
```
<VirtualHost *:443>
    ServerName hostname # For all 3 hostnames

    ProxyPass /  http://127.0.0.1:8080/  nocanon
    ProxyPassReverse /  http://127.0.0.1:8080/

    ProxyPreserveHost On

    AllowEncodedSlashes On

    # TLS configuration here
</VirtualHost>
```

Here's an example with Traefik using Docker compose labels:

This must be applied on the nginx container.

```
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.piped.rule=Host(`hostname`,`hostname2`,`hostname3`)"
  - "traefik.http.routers.piped.entrypoints=web"
```

Here's an example using Caddy:

```
hostname, hostname2, hostname3 {
    reverse_proxy http://127.0.0.1:8080
}
```


Finally, configure your TLS certificates if necessary. For nginx, you could use certbot with the nginx plugin.

Consider joining the federation protocol at https://github.com/TeamPiped/piped-federation#how-to-join

# Manually updating

Run `docker run --rm -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --run-once piped-frontend piped-backend piped-proxy varnish nginx caddy postgres watchtower`

## Docker Compose with system Nginx

**WARNING**: This setup is not recommended, as it is difficult to setup and maintain.

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
docker compose up -d
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

Now, create an nginx snippet like so:

`/etc/nginx/snippets/ytproxy.conf`

```
proxy_buffering on;
proxy_buffers 1024 16k;
proxy_set_header X-Forwarded-For "";
proxy_set_header CF-Connecting-IP "";
proxy_hide_header "alt-svc";
sendfile on;
sendfile_max_chunk 512k;
tcp_nopush on;
aio threads=default;
aio_write on;
directio 16m;
proxy_hide_header Cache-Control;
proxy_hide_header etag;
proxy_http_version 1.1;
proxy_set_header Connection keep-alive;
proxy_max_temp_file_size 32m;
access_log off;
proxy_pass http://unix:/var/run/ytproxy/actix.sock;
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

# IPv6 rotator using Docker

This requires you to already have set up a working Piped instance using Docker Compose as described above.

First of all, make sure that your server supports IPv6 by running `curl -m 5 ipv6.icanhazip.com` and confirm that an IPv6 address is returned as response. If it doesn't, this tutorial won't work.

Next, you need to enable IPv6 in Docker. First, edit `/etc/docker/daemon.json` and insert
```
{
  "experimental": true,
  "ip6tables": true
}
```
After, edit the `docker-compose.yml` of Piped and insert the following at the end of the file
```
networks:
  default:
    enable_ipv6: true
    ipam:
      config:
        - subnet: 2001:db8::/112
```
Now restart Docker. To confirm everything worked as expected and your instance now uses IPv6, run `docker exec -it piped-backend curl icanhazip.com`. If you see an IPv6 address here, you successfully force-enabled IPv6.

More information about Docker and IPv6 can be found at https://docs.docker.com/config/daemon/ipv6/.

Last but not least, the `Smart IPv6 Rotator` (credits to the Invidious team!) has to be set up. In order to do so, please follow the steps [at its GitHub repository](https://github.com/iv-org/smart-ipv6-rotator#how-to-setup-very-simple-tutorial) after installing the [required Python packages](https://github.com/iv-org/smart-ipv6-rotator#requirements). That's it, your Piped instance now periodically rotates its IPv6 address!
