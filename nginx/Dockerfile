#START OF NGINX SERVER
FROM nginx:mainline-alpine-slim

RUN apk update
RUN apk upgrade

RUN apk add --update nano openssl git curl wget sudo

#INSTALL AND GENERATE OPENSSL CERTIFICATE (HTTPS)
USER root

#INSTALL OPENSSL ECDH prime256v1 (256 BITS) CERTIFICATE
RUN openssl ecparam -genkey -name prime256v1 -out prime256v1.key && \
    openssl req -x509 -new -sha256 -nodes -key prime256v1.key -days 3650 -out prime256v1.crt -subj "/CN=localhost, localhost.com, system.localhost, api.localhost, app.localhost" && \
    cat prime256v1.crt prime256v1.key > prime256v1.pem && \
    mkdir /etc/nginx/ssl && \
    cp prime256v1.key /etc/nginx/ssl && \
    cp prime256v1.crt /etc/nginx/ssl && \
    cp prime256v1.pem /etc/nginx/ssl

##INSTALL OPENSSL RSA-256 SSL CERTIFICATE
#RUN openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout rsa256.key -out rsa256.crt -subj "/CN=localhost.com" -addext "subjectAltName=DNS:localhost.com,DNS:localhost,IP:0.0.0.0,IP:127.0.0.1" && \
#    cat rsa256.crt rsa256.key > rsa256.pem && \
#    cp rsa256.key /etc/nginx/ssl && \
#    cp rsa256.crt /etc/nginx/ssl && \
#    cp rsa256.pem /etc/nginx/ssl

#INSTALL OPENSSL ECDH secp384r1 (384 BITS) CERTIFICATE
RUN rm -rf /var/cache/apk/* && \
    openssl ecparam -genkey -name secp384r1 -out secp384r1.key && \
    openssl req -x509 -new -sha384 -nodes -key secp384r1.key -days 3650 -out secp384r1.crt -subj "/CN=localhost.com" && \
    cat secp384r1.crt secp384r1.key > secp384r1.pem && \
    cp secp384r1.key /etc/nginx/ssl && \
    cp secp384r1.crt /etc/nginx/ssl && \
    cp secp384r1.pem /etc/nginx/ssl

#MAKE NGINX MICRO CACHE DIRECTORIES
RUN mkdir /var/nginx && \
    mkdir /var/nginx/proxy_micro_cache && \
    mkdir /var/nginx/fastcgi_micro_cache

#INSTALL & IMPLEMENT MOD SECURITY WAF
ENV MODSECURITY_INC=/opt/modsecurity/headers/
ENV MODSECURITY_LIB=/opt/modsecurity/src/.libs/

RUN apk add --no-cache --virtual .build-modsec \
  automake \
  autoconf \
  build-base \
  libtool \
  libxml2-dev \
  linux-headers \
  git \
  pcre-dev \
  openssl

WORKDIR /opt
RUN git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity modsecurity
WORKDIR /opt/modsecurity
RUN git submodule init \
  && git submodule update \
  && ./build.sh \
  && ./configure \
  && make \
  && make install

WORKDIR /opt
RUN git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git modsecurity-nginx
RUN wget http://nginx.org/download/nginx-1.23.3.tar.gz
RUN tar zxvf nginx-1.23.3.tar.gz
WORKDIR nginx-1.23.3
RUN ./configure --with-compat --add-dynamic-module=../modsecurity-nginx
RUN make modules
RUN cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules

WORKDIR /
RUN mkdir /etc/nginx/modsec
RUN wget -P /etc/nginx/modsec/ https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended
RUN mv /etc/nginx/modsec/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf

WORKDIR /opt
RUN cp modsecurity/unicode.mapping /etc/nginx/modsec
RUN wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.0.2.tar.gz
RUN tar -xzvf v3.0.2.tar.gz
RUN mv owasp-modsecurity-crs-3.0.2 /usr/local
WORKDIR /usr/local/owasp-modsecurity-crs-3.0.2
RUN cp crs-setup.conf.example crs-setup.conf

#TODO:fix all dependencies are uncomment out all rules
RUN echo -e "Include /etc/nginx/modsec/modsecurity.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/crs-setup.conf \n \
#    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-901-INITIALIZATION.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-905-COMMON-EXCEPTIONS.conf \n \
#    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-910-IP-REPUTATION.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-911-METHOD-ENFORCEMENT.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-912-DOS-PROTECTION.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-913-SCANNER-DETECTION.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-921-PROTOCOL-ATTACK.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-931-APPLICATION-ATTACK-RFI.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-933-APPLICATION-ATTACK-PHP.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/REQUEST-949-BLOCKING-EVALUATION.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-950-DATA-LEAKAGES.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-951-DATA-LEAKAGES-SQL.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-952-DATA-LEAKAGES-JAVA.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-953-DATA-LEAKAGES-PHP.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-954-DATA-LEAKAGES-IIS.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-959-BLOCKING-EVALUATION.conf \n \
    Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-980-CORRELATION.conf \n \
#   Include /usr/local/owasp-modsecurity-crs-3.0.2/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf \
    " > /etc/nginx/modsec/main.conf

RUN sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsec/modsecurity.conf

#TODO:: ADD A BASIC EXTENAL FIREWALL

EXPOSE 80 443 444 445

CMD ["nginx", "-g", "daemon off;"]

#TODO:CONVERT THIS TO RUN ON HTTP3 QUIC
