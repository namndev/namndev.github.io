---
layout: post
title: Docker tricks!
image: /img/docker/docker_icon.jpeg
---

Since its introduction in 2013, Docker quickly became the world’s most popular container management system. From fancy start-ups to a more traditional use cases in large banks and insurance companies. From development, testing, staging to production environments.
    
    ![Docker Architecture](/img/docker/docker_architecture.png)

In this article I just want to share tips and tricks learned using and enjoying Docker.

## 1. Tailing logs
In a perfect world you would have an [ELK](https://www.elastic.co/elk-stack), [Graylog](https://www.graylog.org/) or even [OK Log](https://github.com/oklog/oklog) installation with all your logs aggregated where easily do a full text search. But sometimes knowing how to quickly access your Docker logs can be really useful:

```bash
$ sudo docker logs -t --tail 1000 my_postgres 2>&1 | grep -i error
```

In previous example we are searching for `error` (case-insensitive) in the last 1000 log lines of `my_postgres` container adding the timestamp at the beginning of each line.

## 2. How to backup and restore PostgreSQL databases

Every command that you can run using a database client you can also do it using Docker. In this example you’ll see how to back up/restore a Postgres database.

Docker Postgres backup command:

```bash
$ docker run -i -e PGPASSWORD=[POSTGRESQL_PASSWORD] postgres /usr/bin/pg_dump \
 -h [POSTGRESQL_HOST] \
 -U [POSTGRESQL_USER] [POSTGRESQL_DATABASE] | gzip -9 > backup.sql.gz
```

Docker Postgres restore command:

```bash
$ gunzip < backup.sql.gz | docker exec -i [POSTGRESQL_CONTAINER] \
/bin/bash -c "export PGPASSWORD=[POSTGRESQL_PASSWORD] && /usr/bin/psql -U [POSTGRESQL_USER] [POSTGRESQL_DATABASE]"
```

In previous example we are searching for error (case-insensitive) in the last 1000 log lines of my_postgres container adding the timestamp at the beginning of each line.

### 3. Copy and paste files

Although it is better to use volumes in order to provide content to a container sometimes you need to do it manually. Docker allows you to do it in both ways, from host to a container and vice versa:

**From host to a Docker container**

At compile time:

```bash
# Within a Dockerfile
COPY script.sh /tmp
ADD script.sh /tmp
```

Add and copy perform the same task but add have two more capabilities:

```bash
# Dockerfile
# 1 - Be able to automatically untar files
ADD scripts.tar.gz /tmp
# 2 - Fetching files from remote URLs
ADD http://www.example.com/script.sh /tmp
```

At runtime:

```bash
# Copies script.sh from our current host folder to /tmp inside of the container.
$ docker cp script.sh container_name:/tmp/
$ docker exec -it container_name bash -c 'tree -a /tmp'
>
/tmp
└── script.sh
```

**From Docker container to a host**

```bash
# Copies script.sh from the inside of the container /tmp/script.sh to your current host folder.
$ docker cp container_name:/tmp/. .
$ tree -a
>
└── /tmp/script.sh
```

### 4. Oh.. no no.. wait!

Sometimes services can depend on each other and cannot start all at the same time.

In docker-compose there is a command called depends_on but it’s not recommended because it can’t ensure that a service is fully started and initialized before starting a dependent one.

To make sure a service is fully available you should write a custom wait expression. Here it is an example:

```dockerfile
version: '3'
networks:
  test_net:

services:
  webserver:
    image: nginx:alpine
    command: sh -c 'sleep 10 && nginx -g "daemon off;"'
    networks:
      - test_net

  test:
    image: alpine
    command:  sh -c 'while ! nc webserver 80; do echo '.' && sleep 1; done && echo "Webserver up and running, bye bye..!"'
    networks:
      - test_net
```

In previous example test will be waiting until *webserver* is up and running to finally display *“Webserver up and running, bye bye..!”*.

## 5. Yay! Let’s Encrypt certs!

If you are using [Let’s Encrypt](https://letsencrypt.org/) certificates you can easily use [quay.io/letsencrypt](http://quay.io/letsencrypt/letsencrypt) container to create/renew them.

From your host machine you have to first stop your webserver and then run the following command:

```bash
$ sudo docker run -it -rm -p 443:443 -p 80:80 -name certbot \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  quay.io/letsencrypt/letsencrypt:latest certonly
```

Follow the command line instructions in order to create them. This will create the certs in `/etc/letsencrypt/live/[my_domain]/*`. Copy them to your certs folder:

```bash
$ cp /etc/letsencrypt/live/[my_domain]/* /path/to/certs/folder/*
```

Finally, restart the webserver and open your browser to check if the certificate has been renewed.

## 6. The fastest way to build a REST API
When prototyping, don’t lose time creating an API REST, just run [Docker-Json-server](https://github.com/clue/docker-json-server), which is based on NPM package [json-server](https://github.com/typicode/json-server). Here’s an example:

```bash
#!/usr/bin/env bash

echo '{"cities": [{"name": "Barcelona"}, {"name": "Copenhagen"}, {"name": "Edinburgh"}, {"name": "Hanoi"}]}' > /tmp/cities.json
docker run -d -p 80:80 -v /tmp/cities.json:/data/db.json clue/json-server

# Listing my favourite cities
curl http://localhost/cities
[
  {"name": "Barcelona"},
  {"name": "Copenhagen"},
  {"name": "Edinburgh"},
  {"name": "Hanoi"}
]

# Listing my favourite cities containing *bar*
curl 'http://localhost/cities?name_like=bar'
[
  {"name": "Barcelona"}
]
```

## 7. Running Cron tasks on Docker Alpine container

Docker Alpine is a popular lightweight Docker image that contains a basic list of Linux utilities. One of this utilities is crond. These are the default crond periodic options:

```bash
$ docker run -it alpine ls /etc/periodic
>
15min daily hourly monthly weekly
```

In order to run your tasks periodically you can mount your scripts based on when you want to execute it. In this example, scripts placed in `cron_tasks_folder` will be executed hourly:

```dockerfile
version: '3'
services:
  cron:
  image: alpine:3.6
  command: crond -f -l 8
  volumes:
   - ./cron_tasks_folder:/etc/periodic/hourly/:ro
```
## 8. Using volumes in docker-compose

Docker allows you to use two types of volumes. Host mounted volumes and named volumes.

**Host mounted volumes**

Syntax: `/host/path:/container/path`

Host path can be defined as an absolute or as a relative path. You should use this type of volumes when you are running your infrastructure in one single machine (using Docker or Docker-compose).

**Named volumes**

Syntax: `named_volume_name:/container/path`

Named volumes can be defined as internal (default) or external. You can use this types of volumes when you are running your infrastructure in one single machine or in a cluster of machines (using Docker, Docker-compose or Docker swarm).

Docker volumes can be internal or external. Internal volumes has the scope of a single docker-compose file while external can be defined/used across the infrastructure and you must declare them before starting your services:

```bash
$ docker volume create --driver local \
 - opt type=none \
 - opt device=/var/opt/my_website/dist \
 - opt o=bind web_data
```
## 9. Using `.dockerignore` file
Many of you are familiar with `.gitignore` file. The concept of `.dockerignore` is similar but this time is used to build better Docker images. With this file you can avoid uploading unnecessary files reducing build time and image size.

First step is to create a `.dockerignore` file adding one line with `.dockerignore` itself (we don’t need to add this file to the image) including all the folders that you want to avoid in the final image. Example:

```bash
# .dockerignore
logs/
MY_SECRET_PASSWORDS.txt
.dockerignore
```

Whereas previous file does the trick you could be more concise. To achieve that you should reject all the files and folders `**` and only add exactly what you need (in this example `src` folder):

```bash
$ cat .dockerignore
**
!src
```

## 10. Housekeeping

It is a good idea to control Docker consumed resources. Running docker stats you’ll know about CPU, memory and network usages and if some container is not performing well you can always stop it. But in the previous indicators there is no information about disk consumption.
The main cause of filling up the disk is the size of the Docker images. With docker image ls you’ll have a list of current Docker images. Using the previous command there is no way to sort images by disk usage but you can use the following command to list images sorted by size ascending, displaying size and image tag. Useful to see what are the heaviest images:

```bash
$ curl -s --unix-socket /var/run/docker.sock http:/images/json | jq '.[] | "\(.Size) \(.RepoTags[])"' test.json | sort -V
```

Once you have identified which images you want to delete you can run the following command:

```bash
$ docker rmi [image_repo]:[image_tag]
```

In development environments it’s easy to end up having a lot of unused images and it’s better to delete multiple images at the same time. Command to remove all unused images:

```bash
$ docker image prune -a
```

Finally, Docker has a command to delete all stopped containers, dangling images, networks, unused volumes and build cache at the same time:

```bash
$ docker system prune -a --volumes
```


## 11. Other tricks

- [How to regenerate Dockerfile form image?](https://stackoverflow.com/questions/19104847/how-to-generate-a-dockerfile-from-an-image)
- [How to move docker image from a host to anther host?](https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-using-a-repository)
- [Some Docker's Command](https://medium.com/the-code-review/top-10-docker-run-command-options-you-cant-live-without-a-reference-d256834e86c1)

### Docker transcode v1 command:

```bash
docker run --name  encode_audio_worker  \
--add-host transcode-controller.internal:10.0.20.1  \
--add-host ftp_internal_storage.local:118.69.82.189 
-e "URL_INPUT_STORAGE_PREFIX=ftp://transcode:16%2F05%2F91@ftp_internal_storage.local/transcode_temp/" \
-e "URL_OUTPUT_STORAGE_PREFIX=ftp://transcode:16%2F05%2F91@ftp_internal_storage.local/ramdisk/" \
-v /data/pureftp/ftpusers:/home/ftpusers transcode/worker-gpu:latest encode_audio_worker
```

```bash
docker run -d --name  post-process-worker \
--add-host transcode-controller.internal:10.0.20.1 \
--add-host ftp_internal_storage.local:118.69.82.189 \
-e "URL_INPUT_STORAGE_PREFIX=ftp://transcode:16%2F05%2F91@ftp_internal_storage.local/transcode_temp/" \
-e "URL_OUTPUT_STORAGE_PREFIX=ftp://transcode:16%2F05%2F91@ftp_internal_storage.local/ramdisk/" \
-v /data/pureftp/ftpusers:/home/ftpusers transcode/worker-gpu:latest post_process_worker
```

### Docker swarm:
- [Traefik Library Image](https://github.com/containous/traefik-library-image)
- [Docker stack deploy](https://docs.docker.com/engine/reference/commandline/stack_deploy/)
- [Update only one service in docker swarm](https://stackoverflow.com/questions/44811886/restart-one-service-in-docker-swarm-stack)
- [List network in docker swarm](https://blog.alexellis.io/docker-stacks-attachable-networks/)
- [Add new container to swarm](https://stackoverflow.com/questions/51199134/adding-a-service-to-a-stack-after-the-stack-has-been-deployed) (yaml is only used when want to restart swarm)
- [Add labels to docker swarm service](https://stackoverflow.com/questions/52642677/how-can-i-add-labels-to-deploy-through-docker-service-create)
- [Crontab trick in docker](https://askubuntu.com/questions/85558/verify-if-crontab-works)
- [How to run a cron job inside a docker container?](https://stackoverflow.com/questions/37458287/how-to-run-a-cron-job-inside-a-docker-container)
- [How can I use docker without sudo?](https://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo)
- [How to copy a file from docker container to host](https://stackoverflow.com/questions/22049212/copying-files-from-docker-container-to-host)
- [How to see docker log realtime](https://blog.jongallant.com/2017/11/delete-docker-container-log-files/)

That’s all! Any comments will be appreciated and please let me know your Docker tips and tricks as well :)

> Thanks!