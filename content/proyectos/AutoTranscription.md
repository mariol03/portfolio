+++
date = '2024-12-08T18:13:33+01:00'
draft = false
title = 'AutoTranscription'
+++
Este proyecto se me ocurrió para poder alargar el tiempo que tiene un día para poder realizar más cosas en el mismo día. Usando esto puedo añadir un video a una carpeta de Google Drive, este automáticamente se descarga y se realiza la transcripción en un momento que no este usando el computo por GPU.
## Creación de las imágenes de docker.
Para realizar este proyecto he creado dos imágenes en docker, basandome en la imagen de n8n y añadiendo una herramienta de cli que necesitaba para una de estas.
``` [Dockerfile]
FROM docker.n8n.io/n8nio/n8n

USER root
RUN apk index
RUN apk add python3 py3-pip ffmpeg
RUN python3 -m pip install -U "yt-dlp[default]" --break-system-packages
USER node
```
Este fichero Dockerfile añade ffmpeg y yt-dlp a la image de n8n. Yt-dlp se instala usando el gestor de librerías de python ya que si no se instala una versión que funciona incorrectamente.  
Y también uso esta otra imagen para realizar la transcripción con el proyecto whisper de openai, esta basada en un ubuntu de nvidia para tener las librerias de CUDA.
``` [Dockerfile]
FROM nvidia/cuda:12.6.2-runtime-ubuntu24.04

# System installations
ARG DEBIAN_FRONTEND=none
RUN apt update; apt install -qq -y --no-install-recommends nodejs npm python3 python3-pip git ffmpeg

# n8n installation
RUN npm install n8n -g

# OpenAI/Whisper installation
# RUN pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118 --break-system-packages
# RUN pip3 install triton --break-system-packages
RUN pip3 install "git+https://github.com/openai/whisper.git" --break-system-packages

# startup
RUN groupadd node; useradd -g node node; mkdir /home/node; chown node:node /home/node; chmod 755 /home/node
WORKDIR /home/node
USER node
ENTRYPOINT [ "n8n" ]
```
## Automatizaciones
Una vez tenia las imágenes funcionando tuve que realizar una primera automatización que lo sube a una carpeta de un Nextcloud y luego usando los ficheros que están en esta carpeta realizar la transcripción.
![Flujo descarga](/images/Flujo2.png "Flujo descarga")
![Flujo transcripción](/images/Flujo1.png "Flujo transcripción")
