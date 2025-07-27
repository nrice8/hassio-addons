ARG BUILD_FROM=ghcr.io/hassio-addons/base-nodejs:14.18.1
FROM ${BUILD_FROM}

# Add labels
LABEL \
  org.opencontainers.image.title="Nginx Proxy Manager" \
  org.opencontainers.image.description="Nginx Proxy Manager for Home Assistant" \
  org.opencontainers.image.source="https://github.com/NginxProxyManager/nginx-proxy-manager" \
  org.opencontainers.image.licenses="MIT"

# Set environment variables
ENV \
  LANG="C.UTF-8" \
  NGINX_PROXY_MANAGER_VERSION="v2.10.4"

# Install dependencies and build NPM
RUN \
  apk add --no-cache --virtual .build-dependencies \
    build-base \
    git \
    libffi-dev \
    openssl-dev \
    patch \
    python3-dev \
  && apk add --no-cache \
    apache2-utils \
    certbot \
    libcap \
    logrotate \
    nginx-mod-stream \
    nginx \
    openssl \
    py3-pip \
    python3 \
  && pip3 install -r /tmp/requirements.txt \
  && yarn global add modclean \
  && curl -L -o /tmp/nginxproxymanager.tar.gz "https://github.com/NginxProxyManager/nginx-proxy-manager/archive/${NGINX_PROXY_MANAGER_VERSION}.tar.gz" \
  && mkdir /app \
  && tar zxvf /tmp/nginxproxymanager.tar.gz --strip 1 -C /app \
  && cd /app/frontend \
  && yarn install \
  && yarn build \
  && rm -rf node_modules \
  && mkdir -p /opt/nginx-proxy-manager/frontend \
  && cp -r /app/frontend/dist/. /opt/nginx-proxy-manager/frontend/ \
  && cd /app/backend \
  && yarn install \
  && rm -rf node_modules \
  && cp -r /app/backend/. /opt/nginx-proxy-manager/ \
  && cp -R /app/global/. /opt/nginx-proxy-manager/global/ \
  && cd /opt/nginx-proxy-manager \
  && yarn install \
  && rm -rf /etc/services.d/frontend /opt/nginx-proxy-manager/config \
  && rm -rf /etc/nginx \
  && cp -r /app/docker/rootfs/etc/nginx /etc/nginx \
  && rm -f /etc/nginx/conf.d/dev.conf \
  && cp /app/docker/rootfs/etc/letsencrypt.ini /etc \
  && cp /app/docker/rootfs/etc/logrotate.d/nginx-proxy-manager /etc/logrotate.d \
  && sed -i "s#user npm;#user root;#" /etc/nginx/nginx.conf \
  && sed -i "s#root /app/
