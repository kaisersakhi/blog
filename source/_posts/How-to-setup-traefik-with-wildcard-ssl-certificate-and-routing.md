---
title: How to setup traefik with wildcard SSL certificates and routing
date: 2025-04-28 15:01:20
tags: traefik, reverse proxy, backend
---

I started working on a multi-tenant system where I had to assign domains on the fly. I was previously using Cloudflare Tunnel, and I have to tell you — it’s a breeze to use. Amazing project. But I like power and don't want to depend on an opaque dependency, so I started looking into how I could set this system up myself. That’s when I discovered Traefik.

Initially, I faced a lot of issues understanding the configuration combined with the Docker setup, but eventually, I got it.

Let's say you are serving your app on yourdomain.com, but this app is a multi-tenant app, and you want to separate tenants based on subdomains. So, when a tenant signs up for your service, you want to assign them a subdomain on the fly, or let the new tenant choose a new subdomain.

Let's see how we can do this.

## Setting up `traefik:v3`

```yml
# docker-compose.yml
services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - shared_net # Docker network to keep all the apps on the same network.
    ports:
      - "80:80"
      - "443:443"
      - "443:443/tcp" # Uncomment if you want HTTP3
      - "443:443/udp" # Uncomment if you want HTTP3
    environment:
      CF_API_TOKEN: ${CF_DNS_API_TOKEN} # Create
    env_file: .env
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./data/traefik.yml:/traefik.yml:ro
      - ./data/acme.json:/acme.json
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.middlewares.traefik-https-redirect.redirectscheme.scheme=https"
      - "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.routers.traefik.middlewares=traefik-https-redirect"
      - "traefik.http.routers.traefik-secure.entrypoints=https"
      - "traefik.http.routers.traefik-secure.tls=true"
      - "traefik.http.routers.traefik-secure.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik-secure.service=api@internal"

networks:
  shared_net:
    external: true
```

_NOTE_: Create new api token on Cloudflare Dashboard and set that in `.env` file named as `CF_DNS_API_TOKEN`.

## Setting up `traefik.yml`

As you can see in the above docker compose, I have kept `traefik.yml` inside data sub-folder, let me show you how it looks like.

```yml
# data/traefik.yml
api:
  dashboard: true
  debug: true
entryPoints:
  http:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: https
          scheme: https
  https:
    address: ":443"
serversTransport:
  insecureSkipVerify: true
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
certificatesResolvers:
  cloudflare:
    acme:
      email: mail@kaisersakhi.com
      storage: acme.json
      caServer: https://acme-v02.api.letsencrypt.org/directory
      dnsChallenge:
        provider: cloudflare
        disablePropagationCheck: true # Disable propagation check if needed
        delayBeforeCheck: 60s
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
log:
  level: DEBUG
```

## Lastly, Setting up Docker app

Now, we can setup the container.

```yml
services:
  myapp:
    build:
      context: .
      dockerfile: Dockerfile
    env_file:
      - .env
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp-web.rule=HostRegexp(`[a-z]+\\.mydomain\\.com`) || Host(`mydomain.com`)"

      - "traefik.http.routers.myapp-web.tls=true"
      - "traefik.http.routers.myapp-web.tls.certresolver=cloudflare"

      # Explicitly define cert resolver for both the apex and wildcard domains
      - "traefik.http.routers.myapp-web.tls.domains[0].main=mydomain.com"
      - "traefik.http.routers.myapp-web.tls.domains[0].sans=*.mydomain.com"

      # Service definition
      - "traefik.http.services.myapp-web.loadbalancer.server.port=80"
    restart: unless-stopped
    networks:
      - shared_net

networks:
  shared_net:
    external: true
```
