# nginx-proxy service

Containerized reverse proxy using [nginx](https://github.com/nginx-proxy/nginx-proxy/). The [docker-gen](https://github.com/jwilder/docker-gen) utility uses Docker's API to inspect containers and access their IP, Ports and other configuration meta-data, builds templates that are rendered and an optional notification command can be run to restart the service. Creation and renewal of SSL certificates are autometed using [acme-companion](https://github.com/nginx-proxy/acme-companion). For more info read [this post](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/).

## Setup

Create network:

    docker network create mcps-epcm-net

Create volumes:

    docker volume create mcps_nginx-proxy_certs
    docker volume create mcps_nginx-proxy_vhost
    docker volume create mcps_nginx-proxy_html
    docker volume create mcps_nginx-proxy_acme

Start nginx-proxy:

    docker run --detach \
        --name mcps_nginx-proxy \
        --publish 80:80 \
        --publish 443:443 \
        --volume mcps_nginx-proxy_certs:/etc/nginx/certs \
        --volume mcps_nginx-proxy_vhost:/etc/nginx/vhost.d \
        --volume mcps_nginx-proxy_html:/usr/share/nginx/html \
        --volume /var/run/docker.sock:/tmp/docker.sock:ro \
        nginxproxy/nginx-proxy:1.4-alpine

Start acme companion for issuing SSL certificates:

    docker run --detach \
        --name mcps_acme-companion \
        --volumes-from mcps_nginx-proxy \
        --volume /var/run/docker.sock:/var/run/docker.sock:ro \
        --volume mcps_nginx-proxy_acme:/etc/acme.sh \
        --env "DEFAULT_EMAIL=fernando@mcps-epcm.org \
        nginxproxy/acme-companion

