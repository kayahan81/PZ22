FROM golang:1.25-alpine AS builder

WORKDIR /app

# Копируем go.mod из корня проекта
COPY go.mod go.sum ./
RUN go mod download

# Копируем весь код
COPY . .

# Собираем tasks сервис
RUN go build -o tasks ./services/tasks/cmd/tasks

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/tasks .

EXPOSE 8082
CMD ["./tasks"]