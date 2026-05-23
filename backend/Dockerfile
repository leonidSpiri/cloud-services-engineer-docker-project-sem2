FROM golang:1.22-alpine

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build -o momo-store ./cmd/api

EXPOSE 8081

CMD ["./momo-store"]
