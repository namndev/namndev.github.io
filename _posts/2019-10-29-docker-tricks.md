---
layout: post
title: Docker tricks!
image: /img/hello_world.jpeg
---

[How to regenerate Dockerfile form image?](https://stackoverflow.com/questions/19104847/how-to-generate-a-dockerfile-from-an-image)

[How to move docker image from a host to anther host?](https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-using-a-repository)

[Some docker's command](https://medium.com/the-code-review/top-10-docker-run-command-options-you-cant-live-without-a-reference-d256834e86c1)

## Docker transcode v1 command:

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

Continues...