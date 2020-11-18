#self hosted data/git

nextcloud and gitlab with postgress, redis and traefik via docker. oh yes please. also object storage via minio.

### pre-requisite
```shell
$ git clone https://github.com/undefinedtea/data.git
$ cd data
```

this environment implements a seperation between traefik and other container service, allowing a microservice implementation.
consider to change the folder structure (namely com.undefinedtea.data.service) to reflect your environment and ammend any following command accordingly. the principle for me is that each service has it's own folder, which means that this environment can be accomodating to different scenarios.

to rename the folder `com.undefinedtea.data.service` to `com.domain.file.service` for instance (meaning that all occurances of data.undefinedtea.com should be replaces with file.domain.com later on to reflect the folder structure)
```shell
$ mv com.undefinedtea.data.service com.domain.file.service
```

in `network.service/docker-compose.yml` make sure the following `command` values reflect your enviornment.
```yml
commands:
  # [...]
  - "--pilot.token=${PILOT}"
  # [...]
  - "--certificatesresolvers.tmp.acme.email=hello@undefinedtea.dev"
  # [...]
```

in `network.service/docker-compose.yml` make sure the following `label` values reflect your enviornment.
```yml
labels:
  # [...]
  - traefik.http.routers.traefik.rule=Host(`api-pilot.undefinedtea.com`)
  # [...]
```

in `com.undefinedtea.data.service/docker-compose.yml` make sure the following `label` values reflect your enviornment.
```yml
labels:
  # [...]
  - traefik.http.routers.service.rule=Host(`data.undefinedtea.com`)
  # [...]
```

in `com.undefinedtea.git.service/docker-compose.yml` make sure the following `label` and `environment` values reflect your enviornment.
```yml
environment:
  GITLAB_OMNIBUS_CONFIG: |
    external_url 'https://git.undefinedtea.com'

    gitlab_rails['gitlab_ssh_host'] = "git.undefinedtea.com"
    gitlab_rails['time_zone'] = 'Europe/Helsinki'

    # [...]

    registry_external_url 'https://docker.git.undefinedtea.com'
    # [...]
    gitlab_rails['api_url'] = 'https://docker.git.undefinedtea.com'

    # [...]
labels:
  # [...]
  - traefik.http.routers.git.rule=Host(`git.undefinedtea.com`)
  # [...]

labels:
  # [...]
  - traefik.http.routers.docker.rule=Host(`docker.git.undefinedtea.com`)
  # [...]
```

in `com.undefinedtea.file.service/docker-compose.yml` make sure the following `label` values reflect your enviornment.
```yml
labels:
  # [...]
  - traefik.http.routers.file.rule=Host(`file.undefinedtea.com`)
  # [...]
```

any instance of `data.undefinedtea.com`, `git.undefinedtea.com`, `docker.git.undefinedtea.com` and `git.undefinedtea.com` should be replaced with your domain.

then initialise and ammend `.env` and `com.undefinedtea.data.service/cache` according to your environment. note that [pilot][1] integration and it's coresponging `PILOT` value in the `.env` file are optional. again, make sure to replace _every_ value in these files. password is not a secure password. seriously. replace _every_ value.
```shell
# edit .env
$ cp e.env .env
$ cp e.git.env .git.env
$ cp e.file.env .file.env

#edit com.undefinedtea.data.service/cache
$ cp com.undefinedtea.data.service/e.cache com.undefinedtea.data.service/cache
$ cp com.undefinedtea.git.service/e.cache com.undefinedtea.git.service/cache
```

### create network interface
to make network interfaces persistent, we create them via terminal.
```shell
$ docker network create --driver=bridge --attachable --internal=false traefik
$ docker network create data
$ docker network create git
```

verify the network interface and ammend `TRUSTED_PROXIES` in `.env` accordingly.
```shell
$ docker network inspect traefik
```

### create container instance
```shell
$ docker-compose --file network.service/docker-compose.yml up --detach
$ docker-compose --file com.undefinedtea.data.service/docker-compose.yml up --detach
$ docker-compose --env-file .git.env --file com.undefinedtea.git.service/docker-compose.yml up --detach
$ docker-compose --env-file .file.env --file com.undefinedtea.file.service/docker-compose.yml up --detach
```

### verify
```shell
$ docker ps --all
```

### redis
if you want to flush the cache, the following command will achieve that.
```shell
$ docker exec -it cache redis-cli -a password FLUSHALL
$ docker exec -it git-cache redis-cli -a password FLUSHALL
```



[1]: https://traefik.io/traefik-pilot/
