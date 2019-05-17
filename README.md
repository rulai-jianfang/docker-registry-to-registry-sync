## Some problems fixed when We sync from private to private registry v2 sync up, better using the python file to test on local, then run the container later

## Problem 1
### Running via Docker:

```
docker run --rm -it \
    -v $(pwd)/config.yml:/config.yml \
    -v /var/run/docker.sock:/var/run/docker.sock
    stefanhudelmaier/docker-registry-to-registry-sync
```
### Change docker.sock permission:
When run the python file in current environment directly, we need set /var/run/docker.sock's permission to 667 or 777 
When run with docker, then we need mount the /var/run/docker.sock into it
```
    -v /var/run/docker.sock:/var/run/docker.sock
```

## Problem 2
Need add dst user name and password to the config.xml, also the port number of the source and destination url
```
source_registry:
  url: https://source.xxx.com:5000
  username: xxxxx
  password: xxxxx


destination_registry:
  url: https://dst.xxx.com:5000
  username: yyyyy
  password: yyyyy
```





# docker-registry-to-registry-sync

Tool for syncing docker images from one registry to another

## Goal

Make is possible to sync all tags of a number of docker repositories
from one registry to another. Can be useful if you have an interal docker
registry but want to sync a subset of the docker images to a globally 
accessibly registry.

## Usage

### Configuration

Create a `config.yml` file. Example:

```yaml
source_registry:
  url: https://docker.example.com:5000
  username: some_user
  password: some_password

destination_registry:
  url: http://127.0.0.1:5000
    username: some_user
  password: some_password

repositories:
  - company/super-project
  - company/cool-project
```

The meaning of the settings:

* `source-registry`: The registry to sync from
* `destination-registry`: The registry to sync to
* `repositories`: A list of docker repositores, consisting of the repository name
  and image name (without the registry part).
  
When the images are synced, the registry part of the tag is replaced, e.g.
in the example above the tag `docker.example.com/company/super-project` is
re-tagged as `127.0.0.1:5000/company/super-project` and pushed to `127.0.0.1:5000`.

### Passwords as environment variables

You can specify the registry passwords using the `SOURCE_REGISTRY_PASSWORD` 
and `DESTINATION_REGISTRY_PASSWORD` environment variables instead of 
including them in the `config.yml` file.

### Running via Docker:

```
docker run --rm -it \
    -v $(pwd)/config.yml:/config.yml \
    -v /var/run/docker.sock:/var/run/docker.sock
    stefanhudelmaier/docker-registry-to-registry-sync
```
### Change docker.sock permission:
When run the python file in current environment directly, we need set /var/run/docker.sock's permission to 667 or 777 
When run with docker, then we need mount the /var/run/docker.sock into it
```
    -v /var/run/docker.sock:/var/run/docker.sock
```


When using environment variables for the passwords:

```
docker run --rm -it \
    -e SOURCE_REGISTRY_PASSWORD=secret \
    -e DESTINATION_REGISTRY_PASSWORD=secret \
    -v $(pwd)/config.yml:/config.yml \
    stefanhudelmaier/docker-registry-to-registry-sync
```

See [Image on Docker Hub](https://hub.docker.com/r/stefanhudelmaier/docker-registry-to-registry-sync/)

## Known limitations

* Does not work with Docker Hub due to a problem in the used registry client, see https://github.com/yodle/docker-registry-client/issues/42
