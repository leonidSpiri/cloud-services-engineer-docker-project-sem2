ARG NGINX_VERSION=1.27-alpine

FROM nginxinc/nginx-unprivileged:${NGINX_VERSION}

USER root

RUN apk upgrade --no-cache

COPY backend-gateway.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080

USER 101

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD wget -qO- http://127.0.0.1:8080/health >/dev/null || exit 1
