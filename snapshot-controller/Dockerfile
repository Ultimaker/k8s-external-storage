# Build external-storage binaries
FROM golang:1.10-alpine

ENV EXTERNAL_STORAGE_VERSION "master"
ENV GITHUB_PATH "github.com/kubernetes-incubator/external-storage"

RUN apk add --no-cache git

RUN mkdir -p ${GOPATH}/src/github.com/kubernetes-incubator
WORKDIR ${GOPATH}/src/github.com/kubernetes-incubator
RUN git clone https://${GITHUB_PATH}
WORKDIR ${GOPATH}/src/${GITHUB_PATH}
RUN git checkout ${EXTERAL_STORAGE_VERSION}

WORKDIR /go/src/${GITHUB_PATH}/snapshot

RUN go build -o _output/bin/snapshot-controller-linux-amd64 cmd/snapshot-controller/snapshot-controller.go
RUN cp /go/src/${GITHUB_PATH}/snapshot/_output/bin/snapshot-controller-linux-amd64 /usr/local/bin/snapshot-controller

ENTRYPOINT ["/usr/local/bin/snapshot-controller"]
