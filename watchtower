sudo vi docker-compose.yaml

watchtower:
  container_name: watchtower
  restart: always
  image: containrrr/watchtower
  environment:
    - WATCHTOWER_NOTIFICATIONS=email
    - WATCHTOWER_NOTIFICATION_EMAIL_FROM=me@server.com
    - WATCHTOWER_NOTIFICATION_EMAIL_TO=them@server.com
    - WATCHTOWER_NOTIFICATION_EMAIL_SERVER=email.server
    - WATCHTOWER_NOTIFICATION_EMAIL_SERVER_USER=user
    - WATCHTOWER_NOTIFICATION_EMAIL_SERVER_PASSWORD=
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  command: --debug true --cleanup true portainer adb tasmoadmin

Once you have all that in there, save the file and 

sudo docker-compose up -d
