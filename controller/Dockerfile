# Precompile key slow-to-build dependencies
FROM golang:1.14.2-alpine as go-deps
WORKDIR /linkerd-build
COPY go.mod go.sum ./
COPY bin/install-deps bin/
RUN go mod download
RUN ./bin/install-deps

## compile controller service
FROM go-deps as golang
WORKDIR /linkerd-build
COPY controller/gen controller/gen
COPY pkg pkg
COPY controller controller
COPY charts/patch charts/patch
COPY charts/partials charts/partials

# Generate static templates
# TODO: `go generate` does not honor -mod=readonly
RUN go generate -mod=readonly ./pkg/charts/static

RUN CGO_ENABLED=0 GOOS=linux go build -o /out/controller -tags prod -mod=readonly -ldflags "-s -w" ./controller/cmd

## package runtime
FROM scratch
ENV PATH=$PATH:/go/bin
COPY LICENSE /linkerd/LICENSE
COPY --from=golang /out/controller /go/bin/controller
# for heartbeat (https://versioncheck.linkerd.io/version.json)
COPY --from=golang /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

ARG LINKERD_VERSION
ENV LINKERD_CONTAINER_VERSION_OVERRIDE=${LINKERD_VERSION}

ENTRYPOINT ["/go/bin/controller"]
