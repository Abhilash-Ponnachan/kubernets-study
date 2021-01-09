# Learn Kubernetes

### `kubectl` commands

#### >> Check Version

```bash
# Get kubernetes version
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.0", GitCommit:"af46c47ce925f4c4ad5cc8d1fca46c7b77d13b38", GitTreeState:"clean", BuildDate:"2020-12-08T17:59:43Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:09:17Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}

# Get version - client only 
$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.0", GitCommit:"af46c47ce925f4c4ad5cc8d1fca46c7b77d13b38", GitTreeState:"clean", BuildDate:"2020-12-08T17:59:43Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```

#### >> Format Output

```bash
# Get version - 'output' as 'json' - format using "jq" utlity
$ kubectl version --client -o json | jq .
{
  "clientVersion": {
    "major": "1",
    "minor": "20",
    "gitVersion": "v1.20.0",
    "gitCommit": "af46c47ce925f4c4ad5cc8d1fca46c7b77d13b38",
    "gitTreeState": "clean",
    "buildDate": "2020-12-08T17:59:43Z",
    "goVersion": "go1.15.5",
    "compiler": "gc",
    "platform": "linux/amd64"
  }
}
```

_Note: we can use `-o json` to format outputs from almost all `kubectl` commands. Then we can pipe it to `jq` and do any further formatting or querying._

#### >> Get Components

```bash
# see cluster information
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.50.2:8443
KubeDNS is running at https://192.168.50.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# list the nodes in the cluster
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   21h   v1.19.4
# ** Minikube - has only one node for bot conrtol-plane as well as worker

# list pods in the cluster
$ kubectl get pods
No resources found in default namespace.
# ** We have not deployed any pods yet

# list name spaces
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   23h
kube-node-lease   Active   23h
kube-public       Active   23h
kube-system       Active   23h
```



#### >> Create Components

```bash
# Create various coponent instances using 'create' command
$ kubectl create --help
Create a resource from a file or from stdin.

 JSON and YAML formats are accepted.

Examples:
  # Create a pod using the data in pod.json.
  kubectl create -f ./pod.json
  
  # Create a pod based on the JSON passed into stdin.
  cat pod.json | kubectl create -f -
  
  # Edit the data in docker-registry.yaml in JSON then create the resource using
the edited data.
  kubectl create -f docker-registry.yaml --edit -o json

Available Commands:
  clusterrole         Create a ClusterRole.
  clusterrolebinding  Create a ClusterRoleBinding for a particular ClusterRole
  configmap           Create a configmap from a local file, directory or literal value
  cronjob             Create a cronjob with the specified name.
  deployment          Create a deployment with the specified name.
  ingress             Create an ingress with the specified name.
  job                 Create a job with the specified name.
  namespace           Create a namespace with the specified name
  poddisruptionbudget Create a pod disruption budget with the specified name.
  priorityclass       Create a priorityclass with the specified name.
  quota               Create a quota with the specified name.
  role                Create a role with single rule.
  rolebinding         Create a RoleBinding for a particular Role or ClusterRole
  secret              Create a secret using specified subcommand
  service             Create a service using specified subcommand.
  serviceaccount      Create a service account with the speci
```

##### Pods, Replica Sets, Deployments

- **Pods**

In _Kubernetes_ the atomic unit of deployment and execution is a **Pod** (`pod`). It is an abstraction over a **container**, and can contain more than one containers in fact (often used with _side-car_ pattern). Each `pod` has its own IP address and shares a PID namespace, network and host-name.

- **Replica Sets**

A **Replica Set** is an object that defines the replication behaviour for the _replication controller_. It would need to specify a _"a Pod template"_ and the _"desired number of Pods"_. This component is responsible to monitor and maintain the desired number of _"replicas"_ of a **"Pod"**; so if a pod dies and the count goes down, then the **Replica Set** will try to bring up a new instance to maintain that count, and vice-versa. It is a _higher-level_ abstraction over pods, so that we do not manually have to deal with the creation and management of pod instances/replicas.

- **Deployments**

Whilst **Replica Sets** provide an abstraction over **Pods**, that automatically manage the instances, to actually _"roll out"_ or _"deploy"_ our **Pods** we rely on yet another _Kubernetes_ object called **Deployment**. This component is responsible for managing the creation (deployment) of our **Pods** (based on some template), and configure a **Replication Set** to manage the replicas/instances. So a **Deployment** is a higher level abstraction component than **Replica Sets** and manages them. In fact as a user of _Kubernetes_ we would only deal with the **Deployment** object and would not have to interact directly with **Pods** or **Replica Sets**, as they are orchestrated by the **Deployment** component. This provides a nice separation-of-concerns for each component in the _Kubernetes_ architecture.

