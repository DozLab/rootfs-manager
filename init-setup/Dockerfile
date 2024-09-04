FROM alpine

WORKDIR /app

RUN apk add --no-cache \
    e2fsprogs-extra

COPY ./disk/image.ext4 /disk/image.ext4 

COPY init.sh init.sh

CMD ["/app/init.sh"]
