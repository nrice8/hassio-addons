ARG BUILD_FROM=ghcr.io/hassio-addons/base-nodejs:0.1.1
FROM ${BUILD_FROM}

# Required for webpack config of NPM
ENV NODE_OPTIONS="--openssl-legacy-provider"

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Define build arguments
ARG NGINX_PROXY_MANAGER_VERSION="v2.10.4"

# Install dependencies and Nginx Proxy Manager
RUN apk add --no-cache --virtual .build-dependencies \
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
    && pip3 install --no-cache-dir -r /tmp/requirements.txt || true \
    && yarn global add modclean \
    && curl -L -o /tmp/nginxproxymanager.tar.gz \
        "https://github.com/NginxProxyManager/nginx-proxy-manager/archive/${NGINX_PROXY_MANAGER_VERSION}.tar.gz" \
    && mkdir /app \
    && tar zxvf /tmp/nginxproxymanager.tar.gz --strip 1 -C /app \
    && mkdir -p /usr/src || true \
    && for i in /usr/src/*.patch; do patch -d /app -p 1 < "${i}" || true; done \
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
    && rm -rf /etc/services.d/frontend \
    && rm -rf /opt/nginx-proxy-manager/config \
    && rm -rf /etc/nginx \
    && cp -r /app/docker/rootfs/etc/nginx /etc/nginx \
    && rm -f /etc/nginx/conf.d/dev.conf \
    && cp /app/docker/rootfs/etc/letsencrypt.ini /etc \
    && cp /app/docker/rootfs/etc/logrotate.d/nginx-proxy-manager /etc/logrotate.d \
    && sed -i "s#user npm;#user root;#" /etc/nginx/nginx.conf \
    && sed -i "s#root /app/frontend;#root /opt/nginx-proxy-manager/frontend;#" /etc/nginx/conf.d/production.conf \
    && sed -i "s#table.string('id').notNull().primary();#table.string('id', 32).notNull(
