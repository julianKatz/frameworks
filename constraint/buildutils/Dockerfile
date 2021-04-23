FROM golang:1.16.3-alpine

RUN apk add build-base

# Create a folder in which to mount the code
RUN mkdir /constraint
WORKDIR /constraint

RUN GO111MODULE=on go get sigs.k8s.io/controller-tools/cmd/controller-gen@v0.5.0
