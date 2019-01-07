FROM golang:1.11.4-alpine3.8 as builder

COPY . .

RUN CGO_ENABLED=0 go build -a -o /app 

FROM alpine:3.8

EXPOSE 8080
ENTRYPOINT ["/app"]
COPY --from=builder /app /
