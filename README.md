# mirror-swarm

openSUSE Brazil Docker Swarm deployment

## Assumptions

- The storage disk has at least 1 TB free
- The host path where files are going to be stored is `/storage/opensuse`
- The owner of `/storage/opensuse` is the user:groud defined by `RSYNC_CHOWN`
- The Swarm cluster is empty
- The load balancer/proxy/ingress is Traefik
- The domain name is configured, uses CloudFlare DNS and Let's Encrypt for certificates

## Setup

Before deployment you need to setup and configure some items.

### Network

Create a network for Traefik and all services.

```bash
docker network create --driver overlay traefik
```

### Volume

Create the data volumes.

```bash
docker volume create --name opensuse-data --opt type=none --opt device=/storage/opensuse --opt o=bind
docker volume create --name packman-data --opt type=none --opt device=/storage/packman --opt o=bind
```

Create the fancyindex volumes, this allows to edit the nginx fancyindex HTMLs.

```bash
docker volume create opensuse-fancyindex
docker volume create packman-fancyindex
```

### Configuration

Create a config that contains the [CF API Token](https://developers.cloudflare.com/api/tokens/create/).

```bash
docker config create cf-dns-api-token cf-dns-api-token.txt
```

Create a config that contains the `user:password` for Traefik Web UI.

```bash
docker config create traefik-users-file traefik-users-file.txt
```

> Use `htpasswd -n <user>` to create an encoded password.

Create the configs that will hold the rsync include pattern.

```bash
docker config create opensuse-rsync-include opensuse-include.txt
docker config create packman-rsync-include packman-include.txt
```

## Deployment

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
docker stack deploy -c opensuse-mirror.yml opensuse
docker stack deploy -c packman-mirror.yml packman
```
