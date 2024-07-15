# Caddy CNAME Cloudflare DNS-01 Challenge

Configuration of a [Caddyserver](https://caddyserver.com/) with DNS-01 Challenge with CNAME Record on [Cloudflare](https://cloudflare.com/).

Useful for you who use an unsupported DNS provider and just want to delegate (?) the DNS-01 challenge to Cloudflare.

## Preresquite

1. Write access to your unsupported DNS Provider records;
2. Write access to a domain that you want to use for DNS-01 Challenge, which you will use Cloudflare on;
3. Write access to your Cloudflare account;
4. A server with Docker installed on.

## Usage

Before you applying the following docker-compose and Caddyfile, make sure you set a CNAME in your main domain. 

For example, if I want to create cert for a subdomain of my main domain, `dev.ndkprd.com`, and the domain I delegated to Cloudflare is `ndkprd.com`, I created a CNAME pointint `_acme-challenge.dev.ndkprd.com` to `_acme-challenge.dev.ndkprd.my.id`.

### Docker Compose Example

```yaml
---
# ./docker-compose.yaml

version: '3.7'

services:

  caddy-cloudflaredns:
    image: ndkprd/caddy-cloudflaredns:v0.0.1
    container_name: caddy
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy/Caddyfile:/etc/caddy/Caddyfile
      - ./caddy/data:/data
      - ./caddy/config:/config
    env_file:
      - caddy/.env

  whoami:
    image: traefik/whoami
    container_name: whoami
    restart: unless-stopped
    ports:
      - 8081:80
```

>[!Caution]
> Please don't use my container image, since I probably won't update it. Instead, you can build it on your own using the following [Dockerfile](Dockerfile_caddy).


With the `caddy/.env` file:

```sh
CLOUDFLARE_EMAIL=<contact@ndkprd.com>
CLOUDFLARE_API_TOKEN=<my-super-secret-token>
ACME_AGREE=true
```

### Caddyfile Example

```
*.dev.ndkprd.com {
    tls contact@ndkprd.com {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
        dns_challenge_override_domain _acme-challenge.dev.ndkprd.my.id
        resolvers 1.1.1.1 1.0.0.1
        # Let's Encrypt staging directory, use this for testing, to
        # mitigate being rate limited by Let's Encrypt.
        # ca https://acme-staging-v02.api.letsencrypt.org/directory
        ca https://acme-v02.api.letsencrypt.org/directory
    }
    reverse_proxy whoami:80
    encode gzip
}
```
