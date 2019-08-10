---
title: Small Docker images with embedded go binaries
date: 2019-08-10
---

I recently wanted to include [Go](https://golang.org) binaries in a Docker image and tried to make the image as small as possible. My goal was to speed up its execution and download time.

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

The trick is to use Dockers' [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/) feature. While Docker is building the image, it actually creates another image called "builder" first. This image gets discarded after the binaries got copied over to the final image and thus the final image doesn't need the entire go build system.

One caveat is that the builder image and the final image have to be binary compatible. In my first attempt I used the `golang-1.12.7` image for the builder image. The binaries got build successfully but wouldn't run in the final image. The reason for that was that this image was using [Debian buster](https://www.debian.org/News/2019/20190706) under the hood and not [Alpine Linux](https://alpinelinux.org). Fortunately there's also the `golang-1.12.7-alpine3.10` image available on Dockerhub that is based on the same Alpine version that the final image uses.

If you want to experiment with this, you can find the complete code and some example files for my [JSON schema](https://json-schema.org) with YAML idea in my [multi-stage-go repository](https://github.com/JanAhrens/multi-stage-go).
