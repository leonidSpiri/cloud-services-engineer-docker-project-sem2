ARG GO_VERSION=1.22
ARG ALPINE_VERSION=3.21

FROM --platform=$BUILDPLATFORM golang:${GO_VERSION}-alpine AS build

WORKDIR /src

COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download

COPY . .

ARG TARGETOS=linux
ARG TARGETARCH=amd64
ENV CGO_ENABLED=0

RUN --mount=type=cache,target=/root/.cache/go-build \
    GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build -trimpath -ldflags="-s -w" \
    -o /out/momo-store ./cmd/api

FROM alpine:${ALPINE_VERSION}

ARG APP_VERSION=local

LABEL org.opencontainers.image.title="momo-store-backend" \
      org.opencontainers.image.description="Go API for Momo Store" \
      org.opencontainers.image.version="${APP_VERSION}"

RUN addgroup -S app && adduser -S -D -H -u 10001 -G app app \
    && apk add --no-cache ca-certificates tzdata \
    && mkdir -p /var/lib/momo-store \
    && chown -R app:app /var/lib/momo-store

WORKDIR /app

COPY --from=build --chown=app:app /out/momo-store /app/momo-store

ENV PORT=8081

EXPOSE 8081

USER app

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD wget -qO- "http://127.0.0.1:${PORT}/health" >/dev/null || exit 1

ENTRYPOINT ["/app/momo-store"]
