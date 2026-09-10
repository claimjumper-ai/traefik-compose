# traefik-compose

Traefik v3 for one claimjumper.ai host: terminates TLS (Let's Encrypt HTTP-01)
and routes to the other stacks on the same box. One Traefik per host, no
shared proxy, no single point of failure across hosts.

## Deploy

Portainer git-ops stack, first on any host, before the stacks that need it:

- Repository: this repo, branch `main`, compose path `docker-compose.yml`
- Environment variables: see `.env.example` (`ACME_EMAIL`, optional `ACME_CASERVER`)

It creates the docker network `proxy`. Every other stack joins it:

```yaml
services:
  app:
    networks: [proxy]
    labels:
      - traefik.enable=true
      - traefik.http.routers.app.rule=Host(`app.claimjumper.ai`)
      - traefik.http.routers.app.entrypoints=websecure
      - traefik.http.routers.app.tls.certresolver=le
      - traefik.http.services.app.loadbalancer.server.port=8080
networks:
  proxy:
    external: true
```

## Certificates

Staging CA while a host is being rebuilt (untrusted certs, no rate limit).
To switch to production: remove `ACME_CASERVER`, delete the
`traefik_letsencrypt` volume on the host, redeploy.
