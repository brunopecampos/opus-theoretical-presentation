# Examplos da Apresentação Teórica de Docker

### Bruno Campos

### https://github.com/brunopecampos

### Opus Software

## Tópicos básicos sobre contêineres

### Pull vs Create vs Run

docker pull wordpress;\
docker image ls;\
docker container ls;\
docker image rm wordpress;\
docker create wordpress;\
docker container ls -a;\
docker image rm wordpress;\
docker run wordpress;\
docker image ls;\
docker ps;\

### Estados de um contêiner

docker container rm -f $(docker container ls -aq);\
docker create --name word wordpress
docker ps;\
docker pause word
docker ps;\
docker unpause word;\
docker stop word;\
docker ps -a;\
docker rm word;\
docker ps -a;\

### Iterativos vs Detach

docker run --name word wordpress;\
ctrl+c
docker run --name word -d wordpress\;
docker rm -f word;\
docker run --name word -it wordpress /bin/sh;\
exit
docker rm -f word;\

### Repositórios Alternativos

docker container rm -f $(docker container ls -aq) ; docker image rm $(docker image ls -q);\
docker pull quay.io/dockerinaction/ch3_hello_registry:latest

### Exec, Entrypoints, Variáveis de Ambiente e Tolerância à falha

docker run -rm --name word wordpress
docker exer -it word /bin/sh
docker rm -f word
docker run --name word --entrypoint echo wordpress "Hello World"
docker rm -f word
docker run --name word --env MY_VAR=opus_software -it wordpress /bin/sh;\
echo $MY_VAR;\
docker run --name word --restart always wordpress

### logs e inspect

docker run --name word wordpress;\
docker logs word;\
docker rm -f word;\
docker run --name word --label AUTHOR=bruno word wordpress;\
docker inspect word;\
docker inspect word | grep AUTHOR

## Volumes e Armazenamento

### Bind Mounts

docker run --mount type=bind,src=$PWD/volumes/bind,dst=/home -it ;\
mysql /bin/sh;\
cd /home;\
ls;\
\# cria outro terminal \
cd /volumes/bind;\
touch opus.txt;\
ls;\
touch opus.txt;

### Tmpfs

docker run --mount type=tmpfs,dst=/home --name my -it mysql /bin/sh;\
cd home ;\
touch opus.txt;\
exit;\
docker restart my;\
cd home;\
ls;\

### Volumes

docker volume create my-volume;\
docker volume ls;\
docker run --mount type=volume,src=my-volume,dst=/home --name mysql1 -it mysql /bin/sh;\
cd /home;\
\#Abre outro terminal
docker run --mount type=volume,src=my-volume,dst=/home --name mysql2 -it mysql /bin/sh;\
cd /home;\
touch opus.txt;\
ls;\ \# no outro terminal

### Herança de Volumes

docker volume create my-volume2;\
docker volume create my-volume3;\
docker create --mount type=volume,src=my-volume1,dst=/home --mount type=volume,src=my-volume2,dst=/tmp --name my-container1 mysql;\
docker create --mount type=volume,src=my-volume3,dst=/opt --name my-container2 mysql;\
docker create --volumes-from my-container1 --volumes-from my-container2 --name my-container3 mysql;\
docker inspect my-container3 | less;\

### Volumes anonônimos

docker create --mount type=volume,dst=/home --name word wordpress;\
docker volume ls;\
docker inspect word | less;

## Redes Docker

### Redes Host e Nobody

docker network ls
docker run --name nettools --network network1 -d --rm praqma/network-multitool;\
docker exec -it nettools /bin/sh;\
ifconfig
exit

docker run --name nettools --network host -d --rm praqma/network-multitool;\
docker exec -it nettools /bin/sh;\
ifconfig
exit

\#Abre outro terminal\
ifconfig

### Redes Bridge

