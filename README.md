# Docker Registry with Minio

Docker Registry with Minio Using Docker Compose

## Getting Started

This section assumes that you know the basic usage of `Docker` and `Docker Compose`.

If you want to understand `docker-compose.yml` in this repository in detail, please refer to [DOCKER-COMPOSE_EXPLAINED.md](DOCKER-COMPOSE_EXPLAINED.md).


### Prerequisites

* Docker Compose


### Clone the Repository

Clone this repository using `git clone`

```
$ https://github.com/Pick1a1username/docker_registry_with_minio.git
```


### Edit `docker-compose.yml`

Set Minio's access key and secret.

(You can also use values already set if it's only for test)

```
MINIO_ACCESS_KEY: AKIAIOSFODNN7EXAMPLE
MINIO_SECRET_KEY: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```


### Create Directories

Create Directories required for apps.

```
$ cd docker_registry_with_minio
$ mkdir -p ./minio/data
$ mkdir -p ./minio/root/.minio
$ mkdir -p ./registry/certs
$ mkdir -p ./registry/etc/docker/registry # This must be already exist.
```


### Create Self-signed SSL Certificates

Create Self-signed SSL certificates so that Docker Registry use HTTPS.

```
$ openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout ./registry/certs/domain.key \
  -x509 -days 365 -out ./registry/certs/domain.crt
```


### Edit `config.yml` of Docker Registry

`config.yml` is included in this repository, and what you need to do is to specify Minio's keys so that Docker Registry can use Minio.


```
$ vi ./registry/etc/docker/registry/config.yml
$ cat ./registry/etc/docker/registry/config.yml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  # filesystem:
  #    rootdirectory: /var/lib/registry
  s3:
    accesskey: AKIAIOSFODNN7EXAMPLE
    secretkey: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    region: us-east-1
    regionendpoint: http://minio:9000
    bucket: registry
    encrypt: false
    secure: false
    v4auth: true
    chunksize: 5242880
    rootdirectory: /
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
$
```


### Run the Apps

Run the apps and test it.

```
$ docker-compose up -d
```

### Test Registry

Get into the Docker in Docker container, and pull and push a docker image.

Note that pushing a docker image from host doesn't work since host OS cannot resolve Minio's hostname(`minio`).


## References

### Docker Registry

* https://docs.docker.com/registry/insecure/
* https://gist.github.com/leanderjanssen/0e5532dc5818ab84b54b06cf80ad93ed
* https://medium.com/@enne/use-minio-as-docker-registry-storage-driver-c9c72c72cc87
* https://docs.docker.com/registry/storage-drivers/s3/#parameters


### Minio

* https://docs.minio.io/docs/minio-docker-quickstart-guide


### Docker in Docker

* https://hub.docker.com/_/docker


## Contributing

Any suggestions are welcome!


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
