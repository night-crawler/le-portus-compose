#### Install nginx-proxy
[docker-compose-letsencrypt-nginx-proxy-companion](https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion/blob/master/docker-compose.yml)

Ensure your nginx-proxy has its own network:

```bash
docker network create --attachable --driver overlay --subnet=10.0.0.0/24 nginx-network
``` 

#### Configure sample
```bash
cp .env.sample .env
editor .env
. .env
```

#### Create a separate docker-registry network:
```bash
docker network create --attachable --driver overlay --subnet=10.64.64.0/24 docker-registry-network
```

#### Configure `vhost.d/${REGISTRY_FQDN}_location`:
```ini
proxy_set_header  Host              $http_host;   # required for docker client's sake
proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header  X-Forwarded-Proto $scheme;
proxy_read_timeout                  900;
```

#### Configure `vhost.d/${PORTUS_FQDN}`:
```ini
## End of configuration add by letsencrypt container
# disable any limits to avoid HTTP 413 for large image uploads
client_max_body_size 0;

# required to avoid HTTP 411: see Issue #1486 (https://github.com/moby/moby/issues/1486)
chunked_transfer_encoding on;
```

#### Run
```bash
docker-compose up
```
