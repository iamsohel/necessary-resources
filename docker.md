https://docs.google.com/presentation/d/1XnsV-ls42h0v5AxfrAhLQu9gB5aow6sH1H8d6jYfqyM/edit?usp=sharing

how to install docker 18.04 -- https://www.hostinger.com/tutorials/how-to-install-and-use-docker-on-ubuntu/

sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

$ which docker-compose

/usr/local/bin/docker-compose

$ docker-compose -v

docker-compose version 1.27.4, build 40524192

RUN addgroup app && adduser -S -G app app


https://mherman.org/presentations/dockercon-2018/#5 ---BEST Practice

docker build -t dockersrana/posts .   -> build image

docker run image_id or image_tag

docker run -d -p 6379:6379 redis

docker run -it iamge_id or tag [cmd][sh]

docker ps -> all running containers

dcoker ps -a -> all container including stopped

docker exec -it containerid [cmd][sh]

docker logs container id

docker logs --tail=50 container-id

docker logs -f container id [follow mode]

docker stop container_id

docker start container_id ,[run -> start a new container, start- start a stoped container]

docker run -d -p browserPort:hostPort --name c1 imagename

docker run -p 3000:8080 -d <your username>/node-web-app

docker container rm -f $(docker container ls -aq) -> delete all container

docker image rm -f $(docker image ls -q) -> delete all images

--delele unused image--

docker container prune

docker image prune

--remove image--

docker image rm image_name

--remove container--

docker rm container_id

docker rm -f container_id

docker container prune

# Enter the container

$ docker exec -it <container id> /bin/bash

docker-compose down --rmi all

docker exec -it -u sohel container_id bash

--copy file from source to docker--

docker cp file.txt container_id:/app

--copy file from container to current dir

dockeer cp container_id:/app/data.txt .

```
version: '3.9'

services:
  nginx:
    build: ./nginx
    ports:
      - 4001:80
    volumes:
      - static_volume:/home/app/static
      - media_volume:/home/app/media
    depends_on:
      - api
    restart: "on-failure"
  api:
    build:
      context: ./app
      dockerfile: Dockerfile
    command: >
      bash -c "python manage.py collectstatic --no-input --clear
      && python manage.py migrate
      && gunicorn flow_code_api.wsgi:application --bind 0.0.0.0:8000"
    #command: sh -c "gunicorn flow_code_api.wsgi:application --bind 0.0.0.0:8000"
    volumes:
      - ./app/:/home/app/
      - static_volume:/home/app/static
      - media_volume:/home/app/media
    env_file:
      - ./.env.prod

    expose:
      - 8000
    restart: "on-failure"
    depends_on:
      - db
  db:
    image: postgres:13.0-alpine
    ports:
      - 4002:5432
    volumes:
      - postgres_prod_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db
    restart: "on-failure"

volumes:
  postgres_prod_data:
  static_volume:
  media_volume:


DB BACKUP
=========

 pgbackups:
    container_name: Backup
    image: prodrigestivill/postgres-backup-local
    restart: always
    volumes:
      - ./backup:/backups
    links:
      - db:db
    depends_on:
      - db
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_DB=${DB_NAME} 
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_EXTRA_OPTS=-Z9 --schema=public --blobs
      - SCHEDULE=@every 0h30m00s
      - BACKUP_KEEP_DAYS=7
      - BACKUP_KEEP_WEEKS=4
      - BACKUP_KEEP_MONTHS=6
      - HEALTHCHECK_PORT=81

```

# copy dump into container

docker cp local/path/to/db.dump CONTAINER_ID:/db.dump


# shell into container
docker exec -it CONTAINER_ID bash

# restore it from within
pg_restore -U postgres -d DB_NAME --no-owner -1 /db.dump


Backup your databases:

**take backup:** docker exec -i 34c14229ac05 pg_dump -U debug -h localhost -p 5432 -d db_name -f 30-septerber-2024-backup.sql

**copy from container to host PC:** docker cp 34c14229ac05:30-septerber-2024-backup.sql . 

**restore:** docker exec -i ce2d8b6ddb10  psql -U db_user -d db_name < memorizeitall.sql

docker exec -t your-db-container pg_dumpall -c -U postgres > dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql

docker exec -t d5e23eaf7d24   pg_dump --no-owner -U postgres dbname > db.sql


Restore your databases

cat your_dump.sql | docker exec -i your-db-container psql -U postgres -d dbname

When restoring the database, make sure you add -d your-db-name to the restore command if your database isn't named postgres

docker exec -t  8006a025ac04 pg_dump -U postgres -d flowcode_dev >db_dump.sql 



periodically delete used images/container/netwok
--------------------------------------------------

cd /etc/cron.daily
sudo nano docker-prune

paste
-------

#!/bin/bash
docker system prune -af  --filter "until=$((7*24))h"    -- 7 days

permission
-----
sudo chmod +x /etc/cron.daily/docker-prune

