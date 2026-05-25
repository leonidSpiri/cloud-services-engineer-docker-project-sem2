ARG NODE_VERSION=20
ARG NGINX_VERSION=1.27-alpine

FROM node:${NODE_VERSION}-alpine AS build

WORKDIR /app

ENV NODE_OPTIONS=--openssl-legacy-provider

COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --no-audit --no-fund

ARG VUE_APP_API_URL=http://localhost:8081
ARG VUE_PUBLIC_PATH=/
ENV VUE_APP_API_URL=${VUE_APP_API_URL}
ENV VUE_PUBLIC_PATH=${VUE_PUBLIC_PATH}

COPY . .
RUN npm run build

FROM nginxinc/nginx-unprivileged:${NGINX_VERSION}

ARG APP_VERSION=local

LABEL org.opencontainers.image.title="momo-store-frontend" \
      org.opencontainers.image.description="Vue frontend for Momo Store" \
      org.opencontainers.image.version="${APP_VERSION}"

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build --chown=101:101 /app/dist /usr/share/nginx/html

EXPOSE 8080

USER 101

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD wget -qO- http://127.0.0.1:8080/healthz >/dev/null || exit 1
