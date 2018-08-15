# kubernetes-docker-in-docker-cluster [![CircleCI](https://circleci.com/gh/richardsonlima/kubernetes-docker-in-docker-cluster/tree/master.svg?style=svg)](https://circleci.com/gh/richardsonlima/kubernetes-docker-in-docker-cluster/tree/master) [![Travis CI](https://travis-ci.org/richardsonlima/kubernetes-docker-in-docker-cluster.svg?branch=master)](https://travis-ci.org/richardsonlima/kubernetes-docker-in-docker-cluster)
A Kubernetes multi-node cluster for developer of Kubernetes or extend Kubernetes. Based on kubeadm and Docker in
Docker.

## Requirements
Docker 1.12+ is recommended. If you're not using one of the
preconfigured scripts (see below) and not building from source, it's
better to have `kubectl` executable in your path matching the
version of k8s binaries you're using (i.e. for example better don't
use `kubectl` 1.9.x with `hyperkube` 1.8.x).

`kubernetes-docker-in-docker-cluster` supports k8s versions 1.8.x, 1.9.x .

**Run `kubernetes-docker-in-docker-cluster` on Docker with `btrfs`
storage driver is not supported.**

By default `kubernetes-docker-in-docker-cluster` uses dockerized builds, so no Go
installation is necessary even if you're building Kubernetes from
source. If you want you can overridde this behavior by setting
`KUBEADM_DIND_LOCAL` to a non-empty value in [config.sh](config.sh).

### Mac OS X considerations

Ensure to have `md5sha1sum` installed. If not existing can be installed via `brew install md5sha1sum`.

## Using preconfigured scripts
`kubernetes-cluster-over-docker` currently provides preconfigured scripts for
Kubernetes 1.8, 1.9. This may be convenient for use with
projects that extend or use Kubernetes. For example, you can start
Kubernetes 1.9 like this:

```shell

# Downloading kubectl (1.9.9 - for linux) 
$ curl -O https://storage.googleapis.com/kubernetes-release/release/v1.9.9/bin/linux/amd64/kubectl
$ mv kubectl /usr/local/bin/

$ wget https://raw.githubusercontent.com/richardsonlima/kubernetes-docker-in-docker-cluster/master/kubernetes-docker-in-docker-cluster-v1.9.sh
$ chmod +x kubernetes-docker-in-docker-cluster-v1.9.sh

$ # start the cluster
$ ./kubernetes-docker-in-docker-cluster-v1.9.sh up

$ # also you can start the cluster with 3 nodes
$ NUM_NODES=3 ./kubernetes-docker-in-docker-cluster-v1.9.sh up

$ ./kubernetes-docker-in-docker-cluster-v1.9.sh initial-config 

$ kubectl --kubeconfig ~/.kube/config get pods --all-namespaces

# Creating a namespace
$ kubectl --kubeconfig ~/.kube/config create namespace test-docker-in-docker

$ kubectl --kubeconfig ~/.kube/config get nodes
NAME          STATUS    ROLES     AGE       VERSION
kube-master   Ready     master    57m       v1.9.9
kube-node-1   Ready     <none>    56m       v1.9.9
kube-node-2   Ready     <none>    56m       v1.9.9
kube-node-3   Ready     <none>    56m       v1.9.9

$ # k8s dashboard available at http://localhost:8080/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy

$ # restart the cluster, this should happen much quicker than initial startup
$ ./kubernetes-docker-in-docker-cluster-v1.9.sh up

$ # stop the cluster
$ ./kubernetes-docker-in-docker-cluster-v1.9.sh down

$ # remove containers and volumes
$ ./kubernetes-docker-in-docker-cluster-v1.9.sh clean
```

Terminal session test::
[![asciicast](https://asciinema.org/a/195341.png)](https://asciinema.org/a/195341?autoplay=1) 

Replace 1.8 with 1.9 or 1.10 to use other Kubernetes versions.
**Important note:** you need to do `./kubernetes-docker-in-docker-cluster-....sh clean` when
you switch between Kubernetes versions (but no need to do this between
rebuilds if you use `BUILD_HYPERKUBE=y` like described below).

### Using local cluster with JenkinsX 

 
```shell 
# Get jx (https://jenkins-x.io/)
curl -L https://github.com/jenkins-x/jx/releases/download/v1.3.168/jx-darwin-amd64.tar.gz | tar xzv 
sudo mv jx /usr/local/bin```

# Install JenkinsX on DIND K8S Cluster
jx install --provider=kubernetes --on-premise
```

