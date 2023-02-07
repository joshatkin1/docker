FROM nginx:mainline-alpine-slim

RUN apk update
RUN apk upgrade

RUN apk add nano

RUN apk add --update nodejs npm
RUN apk add --update npm

#Linux setup
RUN apk update \
  && apk add --update alpine-sdk \
  && apk del alpine-sdk \
  && rm -rf /tmp/* /var/cache/apk/* *.tar.gz ~/.npm \
  && npm cache verify

#INSTALL AND GENERATE OPENSSL CERTIFICATE (HTTPS)
USER root

RUN apk add --update openssl && \
    rm -rf /var/cache/apk/* && \
    openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout localhost.key -out localhost.crt -subj "/CN=localhost.com" -addext "subjectAltName=DNS:localhost.com,DNS:localhost,IP:0.0.0.0,IP:127.0.0.1" && \
    cat localhost.crt localhost.key > localhost.pem

RUN mkdir /etc/nginx/ssl && \
    cp localhost.crt /etc/nginx/ssl && \
    cp localhost.key /etc/nginx/ssl && \
    cp localhost.pem /etc/nginx/ssl

#Angular CLI
RUN npm install -g @angular/cli

CMD ["nginx", "-g", "daemon off;"]

EXPOSE 4201