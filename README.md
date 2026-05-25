# Momo Store Docker Project

В проекте используется контейнеризация Go backend и Vue frontend через Docker.

## Запуск

Собрать и запустить production-стек:

```bash
docker compose up -d --build
```

Проверить состояние контейнеров:

```bash
docker compose ps
```

Остановить стек:

```bash
docker compose down
```

Доступные порты:

- Frontend: http://localhost:80
- Backend API через gateway: http://localhost:8081
- Healthcheck backend: http://localhost:8081/health
- Healthcheck frontend: http://localhost/healthz

## Образы

Backend и frontend собираются через multi-stage builds:

- `momo-store-backend:local` - Go builder `golang:1.26-alpine`, runtime `alpine:3.21`, non-root пользователь `app`.
- `momo-store-frontend:local` - Node builder `node:20-alpine`, runtime `nginxinc/nginx-unprivileged:1.27-alpine` с обновлёнными Alpine-пакетами, non-root пользователь `101`.
- `momo-store-backend-gateway:local` - nginx gateway для публикации и балансировки backend на внешнем порту `8081`.

В финальные образы не копируются исходники сборочных зависимостей, `node_modules`, Go cache и другие временные файлы. Контексты сборки очищаются через `backend/.dockerignore` и `frontend/.dockerignore`.

## Конфигурация

Основные переменные и build-аргументы:

- `FRONTEND_PORT` - внешний порт frontend, по умолчанию `80`.
- `BACKEND_PORT` - внешний порт backend gateway, по умолчанию `8081`.
- `VUE_APP_API_URL` - URL API, по умолчанию `/api`.
- `VUE_PUBLIC_PATH` - public path Vue-сборки, по умолчанию `/`.
- `GO_VERSION`, `NODE_VERSION`, `ALPINE_VERSION`, `NGINX_VERSION` - версии базовых образов.
- `BACKEND_IMAGE`, `FRONTEND_IMAGE`, `APP_VERSION` - теги и версия образов.


## Масштабирование

Backend не публикует порт напрямую: внешний доступ идёт через `backend-gateway`, поэтому сервис можно масштабировать без конфликта портов:

```bash
docker compose up -d --scale backend=3
```

Вернуть один экземпляр:

```bash
docker compose up -d --scale backend=1
```

## Безопасность

В production-сервисах настроены:

- non-root пользователи внутри контейнеров;
- `read_only: true` для файловой системы контейнеров;
- `tmpfs` для временных директорий;
- `cap_drop: [ALL]`;
- `security_opt: no-new-privileges:true`;
- лимиты `cpus`, `mem_limit`, `pids_limit`;
- изолированная внутренняя сеть `backend_net`;
- открыты только внешние порты `80` и `8081`;
- Docker secret `backend_token`, который по умолчанию берётся из безопасного example-файла без реального секрета.

Для настоящего секрета создайте файл вне репозитория и передайте путь:

```bash
BACKEND_SECRET_SOURCE=/absolute/path/backend_token.txt docker compose up -d
```

## Сканирование

В `.github/workflows/deploy.yaml` добавлен отдельный job `security_scan`, который собирает образы и проверяет их через Trivy по `HIGH` и `CRITICAL` уязвимостям.

Локально можно запустить аналогичную проверку, если установлен Trivy:

```bash
trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 momo-store-backend:local
trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 momo-store-frontend:local
trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 momo-store-backend-gateway:local
```

## Frontend dev profile

В `docker-compose.yml` есть отдельный сервис `frontend-dev` для разработки. Он запускает Vue dev server с hot reload и не используется в production-запуске.

По умолчанию он не стартует:

```bash
docker compose up -d
```

Для запуска dev-режима нужно включить профиль:

```bash
docker compose --profile dev up frontend-dev
```

Основной production frontend запускается сервисом `frontend`: он собирает Vue-приложение и раздает статические файлы через nginx на порту `80`.