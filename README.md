# mirror-swarm

openSUSE Brazil Docker Swarm deployment

## Assumptions

- The host path where files are going to be stored is `/storage/opensuse`
- The owner of `/storage/opensuse` is the user:groud defined by `RSYNC_CHOWN`
- The Swarm cluster is empty
- The load balancer/proxy/ingress is Traefik
- The domain name is configured, uses CloudFlare DNS and Let's Encrypt for certificates

## Deployment

The deployment must follow an order, as services depend on external configuration.

### Volume

Create a volume named `opensuse` that points to `/storage/opensuse`

```bash
docker volume create --name opensuse --opt type=none --opt device=/storage/opensuse --opt o=bind
```

Create a volume named `fancyindex`, this allows to edit the nginx fancyindex HTMLs.

```bash
docker volume create fancyindex
```

### Network

Create a network for Traefik and all services.

```bash
docker network create --driver overlay traefik
```

### Configuration

Create a config named `rsync-include` that will hold the rsync include pattern of what will be mirrored from openSUSE servers.

```bash
docker config create rsync-include rsync-include.txt
```

Create a config named `cf-dns-api-token` contains the CF API Token.

```bash
docker config create cf-dns-api-token cf-dns-api-token.txt
```

Create a config named `traefik-users-file` contains the `user:password` for Traefik Web UI.

```bash
docker config create traefik-users-file traefik-users-file.txt
```

### Deployment

Deploy Traefik first as other services may depend on it.

```bash
docker stack deploy -c traefik.yml traefik
```

Deploy CronJob as other services may depend on it.

```bash
docker stack deploy -c cronjob.yml cronjob
```

Deploy mirror services.

```bash
docker stack deploy -c mirror.yml mirror
```
