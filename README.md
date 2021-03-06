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


### Create a Bucket

Connect to `localhost:9000` with a web browser, and create a bucket named `registry`.

Refer to [docker-compose.yml](docker-compose.yml) to get the access key and secret key.

UI of Minio is so simple that you should be able to create a bucket even if you have no experience.


### Test Registry

Get into the Docker in Docker container, and pull and push a docker image.

Note that pushing a docker image from host doesn't work since host OS cannot resolve Minio's hostname(`minio`).

```
$ docker exec -it <Container ID of Docker in Docker> sh
# docker pull alpine \
&& docker tag alpine registry:5000/alpine \
&& docker push registry:5000/alpine
```

If it fails, try to restart containers by `docker-compose down` and `docker-compose up -d` about three times.

I haven't yet found the reason why it fails at first time.


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