```bash
# Creating an 'nginx' pod (via deployment)
$ kubectl create deployment dpl-nginx-test --image=nginx
deployment.apps/dpl-nginx-test created
# deployment name = dpl-nginx-test, and image = niginx from DockerHub (or Minikube regsitry if we are using that)
```

_Note: Here we have specified only the `image`, and left everything else as defaults._

Now let us list out the **Deployments**, **Replica Sets**, and **Pods** to see what they look like.

```bash
# list the deployments
$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
dpl-nginx-test   1/1     1            1           23s
# One 'deplyment' with its name and status

# list the replicasets
$ kubectl get replicasets
NAME                        DESIRED   CURRENT   READY   AGE
dpl-nginx-test-85c6db9dfb   1         1         1       37s
# One 'replicaset' with a name suffix to the 'deployment'

# list the pods
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
dpl-nginx-test-85c6db9dfb-6jqg7   1/1     Running   0          49s
# One 'pod' with additional name suffix to the 'replicaset'
```

_Note: How the names of the **Pods**, **Replica Sets**, and **Deployments** are related through the suffixes._

##### >> Edit Deployment

Let us try to modify our **Pod** now, say we want to use a different version of `nginx`. To do this we do not edit the **Pod** directly rather the **Deployment**, and that takes care of rolling out the update for us.

```bash
$kubectl edit deployments dpl-nginx-test
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
  creationTimestamp: "2020-12-13T16:50:31Z"
  generation: 2
  labels:
    app: dpl-nginx-test
  name: dpl-nginx-test
  namespace: default
  resourceVersion: "28104"
  selfLink: /apis/apps/v1/namespaces/default/deployments/dpl-nginx-test
  uid: 334072ad-7062-41f9-b0fb-b6a4bd061bb0
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: dpl-nginx-test
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: dpl-nginx-test
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
.....
deployment.apps/dpl-nginx-test edited
```

We will see an auto-generated _YAML_ file which is really a _declarative_ representation of everything need to describe our **Deployment**. We will see later how to create these files and apply them to our cluster. For now if we edit the `image: nginx` line to `image: nginx:1.16` and save the file, _Kubernetes_ will automatically apply this change to the cluster (rather it will initiate the workflow that will eventually make this _"desired state"_ the _"actual state"_ of the cluster).

At this stage if we check the **Deployments**, **Replica Sets** and **Pods** we should see that - the **Deployment** was updated, a new **Replica Set** was created (with the new image specification), and a new **Pod** is created using that while the old one is terminated.

```bash
$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
dpl-nginx-test   1/1     1            1           19h

$ kubectl get pods
NAME                              READY   STATUS              RESTARTS   AGE
dpl-nginx-test-79bff8df68-ppmzb   0/1     ContainerCreating   0          10s
dpl-nginx-test-85c6db9dfb-6jqg7   1/1     Running             1          19h

$ kubectl get pods
NAME                              READY   STATUS        RESTARTS   AGE
dpl-nginx-test-79bff8df68-ppmzb   1/1     Running       0          16s
dpl-nginx-test-85c6db9dfb-6jqg7   1/1     Terminating   1          19h

$ kubectl get pods
NAME                              READY   STATUS        RESTARTS   AGE
dpl-nginx-test-79bff8df68-ppmzb   1/1     Running       0          21s
dpl-nginx-test-85c6db9dfb-6jqg7   0/1     Terminating   1          19h

$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
dpl-nginx-test-79bff8df68-ppmzb   1/1     Running   0          27s

$ kubectl get replicasets.apps 
NAME                        DESIRED   CURRENT   READY   AGE
dpl-nginx-test-79bff8df68   1         1         1       42m
dpl-nginx-test-85c6db9dfb   0         0         0       20h
# The old replicaset has no pods in it
```

The old **Pod** `dpl-nginx-test-85c6db9dfb-6jqg7` got terminated and a new one `dpl-nginx-test-79bff8df68-ppmzb ` was created and running.

##### >> Debug Pods

Often when we want to troubleshoot or debug our cluster, one of the most common things to do is to check the **logs**. To check this let us create another **Pod** for say `MongoDB`.

```bash
$ kubectl create deployment dpl-mongo-db --image=mongo
deployment.apps/dpl-mongo-db created
```

We can now check our **Pod** to see if it is running yet.

```bash
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
dpl-mongo-db-65756887fd-d4hcm     1/1     Running   0          2m20s
dpl-nginx-test-79bff8df68-ppmzb   1/1     Running   0          57m
```

