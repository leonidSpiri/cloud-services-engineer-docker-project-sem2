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

WORKDIR /app

COPY --from=build /out/momo-store /app/momo-store

EXPOSE 8081

ENTRYPOINT ["/app/momo-store"]
