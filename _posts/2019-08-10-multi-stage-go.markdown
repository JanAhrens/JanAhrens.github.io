---
title: Small Docker images with embedded go binaries
date: 2019-08-10
---

I recently wanted to include two go binaries in a Docker image and tried to make the resulting image as small as possible. My goal was to speed up its execution and download.

The solution is actually pretty simple:

```dockerfile
FROM golang:1.12.7-alpine3.10 AS builder
RUN apk --no-cache add git
RUN go get github.com/wakeful/yaml2json
RUN go get github.com/santhosh-tekuri/jsonschema/cmd/jv

FROM alpine:3.10
WORKDIR /root/
COPY --from=builder /go/bin/yaml2json /usr/local/bin
COPY --from=builder /go/bin/jv /usr/local/bin
```

The trick is to use Dockers [multi-stage build]() feature. While Docker is building the image, it actually creates an intermediate image called "builder".
