```bash
# rabbitmq
docker run -d --hostname rabbitmq --name rabbitmq -e RABBITMQ_DEFAULT_USER=portal -e RABBITMQ_DEFAULT_PASS=xuanyuan -p 15672:15672 -p 5672:5672  rabbitmq:management

# redis
docker run --name redis -d -p 6379:16379 redis

# tachidesk
docker run --name tachidesk -d -p 4567:4567 -v /mervyn/docker/volumes/tachidesk/:/home/suwayomi/.local/share/Tachidesk ghcr.io/suwayomi/tachidesk 

#wiznote 为知笔记
docker run --name wiz --restart=always -it -d -v /mervyn/docker/volumes/wizdata:/wiz/storage -v /etc/localtime:/etc/localtime -p 80:80  wiznote/wizserver
```

