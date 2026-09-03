---
{"dg-publish":true,"permalink":"/src/site/notes/notes/agentic-ai/public-notes/day-11-1/","dg-note-properties":{}}
---



Fedora Linux
│
├── /usr/bin/docker              ← Docker CLI
├── /usr/bin/dockerd             ← Docker daemon
├── /usr/lib/systemd/system/     ← Docker service
│
└── /var/lib/docker/             ← Docker ka data
    ├── images
    ├── containers
    ├── volumes
    └── ...

Docker ko control kaise karna hai?

Docker ko mainly 3 cheezon se samjho:

Docker
├── Images       → templates
├── Containers   → running/stopped instances
└── Volumes      → persistent data


Laravel ke time tum in teenon ka bahut use karoge.

1. Docker service control
Docker start
sudo systemctl start docker

Docker stop
sudo systemctl stop docker

Docker restart
sudo systemctl restart docker

Docker status
sudo systemctl status docker

Fedora boot par automatically start

Tum already ye kar chuke ho:

sudo systemctl enable --now docker


Check:

systemctl is-enabled docker


Aana chahiye:

enabled

2. Docker version
docker --version


Example:

Docker version 29.7.2


Compose:

docker compose version


Notice karo:

docker compose


not

docker-compose


Modern Docker mein Compose plugin isi tarah use hota hai.

3. Docker chal raha hai ya nahi?
docker info


Ye Docker daemon ki detailed information dega.

Simple test:

docker run hello-world


Tum ye already successfully chala chuke ho. ✅

4. Docker Images

Images dekhne ke liye:

docker images


Ya:

docker image ls


Tumhare case mein abhi hello-world image dikhegi.

Image download karna:

docker pull ubuntu


Laravel ke time example:

docker pull php:8.4


Image delete:

docker rmi IMAGE_NAME


Example:

docker rmi hello-world

5. Containers

Sabse important commands.

Running containers
docker ps

Saare containers
docker ps -a


-a ka matlab stopped containers bhi.

Container run
docker run hello-world


Interactive Ubuntu:

docker run -it ubuntu bash


Ab tum container ke andar chale jaoge:

root@xxxxxxxx:/#


Bahar aane ke liye:

exit

6. Container ko name dena

Ye useful hai.

docker run --name myubuntu -it ubuntu bash


Ab container ka naam:

myubuntu


Check:

docker ps -a

7. Container start/stop

Stopped container:

docker start myubuntu


Stop:

docker stop myubuntu


Restart:

docker restart myubuntu


Container delete:

docker rm myubuntu


Running container ko force delete:

docker rm -f myubuntu

8. Container ke andar command chalana

Suppose PostgreSQL container hai:

docker exec -it postgres bash


Ya directly command:

docker exec -it postgres psql


Running containers dekhne ke liye:

docker ps

9. Logs

Ye development mein bahut important hai.

docker logs container-name


Live logs:

docker logs -f container-name


Example:

docker logs -f postgres


Ctrl+C se logs se bahar aa jaoge; container band nahi hota.

Last 100 lines:

docker logs --tail 100 postgres

10. Docker Compose

Laravel project ke saath tumhara sabse zyada use likely ye hoga:

docker compose up


Background mein:

docker compose up -d


Stop:

docker compose down


Running services:

docker compose ps


Logs:

docker compose logs


Live logs:

docker compose logs -f


Sirf PostgreSQL:

docker compose logs -f postgres


Sirf Redis:

docker compose logs -f redis


Rebuild:

docker compose build


Rebuild + start:

docker compose up -d --build

11. Docker Volumes

Volumes PostgreSQL jaise data ke liye important hain.

List:

docker volume ls


Details:

docker volume inspect VOLUME_NAME


Delete:

docker volume rm VOLUME_NAME


Warning: PostgreSQL ka volume delete karoge to database data ja sakta hai. Isliye docker volume prune jaise commands blindly mat chalana.

12. Docker networks
docker network ls


Details:

docker network inspect NETWORK_NAME


Compose automatically network bana deta hai, isliye Laravel + PostgreSQL + Redis setup mein manually network banana usually necessary nahi hoga.

13. Cleanup commands

Disk space check:

docker system df


Unused containers/images/networks clean:

docker system prune


⚠️ Isko blindly mat chalana.

Aur ye aur dangerous ho sakta hai:

docker system prune -a


Ye unused images bhi remove kar sakta hai.

Volumes ke saath:

docker system prune -a --volumes


Isko abhi bilkul mat chalana.

Tumhare liye sabse important commands

Agar tum Laravel + Docker seekh rahe ho, pehle inko yaad karo:

# Docker status
sudo systemctl status docker

# Images
docker images

# Running containers
docker ps

# All containers
docker ps -a

# Start
docker start NAME

# Stop
docker stop NAME

# Restart
docker restart NAME

# Logs
docker logs -f NAME

# Container ke andar
docker exec -it NAME bash

# Compose start
docker compose up -d

# Compose stop
docker compose down

# Compose status
docker compose ps

# Compose logs
docker compose logs -f

# Docker disk usage
docker system df

Aur ek kaam abhi kar lo

Tumne sudo docker run hello-world chalaya hai. Ab hum sudo hatana chahenge:

sudo usermod -aG docker $USER


Uske baad logout → login karo (ya system reboot).

Phir:

docker run hello-world


Agar sudo ke bina chal gaya, tumhara Docker user setup bhi ready hai.

Uske baad next step mein hum tumhari NTFS drive ke development folder mein ek proper Laravel + PHP + Nginx + PostgreSQL + Redis Docker project bana sakte hain.