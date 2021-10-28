# squidinkube

## Build

`docker build . -f ContainerFile -t squidinkube:latest`

## Deploy

oc create configmap squidconfig --from-file=example-squid.conf 