Since `MongoDB` container does a number of things when initialising, and it writes that to the container log, we can see that using the `kubectl logs` command.

```bash
$ kubectl logs dpl-mongo-db-65756887fd-d4hcm
{"t":{"$date":"2020-12-14T13:44:14.007+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
{"t":{"$date":"2020-12-14T13:44:14.009+00:00"},"s":"W",  "c":"ASIO",     "id":22601,   "ctx":"main","msg":"No TransportLayer configured during NetworkInterface startup"}
....
{"t":{"$date":"2020-12-14T13:44:14.555+00:00"},"s":"I",  "c":"INDEX",    "id":20345,   "ctx":"LogicalSessionCacheRefresh","msg":"Index build: done building","attr":{"buildUUID":null,"namespace":"config.system.sessions","index":"lsidTTLIndex","commitTimestamp":{"$timestamp":{"t":0,"i":0}}}}
```

We can in fat format it using `jq` for better readability since it is _JSON_ format.

```bash
alex@kube-test-ub:~$ kubectl logs dpl-mongo-db-65756887fd-d4hcm | jq .
{
  "t": {
    "$date": "2020-12-14T13:44:14.007+00:00"
  },
  "s": "I",
  "c": "CONTROL",
  "id": 23285,
  "ctx": "main",
  "msg": "Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"
}
....
{
  "t": {
    "$date": "2020-12-14T13:44:14.555+00:00"
  },
  "s": "I",
  "c": "INDEX",
  "id": 20345,
  "ctx": "LogicalSessionCacheRefresh",
  "msg": "Index build: done building",
  "attr": {
    "buildUUID": null,
    "namespace": "config.system.sessions",
    "index": "lsidTTLIndex",
    "commitTimestamp": {
      "$timestamp": {
        "t": 0,
        "i": 0
      }
    }
  }
}
```

###### Logging

Logging in container orchestration is in itself a significant topic with its various levels and approaches.

The Kubernetes logging architecture defines three distinct levels:

- **Basic level logging**: the ability to grab pods log using `kubectl` (e.g. `kubectl logs myapp` – where `myapp` is a pod running in my cluster)
- **Node level logging**: The container engine captures logs from the application’s `stdout` and `stderr`, and writes them to a log file.
- **Cluster level logging**: Building upon node level logging; a _log capturing agent_ runs on each node. The agent collects logs on the local filesystem and sends them to a centralised logging destination like _Elasticsearch_ or _CloudWatch_. The agent collects two types of logs:
  - Container logs captured by the container engine on the node.
  - System logs.

