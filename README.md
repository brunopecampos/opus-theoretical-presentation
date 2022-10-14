# Projeto da apresentação teórico de Docker

### Bruno Campos

### https://github.com/brunopecampos

### Opus Software

### Slides utilizados podem ser encontrados em https://docs.google.com/presentation/d/1uCOvtNze2oXrbL97uKUwGO1dMEWF5_-6edjIvSvkCPk/edit#slide=id.g16939117430_0_0

# Script dos exemplos usados

## Tópicos básicos sobre contêineres

### Pull vs Create vs Run

docker pull hello-world\
docker image ls\
docker container ls\
docker create hello-world\
docker container ls -a\
docker run hello-world

### Iterativos vs Detach

docker run --name word --rm wordpress\
ctrl+c\
docker run --rm --name word -d wordpress\
docker rm -f word

### Repositórios Alternativos

docker pull quay.io/dockerinaction/ch4_hello_registry:latest

### Exec, Entrypoints, Variáveis de Ambiente e Tolerância à falha

docker run --rm --name word --env MY_VAR=opus_software wordpress\
docker exec -it word /bin/sh\
echo $MY_VAR\
exit\
docker run --rm --name word --entrypoint echo wordpress "Hello World"\
docker run --name word --restart always -it wordpress echo "opus"
docker logs word

### logs e inspect

docker run --name word -d wordpress\
docker logs word\
clearkd\
docker run --name word --label AUTHOR=bruno -d wordpress\
docker inspect word | less\
docker inspect word | grep AUTHOR

## Volumes e Armazenamento

### Bind Mounts

docker run --mount type=bind,src=$PWD/volumes/bind,dst=/home -it mysql /bin/sh\
cd /home\
ls\
\# cria outro terminal \
cd /volumes/bind\
touch opus.txt\
ls\
touch opus.txt

### Tmpfs

docker run --mount type=tmpfs,dst=/home --name my -it mysql /bin/sh\
cd home \
touch opus.txt\
exit\
docker restart my\
docker exec my ls /home

### Volumes

docker volume create my-volume\
docker volume ls\
docker run --mount type=volume,src=my-volume,dst=/home --name mysql1 -it mysql /bin/sh\
cd /home\
\#Abre outro terminal\
docker run --mount type=volume,src=my-volume,dst=/home --name mysql2 -it mysql /bin/sh\
cd /home\
touch opus.txt\
\# no outro terminal
ls

### Herança de Volumes e volumes anonimos

docker volume create my-volume2\
docker volume create my-volume3\
docker create --mount type=volume,src=my-volume1,dst=/home --mount type=volume,src=my-volume2,dst=/tmp --name my-container1 mysql\
docker create --mount type=volume,src=my-volume3,dst=/opt --name my-container2 mysql\
docker create --volumes-from my-container1 --volumes-from my-container2 --name my-container3 mysql\
docker inspect my-container3 | less\
\# Mostrar volume anonimo do mysql\
docker volume ls

## Redes Docker

### Redes Host e Nobody

docker network ls\
docker run --name nettools --network none -it --rm praqma/network-multitool /bin/sh\
ifconfig\
exit\
docker run --name nettools --network host -it --rm praqma/network-multitool /bin/sh\
ifconfig\
exit\
\#Abre outro terminal\
ifconfig

### Redes Bridge

docker network create --driver bridge --attachable network1\
docker run --name nettools1 -d --rm praqma/network-multitool\
docker run --name nettools2 -d --rm praqma/network-multitool\
docker network connect network1 nettools1\
docker network connect network1 nettools2\
docker exec -it nettools1 /bin/sh\
ifconfig\
\# Abre outro terminal\
docker exec -it nettools2 /bin/sh\
\# Abre outro terminal\
sudo wireshark\
\# Configura para interface any, tcp.port = 12000\
\# Volta pro primeiro terminal\
ifconfig\
nc -vlkp 12000\
\# Volta pro segundo terminal\
ifconfig\
nc -v nettools1 12000

### Encaminhamento de Porta

docker run --name nettools -p 9000:9000/tcp -it --rm praqma/network-multitool /bin/sh \
\# Abre outro terminal\
sudo wireshark\
\# escolher any e tcp.port = 9000\
\# Volta pro primeiro terminal\
nc -vlkp 9000\
\# Volta pro segundo terminal\
nc -v localhost 9000

## Controle de Recursos

### Memória

docker run -it --rm --name forkbomb --memory=2G python /bin/sh\
\# Segundo terminal \
docker cp fork-bomb.py forkbomb:/\
docker stats forkbomb \
\# Primeiro terminal \
ls \
python fork-bomb.py

### CPU

docker run --name cpu1 --cpu-shares 1024 -d --rm progrium/stress --cpu 12\
docker run --name cpu2 --cpu-shares 512 -d --rm progrium/stress --cpu 12\
docker stats cpu1\
\# Segundo terminal\
docker stats cpu2\
docker rm -f cpu1 cpu2

### Driver

\# Desligar a câmera da apresentação\
docker run --name=webcam -d --rm -p 8080:8080 -p 8082:8082 --device /dev/video0:/dev/video0 romankspb/webcam \
\# Ligar câmera de novo

### Memória compartilhada

\# Mostrar códido em c \
vim ipc.c\
\# Rodar sem ipc no segundo container\
docker container run -d --rm --name producer --ipc shareable dockerinaction/ch6_ipc -producer\
docker container run -d --rm --name consumer --ipc container:producer dockerinaction/ch6_ipc -consumer\
docker logs producer\
docker logs consumer

### Operações do Kernel

docker run --name word -it --rm --cap-drop chown wordpress /bin/sh\
touch a.txt\
chown nobody a.txt

## Criação de Imagem

### Sistema de camadas\Tags

docker run --name word -it wordpress /bin/sh\
cd /etc\
rm magic.mime\
echo a >> hosts\
touch a.txt\
exit\
docker diff word\
docker container commit -m "Make my container" word my-word\
docker image ls\
docker tag my-word my-word:1.0\
docker image ls

### Dockerfile básico

\# mostrar Dockerfile_basico.df\
docker image build --file Dockerfile_basico.df -t dockerfile_basico .\
docker run dockerfile_basico\
docker run dockerfile_basico"Command não executado"

### Dockerfile Downstream

vim Dockerfile_downstream.df\
docker image build --file Dockerfile_downstream.df -t dockerfile_downstream .\
docker run dockerfile_downstream\

### Dockerfile Multi Image

vim Dockerfile_mult_img.df\
docker image build --file Dockerfile_mult_img.df -t dockerfile_tgt-1 --target first_target .\
docker run dockerfile_tgt-1\
docker image build --file Dockerfile_mult_img.df -t dockerfile_tgt-2 --target second_target .\
docker run dockerfile_tgt-2

## Docker Compose

### Docker compose básico

vim docker-compose-yml\
docker compose up
