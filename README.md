# Docker Registry with Minio

Docker Registry with Minio Using Docker Compose

## Getting Started

This section assumes that you know the basic usage of `Docker` and `Docker Compose`.

If you want to understand `docker-compose.yml` in this repository in detail, please refer to comments in  [docker-compose.yml](docker-compose.yml).


### Prerequisites

* Docker Compose


### How it works?

There are three services: Docker Registry, Minio, Docker in Docker.

* Docker Registry: Storing docker images
* Minio: Object storage compatible with Amazon S3. Used for Docker Registry's storage.
* Docker in Docker: Used for pulling and pushing docker images to the Docker Registry.

As explained above, if an user pushes a docker image to the Docker Registry, Docker Registry will store the image to the Minio.


### Clone the Repository

Clone this repository using `git clone`

```
$ git clone https://github.com/Pick1a1username/docker_registry_with_minio.git
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
$ mkdir -p ./minio/data ./minio/root/.minio ./registry/certs ./registry/etc/docker/registry 
```


### Create Self-signed SSL Certificates

Create Self-signed SSL certificates so that Docker Registry use HTTPS.

At least `Common Name` must be same as Docker Registry's hostname, `registry`.

```
$ openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout ./registry/certs/domain.key \
  -x509 -days 365 -out ./registry/certs/domain.crt
```


Example:
```
$ openssl req \
>   -newkey rsa:4096 -nodes -sha256 -keyout ./registry/certs/domain.key \
>   -x509 -days 365 -out ./registry/certs/domain.crt
Generating a 4096 bit RSA private key
....++
...............................................................++
writing new private key to './registry/certs/domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:registry
Email Address []:
$
```

### Edit `config.yml` of Docker Registry

`config.yml` is included in this repository, and what you need to do is to specify Minio's keys set above to `accesskey` and `secretkey` so that Docker Registry can use Minio.


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
    # Refer to the following link for details:
    # https://docs.docker.com/registry/storage-drivers/s3/#parameters
    accesskey: AKIAIOSFODNN7EXAMPLE
    secretkey: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    # us-east-1 set below is dummy.
    region: us-east-1
    regionendpoint: http://minio:9000
    bucket: test0
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

### Create a Network

```
$ docker network create registry0
$ docker network ls
```


### Run the Apps

Run the apps and test it.

```
$ docker-compose up -d
```

### Test Registry

Get into the Docker in Docker container, and pull and push a docker image.

Note that pushing a docker image from host doesn't work since host OS cannot resolve Minio's hostname(`minio`).

```
$ docker exec -it <Container ID of Docker in Docker> sh
# docker pull ubuntu
# docker tag ubuntu registry:443/ubuntu
# docker push registry:443/ubuntu
```


## References

### Docker Registry

* https://hub.docker.com/_/registry/
* https://docs.docker.com/registry/insecure/
* https://gist.github.com/leanderjanssen/0e5532dc5818ab84b54b06cf80ad93ed
* https://medium.com/@enne/use-minio-as-docker-registry-storage-driver-c9c72c72cc87
* https://docs.docker.com/registry/storage-drivers/s3/#parameters


### Minio

* https://docs.minio.io/docs/minio-docker-quickstart-guide


### Docker in Docker

* https://hub.docker.com/_/docker

### Etc.

* https://gist.github.com/PurpleBooth/109311bb0361f32d87a2

## Contributing

Any suggestions are welcome!


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