Kubernetes, by itself, doesn’t provide a native solution to collect and store logs. It configures the container runtime to save logs in JSON format on the local filesystem. Container runtime – like _Docker_ – redirects container’s `stdout` and `stderr` streams to a [logging driver](https://docs.docker.com/config/containers/logging/configure/). In Kubernetes, container logs are written to `/var/log/pods/*.log` on the node. `Kubelet` and container runtime write their own logs to `/var/logs` or to `journald`, in operating systems with `systemd`. Then cluster-wide log collector systems like `Fluentd` or `ELK` can `tail` these log files on the node and ship logs for retention. These log collector systems usually run as `DaemonSets` on worker nodes.

**There are three common approaches in Kubernetes for capturing and shipping logs to a centralised log aggregation system:**

- **Node level agent**, like a `Fluentd daemonset` or `Logstash daemonset`. This is the recommended pattern. 

  _Note: We shall learn the difference between `daemonset` and `replicaset` later._

- **Sidecar container**, like a `Fluentd` sidecar container.

- **Directly writing to log collection system.** In this approach, the application is responsible for shipping the logs. This is the least recommended option because you will have to include the log aggregation system’s SDK in your application code instead of reusing community build solutions like Fluentd. This pattern also disobeys the *principle of separation of concerns*, according to which, logging implementation should be independent of the application. Doing so allows you to change the logging infrastructure without impacting or changing your application.

###### Describe

Another useful command to examine a **Pod** is the `describe` command that gives comprehensive information about any **resource object**, along with the _"Events"_ in its lifecycle.

```bash
$ kubectl describe pods dpl-mongo-db-65756887fd-d4hcm 
Name:         dpl-mongo-db-65756887fd-d4hcm
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Mon, 14 Dec 2020 13:43:50 +0000
Labels:       app=dpl-mongo-db
              pod-template-hash=65756887fd
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:           172.17.0.3
Controlled By:  ReplicaSet/dpl-mongo-db-65756887fd
Containers:
  mongo:
    Container ID:   docker://5b65774754510aec9399f7ac49fc466036dcd9d6beadaa09571343d3c1c9d376
    Image:          mongo
    Image ID:       docker-pullable://mongo@sha256:02e9941ddcb949424fa4eb01f9d235da91a5b7b64feb5887eab77e1ef84a3bad
.....
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  55m   default-scheduler  Successfully assigned default/dpl-mongo-db-65756887fd-d4hcm to minikube
  Normal  Pulling    55m   kubelet            Pulling image "mongo"
  Normal  Pulled     55m   kubelet            Successfully pulled image "mongo" in 21.978635081s
  Normal  Created    55m   kubelet            Created container mongo
  Normal  Started    55m   kubelet            Started container mongo

```

###### Executing Commands

Occasionally we may want to take a peek inside the **Pod** at the container and probe it. One way to do that is to _execute_ commands on it, possibly by getting access to the _shell_. We can do that using the `kubectl exec` command (it works very similar to the `docker exec` command). The syntax is `kubectl exec (POD | TYPE/NAME) [-c CONTAINER] [flags] -- COMMAND [args...] [options]`.

```bash
# Execute the `date` command on the `momgodb` pod
$ kubectl exec  dpl-mongo-db-65756887fd-d4hcm -- date
Mon Dec 14 16:20:23 UTC 2020
# Find the user the `momgodb` container is running as
$ kubectl exec  dpl-mongo-db-65756887fd-d4hcm -- whoami
root
```

Now let us try to execute commands interactively with the _shell_ of the _container_ attaching our _TTY_.

```bash
$ kubectl exec  -it dpl-mongo-db-65756887fd-d4hcm -- bin/bash
# now we are inside the container as `root`
root@dpl-mongo-db-65756887fd-d4hcm:/# ls
bin   dev                         home        lib64  opt   run   sys  var
boot  docker-entrypoint-initdb.d  js-yaml.js  media  proc  sbin  tmp
data  etc                         lib         mnt    root  srv   usr
```

We can now check and see the logs within the _container_.

```bash
root@dpl-mongo-db-65756887fd-d4hcm:/# ls -la var/log/       
total 668
drwxr-xr-x 1 root    root      4096 Nov 26 01:28 .
drwxr-xr-x 1 root    root      4096 Nov 19 13:09 ..
-rw-r--r-- 1 root    root      4114 Nov 26 01:27 alternatives.log
drwxr-xr-x 1 root    root      4096 Nov 26 01:28 apt
-rw-r--r-- 1 root    root     35330 Nov 19 13:07 bootstrap.log
-rw-rw---- 1 root    utmp         0 Nov 19 13:07 btmp
-rw-r--r-- 1 root    root    227145 Nov 26 01:28 dpkg.log
-rw-r--r-- 1 root    root     32000 Nov 26 01:27 faillog
-rw-rw-r-- 1 root    utmp    292000 Nov 26 01:27 lastlog
drwxr-xr-x 2 mongodb mongodb   4096 Nov 26 01:28 mongodb
-rw------- 1 root    root     64000 Nov 26 01:27 tallylog
-rw-rw-r-- 1 root    utmp         0 Nov 19 13:07 wtmp
```

We can access the _mongodb_ logs (`var/log/mongodb/`) if we wish to analyse that.

So the `exec` command gives us the ability to get a window into the _container_ within the **Pod**.

##### >> Deleting Pods

As with all the other _CRUD_ operations we saw, to delete **Pods** or  **Replica Sets** we do that with the **Deployment**.

```bash
# delete our mongodb deployment
$ kubectl delete deployment dpl-mongo-db
deployment.apps "dpl-mongo-db" deleted

# 'mongodb' pods are gone, only the 'nginx' ones are left
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
dpl-nginx-test-79bff8df68-ppmzb   1/1     Running   0          3h49m
```

_Note: Deleting the `dployment` will also delete the associated `replicaset`._

#### >> Kubernetes Configuration File

To create and modify _Kubernetes resources_ in practice we would almost always rely on **configuration files** (generally in _YAML_ format, but can also be _JSON_), rather than individually create each component with `kubectl create`. In the **configuration file** we specify all the components that we need, and their properties and then use the `kubectl apply` command to make that the _"desired state"_. _This is a **declarative approach** rather than trying to take specify each step using API with an **imperative** approach._

Let us try to create a very simple `nginx` deployment like we did previously, only this time we will use a **configuration file**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dpl-nginx-test
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-web
          image: nginx:1.16
          ports:
          - containerPort: 80		
```

We shall get into the structure and syntax of these files later, for now let us try to create a **Deployment** with this.

```bash
# apply configuration file with `kubectl apply`
$ kubectl apply -f dpl-nginx-test.yaml 
deployment.apps/dpl-nginx-test created

# get deployments
$ kubectl get deployments.apps 
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
dpl-nginx-test   1/1     1            1           4m41s

# get replicasets
$ kubectl get replicasets.apps 
NAME                        DESIRED   CURRENT   READY   AGE
dpl-nginx-test-644599b9c9   1         1         1       5m2s

# get pods
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
dpl-nginx-test-644599b9c9-df5gm   1/1     Running   0          5m9s
```

Again if we wish to change something in our deployment simply, edit the **configuration file** and reapply it. Let us change the `replicas` to `2`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dpl-nginx-test
  labels:
    app: nginx
spec:
  replicas: 2
...
```

Now simply `apply` the configuration file again, and we sill see that _Kubernetes_ will orchestrate these changes to the deployment.

```bash
$ kubectl apply -f dpl-nginx-test.yaml 
deployment.apps/dpl-nginx-test configured

$ kubectl get replicasets.apps 
NAME                        DESIRED   CURRENT   READY   AGE
dpl-nginx-test-644599b9c9   2         2         2       14m
# Replicaset now has 2 instances

$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
dpl-nginx-test-644599b9c9-cc2qf   1/1     Running   0          22s
dpl-nginx-test-644599b9c9-df5gm   1/1     Running   0          15m
# we can see a second node `cc2qf` launched
```

#### >> Configuration file Structure

Whilst the **configuration file** details may differ based on the _Kubernetes object_, they all have a common high-level structure:

- `apiVersion` - the version of the _Kubernetes_ API used to create the object (find this from documentation for object)
- `kind` - what "type" of object are we creating (e.g. `Deployment`, `Service` etc.)
- `metadata` - attributes that help identify the object (`name`), and labels for reference that help associate other objects to this (`labels`)
- `spec` - details of the "desired state", and will depend on the `kind` of the object, and this where we generally specify the `template`
- _`status`_: this is Not specified by us (not part of the "desired state" definition), but it is an internal representation of current state of the object (as saved in`etcd`), and can be get/viewed as a point-in-time snapshot.

Let us examine our **Deployment** **configuration file** to understand the objects and their relationships better.

```yaml
# API version for Deployment
# Note the use of camelCase
apiVersion: apps/v1
# This is a Deployment
kind: Deployment
# Metadata related to the object
metadata:
  # The name for this Deployment object
  name: dpl-nginx-test
  # Labels - additional descriptors
  lables:
    # A custom key-value pair attribute
    app: nginx
# Specification of the object properties
spec:
  # Number of instances/replicas to miantian
  replicas: 2
  # "Relationship" between Deployment and Pods (or other objects)
  selector:
    # Match items with .label.app = nginx
    matchLabel:
      app: nginx
  # Blueprint of the object to be "deployed" (Pods)
  template:
    # Metadata related to the Pod
    metadata:
      # Labels
      lables:
        # Custom k-v pair attribute
        app: nginx
    # Specification for the Pod properties
    spec:
      # Containers within the Pod
      containres:
        # Name of first container
        - name: nginx-web
          # Image for the container
          image: nginx: 1.16
          # Specify ports and port-types for container
          ports:
          - containerPort: 80  
```

One important thing to note is how we use the `selector` to form a relationship between objects. In this case:

```bash

  |   Deployment  |                                                      |   template    |
  |---------------|                                                      |---------------|
  |  labels:      |<---------[selector: matchLabels (app: nginx)]------->|  labels:      |
  |    app: nginx |                                                      |    app: nginx |
  |               |                                                      |               |
  |---------------|                                                      |---------------|

```



#### >> Demo Project Deployment

In this section we will take a sample `Node.js` application with a _frontend web app_ that uses a _backend service app_, that is containerised and try to _deploy_ them on `minikube`. 

- **frontend web app** - This is a simple `Node.js` + `express` web application that has a few HTML elements to take an input (a number in this case), and a button that will POST an `xhr` request with that data. The server-side of the _frontend app_ has a route to accept this request and call an API endpoint of the _backend service_, and return the response ultimately back to the client (browser). The HTML page then displays the response in a text box.

- **backed service app** - This too is another simple `Node.js` + `express` app that has an API endpoint to accept a request with a number as a parameter and check if it is a _"prime number"_ or not. The result of this logic check is sent back to the _frontend app_ as a response. This a synchronous API (over HTTP) communication between the two services/apps. 

- The _port_ and _host (IP)_ of the _backend service_ is specified to the _frontend_ (so that it can locate the  _backend service endpoint_) via a couple of _"environment variables"_ (`SRV_HOST` && `SRV_PORT`).

- Both of these apps have `Dockerfile` that allow them to be packaged as _docker images_.

  The code for this app can be found at https://github.com/Abhilash-Ponnachan/node-demo

Let us create our **Deployment & Service** _YAML_ files. Since they almost always belong together, we typically write them together in one file (separated by `---`).  So our _backend service app_ (`bk-srv-app.yaml`) like like so:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bk-srv-app-dpl
  labels:
    app: bk-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bk-app
  template:
    metadata:
      labels:
        app: bk-app
    spec:
      containers:
      - name: bk-srv-app
        image: img-srv-app
        imagePullPolicy: Never
        ports:
        - containerPort: 3001
---
apiVersion: v1
kind: Service
metadata:
  name: bk-srv
spec:
  selector:
    app: bk-app
  ports:
  - protocol: TCP
    port: 3001
    targetPort: 3001
```

And the _frontend app_ (`fe-tst-app.yaml`) will be like below:

```yaml
metadata:
  name: fe-tst-app-dpl
  labels:
    app: fe-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fe-app
  template:
    metadata:
      labels:
        app: fe-app
    spec:
      containers:
      - name: fe-tst-app
        image: img-tst-app
        imagePullPolicy: Never
        ports:
        - containerPort: 3000
        env:
        - name: SRV_HOST
          value: bk-srv
        - name: SRV_PORT
          value: "3001"
---
apiVersion: v1
kind: Service
metadata:
  name: fe-srv
spec:
  selector:
    app: fe-app
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
    nodePort: 30000
```

> > **Note: With `minikube` if we need to access our local `docker` images, one of the simplest ways to do that is to execute a `mijnikube` command that will point the shells docker commands to target the `minikube` `docker` environment (instead of the host `docker` environment). This is shown in the snippet below. keep in mind though that this is active/valid only in the current shell till we exit from it. If we open a new shell instance this local env setting will not apply. **

```bash
# Run this in current shell -
# - docker commands will use minikube's bundled docker env
# - instead of the host docker environment
$ eval $(minikube docker-env)
# Note: this is valid only in current shell session

```

#### >> Namespaces

**Namespaces** in _Kubernetes_ is analogous to their idea in other aspects like programming languages. They essentially provide a _logical grouping_ construct to organise resources into meaningful groups. This helps manage the complexity of having too many resources all cluttered together, and also helps avoid name conflicts when different groups are working together on the same cluster. If we do not explicitly specify a _namespace_ the resource will be placed in the `default` _namespace_. If we look at the namespaces for a standard deployment we might see something like (_in the case of `minikube`, for full `k8s` it might be slightly different_).

```bash
$ kubectl get namespaces 
NAME              STATUS   AGE
default           Active   26d
kube-node-lease   Active   26d
kube-public       Active   26d
kube-system       Active   26d
```

Another useful feature of _namespaces_ is that we can use that to "control access" (grant permissions) to users at _namespace_ level.

From an administration perspective we can also specify "resource quotas" (CPU, RAM, storage etc.) per _namespace_. This gives us a way to control limits to how much underlying cluster resources a namespace can consume.

One thing to keep in mind is that most _Kubernetes_ resources cannot be shared across _namespaces_. For example the same **ConfigMap** or **Secrets** configuration cannot be used in more than one _namespace_, each _namespace_ will need to define its own. However certain resources make sense to share, like **Services**. When we want to do this we would have to refer to the **service** with its _"fully qualified name"_(suffix with the _namespace_ separated with a `.`). An example might be `db_url: my-sql-db-service.data-namespace`.

Also certain resources live in a _"global" namespace_, that is they cannot be placed into any particular one. Examples are **Volume** and **Node**. In fact we can list the _Kubernetes_ resources that are not namespaced using the following command:

```bash
$ kubectl api-resources --namespaced=false -o name
componentstatuses
namespaces
nodes
persistentvolumes
mutatingwebhookconfigurations.admissionregistration.k8s.io
validatingwebhookconfigurations.admissionregistration.k8s.io
customresourcedefinitions.apiextensions.k8s.io
apiservices.apiregistration.k8s.io
tokenreviews.authentication.k8s.io
selfsubjectaccessreviews.authorization.k8s.io
selfsubjectrulesreviews.authorization.k8s.io
subjectaccessreviews.authorization.k8s.io
certificatesigningrequests.certificates.k8s.io
ingressclasses.networking.k8s.io
runtimeclasses.node.k8s.io
podsecuritypolicies.policy
clusterrolebindings.rbac.authorization.k8s.io
clusterroles.rbac.authorization.k8s.io
priorityclasses.scheduling.k8s.io
csidrivers.storage.k8s.io
csinodes.storage.k8s.io
storageclasses.storage.k8s.io
volumeattachments.storage.k8s.io
```

Of course the same command can be used with `--namespaced=true` to get resources that exist in _namespaces_.

We can also search for resources by _namespace_ using the `--namespace` option.

```bash
$ kubectl get pods --namespace=default
NAME                              READY   STATUS    RESTARTS   AGE
bk-srv-app-dpl-5f69d7fd79-c2cff   1/1     Running   4          15d
fe-tst-app-dpl-b544776c9-dtrvq    1/1     Running   4          15d
```

To get a list of all _Kubernetes_ resources and their details within a namespace we could do something like:

```bash
$ kubectl api-resources -name o | xrags -n 1 kubectl get --show-kind --ignore-not-found --namespace=<my-namespace>
```

This will pipe the result of the first command output to the second one and execute for each result from the first. The second command gets the  details for each resource item passed to it.

If we just want the regular _Kubernetes_ workload resources we could simply do:

```bash
$ kubectl get all -n <my-namespace>
```



When we create (or get) resources without specifying a _namespace_, the `default` _namespace_ is applied. For example if we want to create a **ConfigMap** in a _namespace_ called _"my-namespace"_, then we can do that like so:

```bash
$ kubectl apply -f mysql-configmap.yaml --namespace=backend-srvs
```

Or we can specify that in our _configuration file_ using the `namespace` attribute.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
  namespace: backend-srvs
data:
  db_url: mysql-service.database
```

So this **ConfigMap** might would be created in the `backend-srvs` _namespace_ and have a `db_url` pointing to `mysql-service` in the `database` _namespace_. Now if we want to list **ConfigMaps** from this _namespace_ we can get it by specifying that in the `get` command:

```bash
$ kubectl get configmap -n backend-srvs
```

Since `kubectl` always uses the `default` _namespace_ unless we explicitly specify another one, it can get quite cumbersome to do that with every command if we are exclusively working with a specific _namespace_. To help with that we can install additional tool set like `kubectx` which has a command `kubens` which we can use to change the _"active namespace"_ to what we want. Once we do that, all the `kubectl` commands will apply implicitly to _namespace_ we have made _"active"_.

#### >> Ingress

In our _"demo application"_ we saw how to access our `fe-tst-app` via the `fe-srv` service. This was specified as an **External Service** (`type: LoadBalancer`), with a specified `nodePort` (which has to be in the range `30000` to `32767`). Now the **External Service** will get an **IP** assigned to it and then we can access our service with that **IP** and the exposed **NodePort**. Whilst this works, it has some downsides such as:

- We can have only only **Service** per **Port**.
- Ports have to be in the range `30000` to `32767`.
- If the **Node** IP changes then this has to be handled (perhaps in whatever **Reverse Proxy** setup we have).

A better way to do expose our services external to the **Cluster** is to use an **Ingress** component. It is not a _Kubernetes_ **Service** but an **API Object** that can act as a **Reverse Proxy**, providing a _single entry point_ to the cluster (exposed as external URLs). It can also provide _Load Balancing_, _SSL termination_ & _TLS_ capability, as well as _Domain Based_ and _Path Based_ **Routing** to different **Services**.

###### Ingress Controller

Now In order for **Ingress** to work, there has to be an **Ingress Controller** that can watch and execute the _"Rules"_ for the configured **Ingress Resource**. The **Ingress Controller** runs as a **Pod** (generally in the `kube-system` **Namespace**). 

To see how it works let us pick one implementation of an **Ingress Controller** and install it in our **Cluster**. For `minikube` we can do that with the following command:

```bash
# install minikube addons for ingress
$ minikube addons enable ingress
```

This should install an `nginx` implementation of **Ingress Controller** for `minikube`, in the **`kube-system`** _namespace_.

```bash
$ kubectl get pods -n kube-system
NAME                                        READY   STATUS      RESTARTS   AGE
coredns-f9fd979d6-vnprd                     1/1     Running     15         29d
etcd-minikube                               1/1     Running     15         29d
ingress-nginx-admission-create-vcpxw        0/1     Completed   0          44h
ingress-nginx-admission-patch-hzbrq         0/1     Completed   2          44h
ingress-nginx-controller-558664778f-82zgj   1/1     Running     2          44h
kube-apiserver-minikube                     1/1     Running     15         29d
kube-controller-manager-minikube            1/1     Running     15         29d
kube-proxy-c7bt7                            1/1     Running     15         29d
kube-scheduler-minikube                     1/1     Running     15         29d
storage-provisioner                         1/1     Running     29         29d
```

We can see our **Ingress Controller** running as a **Pod** with name "`ingress-nginx-controller-558664778f-82zgj`". If we want to see further details we can use the `describe` command on the controller **Pod**:

```bash
$ kubectl describe pod -n kube-system ingress-nginx-controller-558664778f-82zgj
Name:         ingress-nginx-controller-558664778f-82zgj
Namespace:    kube-system
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Thu, 07 Jan 2021 16:46:03 +0000
Labels:       addonmanager.kubernetes.io/mode=Reconcile
              app.kubernetes.io/component=controller
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/name=ingress-nginx
              gcp-auth-skip-secret=true
              pod-template-hash=558664778f
Annotations:  <none>
Status:       Running
IP:           172.17.0.4
...
```

###### Add Ingress for our App

Now we can finally modify our application to add an **Ingress** component to expose our _"Demo Project"_. The steps to do this is as follows:

- Change our _frontend_ **Service** (`fe-srv`) to a normal (`Cluster IP`) **Service**. Currently we had set that up as an **External Service** (`Load Balancer`), to make it externally available with a `nodePort`. When using **Ingress** we do not need the `nodePort`, the **Ingress Resource** will communicate with the `Cluster IP` **Service** internally via the _"Cluster Network"_. So now our `fe-srv` configuration file will look like:

  ```yaml
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: fe-srv
  spec:
    selector:
      app: fe-app
  #  type: LoadBalancer - remove this line
    ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  #    nodePort: 30000 - remove this line
  ```

  We simply remove the commented lines. Now if we list the services we should see:

  ```bash
  $ kubectl get service
  NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
  bk-srv       ClusterIP   10.108.24.241   <none>        3001/TCP   17d
  fe-srv       ClusterIP   10.110.46.94    <none>        3000/TCP   17d
  kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    29d
  ```

  Now `fe-srv` is a normal `ClusterIP`  **Service**.

- Next we create an **Ingress** _configuration file_ with routing details to a _backend_ for it, which in our case is the `fe-srv` **Service**.

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: fe-ingress
    annotations:
      #nginx.ingress.kubernetes.io/rewrite-target: /$1
      kubernetes.io/ingress.class: nginx
  spec:
    rules:
    - host: test-app.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: fe-srv
              port:
                number: 3000
  ```

  The **host**/**domain name** that we will use to access this will be `test-app.com`. The default path `/` will route to our **Service** (`fe-srv`) on **Port** `3000`. of course we create the **Ingress Resource** like any other using the `apply` command:

  ```bash
  $ kubectl apply -f fe-ingress.yaml
  ```

- Now we can `get` our **Ingress Object** and see the details:

  ```bash
  $ kubectl get ingress
  NAME         CLASS    HOSTS          ADDRESS        PORTS   AGE
  fe-ingress   <none>   test-app.com   192.168.49.2   80      25h
  ```

  So our **Ingress** is listening on **Port** `80` at **IP** `192.168.49.2`. If we want we can use a browser and navigate to this **IP** and see our application page loaded.

- Note that in there is no **DNS** that resolves our **host name** `test-app.com` to this **IP** yet, so we will not be able to access it with that till we provide some configuration for that. In a real world scenario there would be some **Gateway** or **Reverse Proxy** in front of our **Cluster** or some mechanism that register this **Ingress IP** with a **DNS registry**. For example "**ExternalDNS**" is a tool that can create **DNS records** in an external **DNS server** (like **`Route53`**). Just like the **Ingress Controller**, **External DNS** will run on a **Pod** and will be configured to monitor the **Ingress Objects** and make the necessary **`Route53`** changes.

- In our local environment wit `minikube` though we will keep it simple and just go and create an entry in the `/etc/hosts` file.

  ```txt
  $ cat /etc/hosts
  127.0.0.1		localhost
  127.0.1.1		kube-test-ub
  192.168.49.2	test-app.com
  ```

  That should be all that is needed and we we should be able to enter the **URL** `http://test-app.com` in the browser and everything should load and work as expected.

- If we wish to troubleshoot or see **logs** of the ingress we can do that with the `logs` command (on the **Ingress Controller Pod**):

  ```bash
  $ kubectl logs -n kube-system ingress-nginx-controller-558664778f-82zgj
  ```

- 