docker network create \
--driver bridge \
--attachable \
network1;\
docker run --name nettools1 -d --rm praqma/network-multitool;\
docker run --name nettools2 -d --rm praqma/network-multitool;\
docker network connect network1 nettools1;\
docker network connect network1 nettools2;\
docker exec -it nettools1 /bin/sh;\
ifconfig;\
\# Abre outro terminal\
docker exec -it nettools2 /bin/sh;\
\# Abre outro terminal\
sudo wireshark\
\# Configura para interface any, tcp.port = 12000\
\# Volta pro primeiro terminal\
ifconfig;\
nc -vlkp 12000
\# Volta pro segundo terminal\
ifconfig;\
nc -v nettools1 12000

### Encaminhamento de Porta

docker run --name nettools -p 9000:9000/tcp -d --rm praqma/network-multitool;\
docker exec -it nettools /bin/sh;\
\# Abre outro terminal
sudo wireshark;\
\# escolher any e tcp.port = 12000
\# Volta pro primeiro terminal
nc -vlkp 9000
\# Volta pro segundo terminal
nc -v localhost 9000

## Controle de Recursos

### Memória

docker run -it --name forkbomb python /bin/sh;\
\# Segundo terminal \
docker cp fork-bomb.py forkbomb;\
docker rm -f forkbomb; \# apenas preparar \
\# Terceiro terminal \
docker stats forkbomb \
\# Primeiro terminal \
ls \
python fork-bomb.py \
\# Segundo terminal
docker rm -f forkbomb; \# executasr \

### CPU

docker run --name cpu1 --cpu-shares 1024 -d progrium/stress --cpu 12;\
docker run --name cpu2 --cpu-shares 512 -d progrium/stress --cpu 12;\
docker stats cpu1;\
\# Segundo terminal\
docker stats cpu2;\
docker rm -f cpu1 cpu2;\

### Driver

\# Desligar a câmera da apresentação\
docker run --name=webcam -d -p 8080:8080 -p 8082:8082 --device /dev/video0:/dev/video0 romankspb/webcam ;\
\# Ligar câmera de novo

### Memória compartilhada

\# Mostrar códido em c \
vim ipc.c
\# Rodar sem ipc no segundo container

docker container run -d -u nobody --name producer \
--ipc shareable \
dockerinaction/ch6_ipc -producer;\
docker container run -d -u nobody --name consumer \
dockerinaction/ch6_ipc -consumer;\

\# mostrar os logs
docker logs producer;\
docker logs consumer;\

\# Rodar com o ipc no segundo container
docker container rm -v consumer;\
docker container run -d --name consumer \
--ipc container:producer \
dockerinaction/ch6_ipc -consumer;\

### Operações do Kernel

docker run --name word -it wordpressl /bin/sh;\
touch a.txt;\
chown nobody a.txt;\
docker run --name word -it --cap-drop chown wordpress /bin/sh;\
touch a.txt
chown nobody a.txt;\

## Criação de Imagem

### Sistema de camadas\Tags

docker run --name word -it wordpress /bin/sh;\
cd /etc;\
rm magic.mime;\
echo hosts;\
touch a.txt;\
exit;\
docker diff word;\
docker container commit -m "Make my container" word my-word;\
docker image ls;\
docker tag my-word my-word:1.0;\
docker image ls;\

### Dockerfile básico

\# mostrar Dockerfile_basico.df\

docker image build --file Dockerfile_basico.df -t dockerfile_basico .;\
docker run dockerfile_basico;\
docker run dockerfile_basico"Command não executado";\

### Dockerfile Downstream

\# mostrar Dockerfile_downstream.df\

docker image build --file Dockerfile_downstream.df -t dockerfile_downstream .;\
docker run dockerfile_downstream;\

### Dockerfile Multi Image

\# mostrar Dockerfile_mult_img\
docker image build --file Dockerfile_mult_img.df -t dockerfile_tgt-1 --target first_target .;\
docker run dockerfile_tgt-1;\
docker image build --file Dockerfile_mult_img.df -t dockerfile_tgt-2 --target second_target .;\
docker run dockerfile_tgt-2;

## Docker Compose

### Docker compose básico

docker compose up;\