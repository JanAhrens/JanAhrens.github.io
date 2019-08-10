---
title: Small Docker images with embedded go binaries
date: 2019-08-10
---

I recently wanted to include two go binaries in a Docker image and tried to make the resulting image as small as possible. My goal was to speed up its execution and download time.

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

The trick is to use Dockers [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/) feature. While Docker is building the image, it actually creates another image called "builder" first. This image gets discarded after the binaries got copied over to the final image and thus the final image doesn't need the entire go build system.

One caveat is that the builder image and the final image have to be binary compatible. In my first attempt I used the `golang-1.12.7` image for the builder image. The binaries got build successfully but wouldn't run in the final image. The reason for that was that this image was using Debian buster under the hood and not Alpine Linux. Fortunately there's also the `golang-1.12.7-alpine3.10` image available that is based on the final image.

If you want to experiment with this, you can find the complete code and some example files for my [JSON schema](https://json-schema.org) with YAML idea my [multi-stage-go repository](https://github.com/JanAhrens/multi-stage-go).
