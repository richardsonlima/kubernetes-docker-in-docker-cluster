# KubeWolf [![CircleCI](https://circleci.com/gh/richardsonlima/kubernetes-docker-in-docker-cluster/tree/master.svg?style=svg)](https://circleci.com/gh/richardsonlima/kubernetes-docker-in-docker-cluster/tree/master) [![Travis CI](https://travis-ci.org/richardsonlima/kubernetes-docker-in-docker-cluster.svg?branch=master)](https://travis-ci.org/richardsonlima/kubernetes-docker-in-docker-cluster)

<img src="/pics/logo.png" width="155px" alt="kubewolf logo">

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

```console

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

 
```console
# Get jx (https://jenkins-x.io/)
curl -L https://github.com/jenkins-x/jx/releases/download/v1.3.168/jx-darwin-amd64.tar.gz | tar xzv 
sudo mv jx /usr/local/bin

# Install JenkinsX on DIND K8S Cluster
jx install --provider=kubernetes --on-premise (or  --local-cloud-environment)
```

## Deploy a Guestbook Example App

This example shows how to build a simple multi-tier web application using Kubernetes and Docker. The application consists of a web front end, Redis master for storage, and replicated set of Redis slaves, all for which we will create Kubernetes replication controllers, pods, and services.

##### Table of Contents

 * [Step Zero: Prerequisites](#step-zero)
 * [Step One: Create the Redis master pod](#step-one)
 * [Step Two: Create the Redis master service](#step-two)
 * [Step Three: Create the Redis slave pods](#step-three)
 * [Step Four: Create the Redis slave service](#step-four)
 * [Step Five: Create the guestbook pods](#step-five)
 * [Step Six: Create the guestbook service](#step-six)
 * [Step Seven: View the guestbook](#step-seven)
 * [Step Eight: Cleanup](#step-eight)

### Step Zero: Prerequisites <a id="step-zero"></a>

This example assumes that you have a working cluster.  

**Tip:** View all the `kubectl` commands, including their options and descriptions in the [kubectl CLI reference](https://kubernetes.io/docs/user-guide/kubectl-overview/).

### Step One: Create the Redis master pod<a id="step-one"></a>

Use the `examples/guestbook-go/redis-master-controller.json` file to create a [replication controller](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) and Redis master [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/). The pod runs a Redis key-value server in a container. Using a replication controller is the preferred way to launch long-running pods, even for 1 replica, so that the pod benefits from the self-healing mechanism in Kubernetes (keeps the pods alive).

1. Use the [redis-master-controller.json](redis-master-controller.json) file to create the Redis master replication controller in your Kubernetes cluster by running the `kubectl create -f` *`filename`* command:

    ```console
    $ kubectl create -f examples/guestbook-go/redis-master-controller.json
    replicationcontrollers/redis-master
    ```

2. To verify that the redis-master controller is up, list the replication controllers you created in the cluster with the `kubectl get rc` command(if you don't specify a `--namespace`, the `default` namespace will be used. The same below):

    ```console
    $ kubectl get rc
    CONTROLLER             CONTAINER(S)            IMAGE(S)                    SELECTOR                         REPLICAS
    redis-master           redis-master            gurpartap/redis             app=redis,role=master            1
    ...
    ```

    Result: The replication controller then creates the single Redis master pod.

3. To verify that the redis-master pod is running, list the pods you created in cluster with the `kubectl get pods` command:

    ```console
    $ kubectl get pods
    NAME                        READY     STATUS    RESTARTS   AGE
    redis-master-xx4uv          1/1       Running   0          1m
    ...
    ```

    Result: You'll see a single Redis master pod and the machine where the pod is running after the pod gets placed (may take up to thirty seconds).

### Step Two: Create the Redis master service <a id="step-two"></a>

A Kubernetes [service](https://kubernetes.io/docs/concepts/services-networking/service/) is a named load balancer that proxies traffic to one or more pods. The services in a Kubernetes cluster are discoverable inside other pods via environment variables or DNS.

Services find the pods to load balance based on pod labels. The pod that you created in Step One has the label `app=redis` and `role=master`. The selector field of the service determines which pods will receive the traffic sent to the service.

1. Use the [redis-master-service.json](redis-master-service.json) file to create the service in your Kubernetes cluster by running the `kubectl create -f` *`filename`* command:

    ```console
    $ kubectl create -f examples/guestbook-go/redis-master-service.json
    services/redis-master
    ```

2. To verify that the redis-master service is up, list the services you created in the cluster with the `kubectl get services` command:

    ```console
    $ kubectl get services
    NAME              CLUSTER_IP       EXTERNAL_IP       PORT(S)       SELECTOR               AGE
    redis-master      10.0.136.3       <none>            6379/TCP      app=redis,role=master  1h
    ...
    ```

    Result: All new pods will see the `redis-master` service running on the host (`$REDIS_MASTER_SERVICE_HOST` environment variable) at port 6379, or running on `redis-master:6379`. After the service is created, the service proxy on each node is configured to set up a proxy on the specified port (in our example, that's port 6379).


### Step Three: Create the Redis slave pods <a id="step-three"></a>

The Redis master we created earlier is a single pod (REPLICAS = 1), while the Redis read slaves we are creating here are 'replicated' pods. In Kubernetes, a replication controller is responsible for managing the multiple instances of a replicated pod.

1. Use the file [redis-slave-controller.json](redis-slave-controller.json) to create the replication controller by running the `kubectl create -f` *`filename`* command:

    ```console
    $ kubectl create -f examples/guestbook-go/redis-slave-controller.json
    replicationcontrollers/redis-slave
    ```

2. To verify that the redis-slave controller is running, run the `kubectl get rc` command:

    ```console
    $ kubectl get rc
    CONTROLLER              CONTAINER(S)            IMAGE(S)                         SELECTOR                    REPLICAS
    redis-master            redis-master            redis                            app=redis,role=master       1
    redis-slave             redis-slave             kubernetes/redis-slave:v2        app=redis,role=slave        2
    ...
    ```

    Result: The replication controller creates and configures the Redis slave pods through the redis-master service (name:port pair, in our example that's `redis-master:6379`).


3. To verify that the Redis master and slaves pods are running, run the `kubectl get pods` command:

    ```console
    $ kubectl get pods
    NAME                          READY     STATUS    RESTARTS   AGE
    redis-master-xx4uv            1/1       Running   0          18m
    redis-slave-b6wj4             1/1       Running   0          1m
    redis-slave-iai40             1/1       Running   0          1m
    ...
    ```

    Result: You see the single Redis master and two Redis slave pods.

### Step Four: Create the Redis slave service <a id="step-four"></a>

Just like the master, we want to have a service to proxy connections to the read slaves. In this case, in addition to discovery, the Redis slave service provides transparent load balancing to clients.

1. Use the [redis-slave-service.json](redis-slave-service.json) file to create the Redis slave service by running the `kubectl create -f` *`filename`* command:

    ```console
    $ kubectl create -f examples/guestbook-go/redis-slave-service.json
    services/redis-slave
    ```

2. To verify that the redis-slave service is up, list the services you created in the cluster with the `kubectl get services` command:

    ```console
    $ kubectl get services
    NAME              CLUSTER_IP       EXTERNAL_IP       PORT(S)       SELECTOR               AGE
    redis-master      10.0.136.3       <none>            6379/TCP      app=redis,role=master  1h
    redis-slave       10.0.21.92       <none>            6379/TCP      app-redis,role=slave   1h
    ...
    ```

    Result: The service is created with labels `app=redis` and `role=slave` to identify that the pods are running the Redis slaves.

Tip: It is helpful to set labels on your services themselves--as we've done here--to make it easy to locate them later.

### Step Five: Create the guestbook pods <a id="step-five"></a>

This is a simple Go `net/http` ([negroni](https://github.com/codegangsta/negroni) based) server that is configured to talk to either the slave or master services depending on whether the request is a read or a write. The pods we are creating expose a simple JSON interface and serves a jQuery-Ajax based UI. Like the Redis slave pods, these pods are also managed by a replication controller.

1. Use the [guestbook-controller.json](guestbook-controller.json) file to create the guestbook replication controller by running the `kubectl create -f` *`filename`* command:

    ```console
    $ kubectl create -f examples/guestbook-go/guestbook-controller.json
    replicationcontrollers/guestbook
    ```

 Tip: If you want to modify the guestbook code open the `_src` of this example and read the README.md and the Makefile. If you have pushed your custom image be sure to update the `image` accordingly in the guestbook-controller.json.

2. To verify that the guestbook replication controller is running, run the `kubectl get rc` command:

    ```console
    $ kubectl get rc
    CONTROLLER            CONTAINER(S)         IMAGE(S)                               SELECTOR                  REPLICAS
    guestbook             guestbook            k8s.gcr.io/guestbook:v3  app=guestbook             3
    redis-master          redis-master         redis                                  app=redis,role=master     1
    redis-slave           redis-slave          kubernetes/redis-slave:v2              app=redis,role=slave      2
    ...
    ```

3. To verify that the guestbook pods are running (it might take up to thirty seconds to create the pods), list the pods you created in cluster with the `kubectl get pods` command:

    ```console
    $ kubectl get pods
    NAME                           READY     STATUS    RESTARTS   AGE
    guestbook-3crgn                1/1       Running   0          2m
    guestbook-gv7i6                1/1       Running   0          2m
    guestbook-x405a                1/1       Running   0          2m
    redis-master-xx4uv             1/1       Running   0          23m
    redis-slave-b6wj4              1/1       Running   0          6m
    redis-slave-iai40              1/1       Running   0          6m
    ... 
    ```

    Result: You see a single Redis master, two Redis slaves, and three guestbook pods.

### Step Six: Create the guestbook service <a id="step-six"></a>

Just like the others, we create a service to group the guestbook pods but this time, to make the guestbook front end externally visible, we specify `"type": "LoadBalancer"`.

1. Use the [guestbook-service.json](guestbook-service.json) file to create the guestbook service by running the `kubectl create -f` *`filename`* command:

    ```console
    $ kubectl create -f examples/guestbook-go/guestbook-service.json
    ```


2. To verify that the guestbook service is up, list the services you created in the cluster with the `kubectl get services` command:

    ```console
    $ kubectl get services
    NAME              CLUSTER_IP       EXTERNAL_IP       PORT(S)       SELECTOR               AGE
    guestbook         10.0.217.218     146.148.81.8      3000/TCP      app=guestbook          1h
    redis-master      10.0.136.3       <none>            6379/TCP      app=redis,role=master  1h
    redis-slave       10.0.21.92       <none>            6379/TCP      app-redis,role=slave   1h
    ...
    ```

    Result: The service is created with label `app=guestbook`.

### Step Seven: View the guestbook <a id="step-seven"></a>

You can now play with the guestbook that you just created by opening it in a browser (it might take a few moments for the guestbook to come up).

 * **Service Port Forward:**
    It should be a trick to connect to guestbook running in a Kubernetes cluster
  
  ```console
  $ kubectl port-forward svc/guestbook 3000:3000
  Forwarding from 127.0.0.1:3000 -> 3000
  Forwarding from [::1]:3000 -> 3000
  ```
  
 More info: https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

 To view the guestbook, navigate to `http://localhost:3000` in your browser.

### Installing KubeApps
```console
$ curl -fsSL https://raw.githubusercontent.com/fishworks/gofish/master/scripts/install.sh | bash
$ gofish init
$ gofish install helm
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm init
$ helm install --name kubeapps --namespace kubeapps bitnami/kubeapps
$ kubectl create serviceaccount kubeapps-operator
$ kubectl create clusterrolebinding kubeapps-operator --clusterrole=cluster-admin --serviceaccount=default:kubeapps-operator
$ kubectl get secret $(kubectl get serviceaccount kubeapps-operator -o jsonpath='{.secrets[].name}') -o jsonpath='{.data.token}' | base64 --decode\n
$ kubectl port-forward --namespace kubeapps svc/kubeapps 8080:80
```

 * **Service Port Forward:**
    It should be a trick to connect to KubeApps running in a Kubernetes cluster
  
  ```console
  $ kubectl port-forward --namespace kubeapps svc/kubeapps 8080:80
  Forwarding from 127.0.0.1:8080 -> 8080
  Forwarding from [::1]:8080 -> 8080
  ```
 To view the KubeApps, navigate to `http://localhost:8080` in your browser.
