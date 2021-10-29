# squidinkube

## Intro

This repo is for creating a squid proxy inside a kubernetes cluster. Why would you ever want to do this? If you have an application that does not honor the "NO_PROXY" settings, this squid configuration will act as an intermediary sending requests to a "upstream" proxy to get outside the Kubernetes cluster and will direct connect to all services that are inside the kubernetes platform. This will allow your application that does not know how to use the "NO_PROXY" settings to connect to both internal cluster resources as well as external resources.

## Build

To build this container image use the following command:

`docker build . -f ContainerFile -t squidinkube:latest`

There is also a published container image here: quay.io/markd/squidinkube:v3

## Configuration

In order to use this pod, you will need to update the `k8s/squid.conf` file located in this repo. Start by making a copy of the squid.conf.example file in the top of this repo:

`cp k8s/squid.conf.example squid.conf`

Now, open the `squid.conf` file in your favorite editor. We will need to update mulitple lines in the configuration.

Start by adding YOUR upstream proxy, replacing "<upstream proxy goes here>" and "<upstream proxy port goes here>" with your appropriate information. This should be host name or IP address only. DO NOT pu "http://" are part of the proxy configuration.

Next review the "acl local-servers dstdomain .svc.cluster.local" entry. This entry will catch host names that are generated for within your kubernetes cluster. If you require additional domain names that should also bypass the proxy, add additional "acl local-servers dstdomain .svc.cluster.local" lines to the configuration with the additional host names. For example, if you should also bypass the upstream proxy for test.example.com, you would add a new line that says "acl local-servers dstdomain .example.com".

Finally review the "acl local-network" entries. The two entries found here should cover a default kubernetes install for bypassing local internal networks. Review and update as required. In addition if you need to add additional internal only networks by IP CIDR, add a new line, and be sure to add a new "always_direct" line that uses the new ACL you created and add the new ACL to the "never_direct" directive as well.

## Deploy

We can now deploy the internal proxy server. The following commands will use the "oc" (OpenShift command line utility), but can be modified to use kubectl should you want. 

1. Start by logging into your cluster `oc login`
2. Switch to the project/namespace you wish to run the squidproxy `oc project pam`
3. create a configmap using the squid.conf file we created above `oc create configmap squidconfig --from-file=squid.conf`
4. create the squid deployment `oc create -f k8s/squid-deployment.yaml`
5. create the service for our squid proxy `oc create -f k8s/squid-service.yaml`

You can now configure your application to point to the following proxy settings:

```
http_proxy=http://squidproxy:3128
https_proxy=http://squidproxy:3128
```
