# Dockerfile for tls-shunt-proxy based alpine
# Copyright (C) 2019 - 2020 namowens
# Reference URL:
# https://github.com/liberal-boy/tls-shunt-proxy

# STEP 1 build executable binary

FROM golang:alpine AS builder
RUN apk update && apk add --no-cache git
WORKDIR /go/src
RUN git clone --progress https://github.com/liberal-boy/tls-shunt-proxy.git && \
    cd tls-shunt-proxy && \
    env CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -v -tags "full" -ldflags "-s -w" -o /tmp/tsp && \
    chmod +x /tmp/tsp && cd .. && rm -rf tls-shunt-proxy

# STEP 2 build a small image

FROM alpine
LABEL maintainer="namowens <namowen@protonmail.com>"

COPY --from=builder /tmp/tsp /tmp
COPY config.yaml /etc/tls-shunt-proxy/config.yaml
RUN apk update && apk add ca-certificates && \
    mkdir -p /usr/bin/tls-shunt-proxy && \
    cp /tmp/tsp /usr/bin/tls-shunt-proxy/tls-shunt-proxy

#ENTRYPOINT ["/usr/bin/tls-shunt-proxy/tls-shunt-proxy"]
ENV PATH /usr/bin/tls-shunt-proxy:$PATH
WORKDIR /etc/ssl/tls-shunt-proxy
CMD ["tls-shunt-proxy", "-config", "/etc/tls-shunt-proxy/config.yaml"]