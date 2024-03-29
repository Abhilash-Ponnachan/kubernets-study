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

- **Ingress** gives us the ability to _route_ to multiple **Services**. 

  - We can either have multiple **Services** as different **Paths** under the same **Hostname** (for e.g. `https://my-application.com/service1`, `https://my-application.com/service2`).
  - Or we can have the **Services** under different **Hostnames** (typically using different subdomains, for e.g. `https://service1.my-application.com`, `https://service2.my-application.com`).

###### Configuring TLS

So far we have seen how to use **Ingress** only with **HTTP**. _Kubernetes_ makes it rather easy for us to add **TLS** capabilities with **Ingress**. All we have to do is to modify our **Ingress Configuration** with **Specification** information about the **TLS certificate** which can be added to a **Secrets Resource**.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fe-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  tls:
  - hosts:
    - test-app.com
      secretName: test-app-sec-tls
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

We add a section for `tls` with the `secretName` pointing to a _Kubernetes_ **TLS Secret**. To create the actual **TLS certificate** we could use `openssl` (in our case it can be a _self signed certificate_).

```bash
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -out test-app-ingress-tls.crt \
    -keyout test-app-ingress-tls.key \
    -subj "/CN=test-app.com/O=test-app-sec-tls"
```

This will create a `2048` bit `x509` certificate valid for `365` days with a _certificate file_ `test-app-ingress-tls.cert` and a _private key file_ `test-app-ingres-tls.key`.

```bash
$ kubectl create secret tls test-app-sec-tls \
    --key test-app-ingress-tls.key \
    --cert test-app-ingress-tls.cert
```

Next we create a **Secret** passing in these `cert` and `key` files. We can then _reference_ this **Secret** in the **Ingress** routes as we saw in the configuration file above.

_Of course, the **Ingress** and the **Secret** have to be in the same namespace_.

#### >> Helm

### >> Common Commands & Tips

```bash
# shortcut to create configuration file blueprint imperatively
# get a baseline yaml config for Pods
$ kubectl run my-alp-app --image=alpine --dry-run -o yaml >> my-alp-pod.yaml

# open and edit this 'my-alp-app.yaml' config
```

```bash
# simialrly get a definition file from an existing Pod
$ kubectl get pod my-pod-name -o yaml >> my-pod-config.yaml
# edit this file and apply it using 'apply' command to edit the pod
$ kubectl apply -f my-pod-config.yaml
```

```bash
# we can imperatively edit a Pod using the 'edit' command
$ kubectl eidt pod my-pod-name 
```

Of course these commands work for other **K8s** resources such as **Deployments**, **Services** etc., not just **Pods**.

```bash
# see more information with the 'wide' option
$ k get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE    IP            NODE       NOMINATED NODE   READINESS GATES
# this is an easy way to see the Nodes that the Pods run on
```

Formatting outputs using **`awk`**.

```bash
# print Pods and their Nodes
$kubectl get pods -o wide | awk '{print NR ":" $1 "-" $7}'
# 1st column is Pod name and 7th column is Node name
```

Formatting outputs using **`jsonpath`** and **`jq`**.

```bash
# get pod names only using jq
$kubectl get pods -o json | jq '.items[].metadata.name'

# get pod names only using jsonpath
$kubectl get pods -o jsonpath="{@.items[*].metadata.name}"
```

### >> User Authentication (Certificates & kube-config)

```bash
# Extract "ca.crt" && "ca.key" from the control-plane/master node (which has the kube-api server)
# Example command for a "kind" cluster
$ kubectl cp kind-cluster:/etc/kubernetes/pki/ca.crt ca.crt
$ kubectl cp kind-cluster:/etc/kubernetes/pki/ca.key ca.key

# Create certificate for user "Alice", and sign it with kubernetes CA
# Generate rsa key-file
$ openssl genrsa -out alice.key 2048
# Now we should have an RSA key file "alice.key" with private-key 2048 bit long (modulus) & exponent=65537

# Next create a Certificate Signing Request (CSR) for our key
$ openssl req -new -key alice.key -out alice.csr -subj "/CN=Alice Smith/O=Accounting"
# CSR using alice.key for identity/subject "Alice Smith" & organisation/group = "Accounting"

# Use the CSR and our Kubernetes cluster CA to generate and sign a client certificate for Alice
$ openssl x509 -req -in alice.csr -CA kind-ca.crt -CAkey kind-ca.key -CAcreateserial -out alice.crt -days 365
    ...
    Signature ok
    subject=CN = Alice Smith, O = Accounting
    Getting CA Private Key
# Now we should have alice.key & alice.crt signed by kind-ca.crt & kind-ca.key

# Let's create a different kube-config file for "Alice", in order to keep things clean & separate
# for this we require the "server" URL of teh existing cluster, which we can obtain by looking at the existing "config"
$ kubectl config view
# Note the values of cluster.server: https://127.0.0.1:42773 & cluster.name: kind-kind

# Create a new kube-config file using the following command
$ kubectl config set-cluster kind-kind \
  --server=https://127.0.0.1:42773 \
  --certificate-authority=kind-ca.crt \
  --embed-certs=true \
  --kubeconfig="$HOME/.kube/acc-config"
# The last flag "--kubeconfig=$HOME/.kube/acc-config" is VERY IMPORTANT, it makes sure this command POINTS 
# to a different config file (in this case it creates a new one). 
# If we do not specify this the ORIGINAL/DEFAULT config will get modified!!
# Now a new "acc-config" should be aavailabel with the cluster/server details we provided.
# Another (perhaps safer) way to do this is to override the USE of DEFAULT config by specifying the KUBECONFIG env var
# export KUBECONFIG=~/.kube/acc-config
# At this stage we should be able to view the "new config" using kubectl, even though other details are missing.
$ k config view --kubeconfig="$HOME/.kube/acc-config"
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: https://127.0.0.1:42773
      name: kind-kind
    contexts: null
    current-context: ""
    kind: Config
    preferences: {}
    users: null

# Now we have cluster in our "acc-config", next add user(credentials) to it for Alice
$ kubectl config set-credentials alice \
 --client-certificate=alice.crt \
 --client-key=alice.key \
 --embed-certs=true \
 --kubeconfig="$HOME/.kube/acc-config"
# Now a user "alice" (with credentials as alice.crt & alice.key) should be added to our "acc-config" file
$ k config view --kubeconfig="$HOME/.kube/acc-config"
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: https://127.0.0.1:42773
      name: kind-kind
    contexts: null
    current-context: ""
    kind: Config
    preferences: {}
    users:
    - name: alice
      user:
        client-certificate-data: REDACTED
        client-key-data: REDACTED

# Next link the "cluster" to the "user" with a context, so add a new conext (let us call it "dev" for developers)
$ kubectl config set-context dev --cluster=kind-kind --user=alice --namespace=accounting --kubeconfig="$HOME/.kube/acc-config"
# Note we have not created namespace "accounting" just yet, we can do that later
# Here we just specify the "namespace" to beteh default one for this user
# Also Note that the user "alice" in the "config" is just a string for the "config file",
# The Kuernetes cluster does not know or see this, it uses the "CN" (Alice Smith) in the certificate as the identity/principal. 

# In order for "kubectl" to use this conext, we have to make it the "current-context" 
# OR keep specifying this context with every command.
# Let us make this the current-context
$ k config use-context dev --kubeconfig="$HOME/.kube/acc-config"
# Finaly our new kube-cong file "acc-config" should look like
$ k config view --kubeconfig="$HOME/.kube/acc-config"
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: https://127.0.0.1:42773
      name: kind-kind
    contexts:
    - context:
        cluster: kind-kind
        namespace: accounting
        user: alice
      name: dev
    current-context: dev
    kind: Config
    preferences: {}
    users:
    - name: alice
      user:
        client-certificate-data: REDACTED
        client-key-data: REDACTED

# Finally we can use this kube-config ("acc-config") to authenticate to our "kind" cluster
# and try to retrieve some resources
$ k get ns --kubeconfig="$HOME/.kube/acc-config"
 Error from server (Forbidden): namespaces is forbidden: User "Alice Smith" cannot list resource "namespaces" in API group "" at   the cluster scope
# .. And we get an error.. So it looks like we did "authenticate" as "Alice Smith", but this principal is 
# not "authorised" to do anything yet.
# For that we can use RABC.
# Also note that Kubernetes identifies the user as "Alice Smith" which is the value we set as "CN" in the certificate
# subject, and NOT as "alice" which is just a string/token used only within the context of the config file
```

#### >> RBAC

In order to give the user _"Alice Smith"_ permissions to do things in the cluster, we have to create a **Role** and bind that to the _certificate_ using a **RoleBinding**. Since these are **Kubernetes API Resources**, we can create these using the configuration (YAML) files. 

Since we are limiting the user to the `accounting` **namespace** let us first create that

```bash
$ k create ns accounting
```

Next we create a **Role** with the required permissions (in this case just _manage pods_, _list deployments_). To do this we can create a manifest file for the _Kubernetes API Object_ **Role**. The simplest way to have a _starting template_ of the YAML is to _dry-run_ the imperative command and redirect it to a file.

```bash
$ k create role acc-pods --resource=pods --verb=get,list,watch --namespace=accounting --dry-run=client -o yaml > acc-role.yaml
```

Now we should have a `acc-role.yaml` file which we can open and edit and make necessary additions. When done, for our demo purposes it should look like:

```yaml
 1 apiVersion: rbac.authorization.k8s.io/v1
 2 kind: Role
 3 metadata:
 4   name: acc-pods
 5   namespace: accounting
 6 rules:
 7 - apiGroups:
 8   - ""
 9   resources:
 10   - pods
 11   verbs:
 12   - get
 13   - list
 14   - watch
 15   - create
 16   - update
 17   - patch
 18 - apiGroups:
 19   - "apps"
 20   resources:
 21   - deployments
 22   verbs:
 23   - get
 24   - list
```

Now we `apply` the file and create the **Role**.

```bash
$ k apply -n accounting -f acc-role.yaml
```

Next step is to link this **Role** to the **User** (as identified by the `CN` in the certificate) using a **RoleBinding**. Again a simple way to have a starter YAML is to do a _dry-run_ with an imperative command and redirect it.

```bash
$ k create rolebinding acc-alice-rb --user="Alice Smith" --role=acc-pods --namespace=accounting --dry-run=client -o yaml > acc-alice-role-bind.yaml
```

This should give us a **RoleBinding** YAML file:

```yaml
1 apiVersion: rbac.authorization.k8s.io/v1
2 kind: RoleBinding
3 metadata:
4   name: acc-alice-rb
5   namespace: accounting
6 roleRef:
7   apiGroup: rbac.authorization.k8s.io
8   kind: Role
9   name: acc-pods
10 subjects:
11 - apiGroup: rbac.authorization.k8s.io
12   kind: User
13   name: Alice Smith
```

Now we can check it and if we are happy then, `apply` this YAML.

```bash
$ k apply -n accounting -f acc-alice-role-bind.yaml
```

Now the **Role** `acc-pods` should be _bound_ to the **User** `Alice Smith`.

At this stage we can try to use the `kubeconfig` we had created and try to list some resources from our cluster. 

```bash
$ k get pods --kubeconfig=$HOME/.kube/acc-config
No resources found in accounting namespace.
$ k get deploy --kubeconfig=$HOME/.kube/acc-config
No resources found in accounting namespace.
$ k get svc --kubeconfig=$HOME/.kube/acc-config
Error from server (Forbidden): services is forbidden: User "Alice Smith" cannot list resource "services" in API group "" in the namespace "accounting"
```

So with this `kubeconfig`, **Role**, & **RoleBinding**  _Kubernetes_ performs **"authentication"** && **"authorisation"** to allow operations on the cluster. Here the _user_ `Alice Smith` can `list` `pods` and `deployments` but not `services`.

Let us try to **run** a **Pod** using this `kubeconfig`. We shall run a `busybox` **Pod** and execute a `tail -f /dev/null` to keep it running so that the container is not terminated.

```bash
$  k --kubeconfig=$HOME/.kube/acc-config run my-bb --restart=Never --image=busybox -- "/bin/sh" "-c" "tail -f /dev/null"
pod/my-bb created
$  k --kubeconfig=$HOME/.kube/acc-config get pods
NAME    READY   STATUS    RESTARTS   AGE
my-bb   1/1     Running   0          8s
```

So now let us try to **delete** the pod as we don't need it any longer.

```bash
$ k --kubeconfig=$HOME/.kube/acc-config delete pod my-bb --grace-period=0 --force
Error from server (Forbidden): pods "my-bb" is forbidden: User "Alice Smith" cannot delete resource "pods" in API group "" in the namespace "accounting"
```

If we look at the _permission_ we gave to the `acc-pods` **Role**, it does NOT have a `delete` **verb**, therefore **User**  `Alice Smith` will not be allowed to do so.

We can always delete it using our _default admin_ config. 

```bash
$ k -n accounting delete pod my-bb --grace-period=0 --force
```

Now let us assume that we want to allow our user `Alice Smith` to list all the `namespaces`. To do this we will have to use a **ClusterRole** and **ClusterRoleBinding** as `namespaces` are _cluster level_ resources.

Similar to what we did above, we create a **ClusterRole**. Just to make sure we get the values correct, we will take a hybrid approach by creating YAML files imperatively and then checking it, and applying it.

```bash
$ k create clusterrole view-ns --verb=get,list --resource=namespaces --dry-run=client -o yaml > vw-ns-cr.yaml
```

The YAML file should look like:

```yaml
1 apiVersion: rbac.authorization.k8s.io/v1
2 kind: ClusterRole
3 metadata:
4   creationTimestamp: null
5   name: view-ns
6 rules:
7 - apiGroups:
8   - ""
9   resources:
10   - namespaces
11   verbs:
12   - get
13   - list
```

This should give _permissions_ to `list` and `get`  **namespaces**. Now we can `apply` this YAML file and the **ClusterRole** should be created.

```bash
$ k apply -f vw-ns-cr.yaml
```

Next we **bind** this **ClusterRole** (`view-ns`) to our **user** (`Alice Smith`) with a (**ClusterRoleBinding**):

```bash
$ k create clusterrolebinding view-ns-bind --user="Alice Smith" --clusterrole="vw-ns" --dry-run=client -o yaml > vw-ns-cr-bind.yaml
```

The YAML file should be as follows:

```yaml
1 apiVersion: rbac.authorization.k8s.io/v1
2 kind: ClusterRoleBinding
3 metadata:
4   creationTimestamp: null
5   name: view-ns-bind
6 roleRef:
7   apiGroup: rbac.authorization.k8s.io
8   kind: ClusterRole
9   name: view-ns
10 subjects:
11 - apiGroup: rbac.authorization.k8s.io
12   kind: User
13   name: Alice Smith
```

It **binds** the `roleRef` `view-ns` to the `subject` `Alice Smith`. Again we `apply` this file and it should create the **ClusterRoleBinding**.

```bash
$ k apply -f vw-ns-cr-bind.yaml
```

Now time to test it out. One way to check _permissions_ is to use the `auth can-i` command:

```bash
$ k --kubeconfig=$HOME/.kube/acc-config auth can-i get pods
yes
```

We already know we can do that, so now let us see if we can `list` **namespaces**.

```bash
$ k --kubeconfig=$HOME/.kube/acc-config auth can-i list ns
yes
```

And if we try to get the **namespaces** we see we are successful:

```bash
$ k --kubeconfig=$HOME/.kube/acc-config get ns
NAME                 STATUS   AGE
accounting           Active   4h31m
default              Active   170d
...
```

#### >> Service Accounts

Now that we have seen how we can **authenticate** _users_ with _certificates_ and **authorise** them using **RBAC**, we can extend that to understand how **"applications/services"** running in our cluster can securely interact with the _Kubernetes_ API. This is generally used when when we have *CI/CD tools* that need to deploy some resources, or *extensions* like *operators*, *autoscalers*, *custom web hooks*. In general any time we wish to _automate_ something based on _events_ in _Kubernetes_ and/or invoke _Kubernetes APIs_ to take action, we would rely on **Service Accounts**. 

Unlike **Users (or Groups)**,  **Service Accounts** are _Kubernetes API resources_. They can be created and managed using _imperative commands_ or _configuration files_. Once a **Service Account** is created we can link it with **Deployments or Pods**, and when the **Pods** start up they will _"assume"_ the **Service Account**, which means under the hood a **secure token** is created and loaded into a specific path (automatically) mounted into the **Pod**. Applications running in the **Pod** can use this **token** and **authenticate** with _Kubernetes API_ to interact with it. Similar to **users**, we can use **Roles** to specify what _permissions_ we want on the APIs and link that to the **Service Account** using **Role Bindings**.

We can try to simulate this behaviour in our `accounting` namespace.

First step is to create a **Service Account**. Again we shall take a _hybrid_ approach of _imperatively_ creating a YAML file and then applying it. This is juts to make it easier to visualise the fields/properties.

```bash
$ k create sa acc-app-sa -n accounting --dry-run=client -o yaml > sa-acc.yaml
```

This should create a YAML file which looks like:

```yaml
1 apiVersion: v1
2 kind: ServiceAccount
3 metadata:
4   name: acc-app-sa
5   namespace: accounting
```

Now we can apply this configuration file, and create the **Service Account**.

```bash
$ k apply -f sa-acc.yaml
# Note, we did not need to specify the namesapce in the command (like -n accounting)
# this is because, it is already specified in the YAML file.
# Though as a best-practice it is always safer to specify the namespace in the command as well.


# We can see that the serviceaccount is created
$ k get sa -n accounting
NAME         SECRETS   AGE
acc-app-sa   1         14s
default      1         29h

# We should be able to see the details of the serviceaccount
$ k get sa acc-app-sa -n accounting -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"acc-app-sa","namespace":"accounting"}}
  creationTimestamp: "2022-05-31T16:57:45Z"
  name: acc-app-sa
  namespace: accounting
  resourceVersion: "8890581"
  uid: be<redacted>6b8-c776-<redacted>-b8f7-<redacted>
secrets:
- name: acc-app-sa-token-sqt5h
```

When the **Service Account** `acc-app-sa` was created a **token** was generated and stored as a **Kubernetes Secret** (`acc-app-sa-token-sqt5h`).

If we look inside the **Secret** we should be able to see the **token**.

```bash
$ k describe secret acc-app-sa-token-sqt5h -n accounting 
Name:         acc-app-sa-token-sqt5h
Namespace:    accounting
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: acc-app-sa
              kubernetes.io/service-account.uid: be<redacted>6b8-c776-<redacted>-b8f7-<redacted>

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  10 bytes
token:      eyJhbGc---------<redacted>------------.eyJpc3-------------<redacted>-----------EifQ.s7K-----<redacted>----TiWyg
```

So we can see that the **token** issued for the **Service Account** is a **JWT** token. In fact we can examine the _claims_ in this token easily by _Base64 decoding_ the middle part of the **token**.

```bash
# assign the value of 'token' from above to variable $tkn
$ echo $tkn | cut -d '.' -f 2 | base64 -d | jq '.'
..
{
  "iss": "kubernetes/serviceaccount",
  "kubernetes.io/serviceaccount/namespace": "accounting",
  "kubernetes.io/serviceaccount/secret.name": "acc-app-sa-token-sqt5h",
  "kubernetes.io/serviceaccount/service-account.name": "acc-app-sa",
  "kubernetes.io/serviceaccount/service-account.uid": "be<redacted>6b8-c776-<redacted>-b8f7-<redacted>",
  "sub": "system:serviceaccount:accounting:acc-app-sa"
}
```

If we look deeper we can also see the `ca.crt` with which this token is signed.

```bash
$ k get secret acc-app-sa-token-sqt5h -o yaml -n accounting 
apiVersion: v1
data:
  ca.crt: LS0tLS1---<redacted>----LS0tCg==
  namespace: YWNjb3VudGluZw==
  token: ZXlKaGJHY---<redacted>----XeWc=
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: acc-app-sa
    kubernetes.io/service-account.uid: be<redacted>6b8-c776-<redacted>-b8f7-<redacted>
  creationTimestamp: "2022-05-31T16:57:45Z"
  name: acc-app-sa-token-sqt5h
  namespace: accounting
  resourceVersion: "8890580"
  ....
type: kubernetes.io/service-account-token
```

Note that the **"token"** value in this looks different from what we saw above (with the `describe`) command. Actually it is the **same**, it is just _Base64 Encoded_ data since we are looking at the details of the **"Kubernetes Secret"** (it is stored _Base64 Encoded_). If we decode it we will get the same string as we saw previously.

We also see the `ca.crt` data, and if we examine the details it will be the same _Kubernetes Cluster CA certificate_ we saw when were creating the **kube-config** file. We should be able to see the certificate details using the command:

```bash
# assign the vlue of 'ca.crt' from above to the variable 'crt'
# now base64 decode and pass it to openssl
$ echo $crt1 | base64 -d | openssl x509 -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Nov 19 15:21:04 2021 GMT
            Not After : Nov 17 15:21:04 2031 GMT
        Subject: CN = kubernetes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
....
```

Now that we know that **token** (which is a **JWT**) is generated for the **Service Account** and signed by the **Cluster CA certificate**, we shall see how this can be used.

We will do this the easy way, by mimicking an API call using **cURL** from a **Pod** running an `Alpine` _Linux_ container. We have slightly modified the `Alpine` image by installing **cURL** on it. Then we `exec` into this container and make an HTTP call to **Kubernetes** API, using the **token** mounted via the **Service Account**. This seems like a hack approach, but it saves us the trouble of creating an application to test this, and makes it clearer what is happening under the hood. 

So first step is to create a **Pod** YAML manifest that specifies our (modified) `Alpine` image and our `acc-app-sa` **Service Account**.

```bash
# Create a YAML file imperatively 
$ k run dummy-acc-app --restart=Never --image=alpine_plus -n accounting --dry-run=client -o yaml > pod-dummy-app.yaml
```

Note that `alpine_plus` is our modified `alpine` image (with **cURL**).

Now we open this YAML file and add, our **Service Account** to it.

```yaml
  1 apiVersion: v1
  2 kind: Pod
  3 metadata:
  4   name: dummy-acc-app
  5   namespace: accounting
  6 spec:
  7   containers:
  8   - image: alpine_plus
  9     name: dummy-acc-app
 10     imagePullPolicy: Never
 11     command: ["/bin/sh"]
 12     args:
 13       - -c
 14       - >-
 15         echo ===== Dummy App on Alipne with cUrl ====
 16         && tail -f /dev/null
 17   restartPolicy: Never
 18   serviceAccountName: acc-app-sa
```

Note that **Service Account** is at the **Pod** level. Also, since we are using our own `alpine_plus` image which is  a _local Docker image_ (loaded into the `kind` **Cluster**), we should specify the `imagePullPolicy` as `Never`. Additionally we have a `command` to execute something (`tail -f /dev/null`) that will remain running to keep the container from being terminated.

Now we `apply` this file to create our **Pod** in the `accounting` **Namespace**.

```bash
# Apply the pod manifest file to create it
$ k apply -f pod-dummy-app.yaml

# examine the created pod
$ k -n accounting describe pod dummy-acc-app
Name:         dummy-acc-app
Namespace:    accounting
...
IP:           10.244.0.16
IPs:
  IP:  10.244.0.16
Containers:
  dummy-acc-app:
    Container ID:  containerd://.....02901dd5d8
    Image:         alpine_plus
    Image ID:      sha256:....bccbe808b3
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
    Args:
      -c
      echo ===== Dummy App on Alipne with cUrl ==== && tail -f /dev/null
    State:          Running
      Started:      Thu, 02 Jun 2022 12:44:25 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cln5j (ro)
Conditions:
  Type              Status
.....
```

We can see that data related to **Service Account** is injected on to an _automatically mounted volume path_ (`/var/run/secrtes/kubernetes.io/serviceaccount`). Now we can try to `exec` into this and _container_ and look at the _contents_ of this path.

Instead of using the _default admin_ **user**, let us try to `exec` into the **Pod** using our `Alice Smith` **user**. However `Alice Smith` will not have permission to do this, so we have to modify the **Role** bound to this **user** to add the `pod/exec` resource (in the **resources** section).

```yaml
  1 apiVersion: rbac.authorization.k8s.io/v1
  2 kind: Role
  3 metadata:
  4   name: acc-pods
  5   namespace: accounting
  6 rules:
  7 - apiGroups:
  8   - ""
  9   resources:
 10   - pods
 11   - pods/exec
 12   verbs:
 13   - get
 14   - list
 15   - watch
 16   - create
 17   - update
 18   - patch
 19 - apiGroups:
 20   - "apps"
 21   resources:
 22   - deployments
 23   verbs:
 24   - get
 25   - list
```

Now we should be able to exec into our `dummy-acc-app` **Pod** as shown.

```bash
$ k --kubeconfig=$HOME/.kube/acc-config exec -it dummy-acc-app -- /bin/sh
```

Note, that we can set the `KUBECONFIG` _environment variable_ or an _alias_ `kc="kubectl --kubeconfig=$HOME/.kube/acc-config"` if we need to avoid typing the `--kubeconfig` flag with every command. However, I shall leave it like this in the readme, to make it explicit.

Now that we are inside the **Pod** we can look at the _path_ where the _data_ related to the **Service Account** gets injected.

```bash
# We have execed into the pod `dummy-acc-app`
$ ls /var/run/secrets/kubernetes.io/
 serviceaccount
$ ls /var/run/secrets/kubernetes.io/serviceaccount/
 ca.crt     namespace  token
```

There are _three_ files here. The **token** file contains the **JWT** token as its content.

Now we shall try to use this **token** to make a request to _Kubernetes_ API. To make an HTTP call to the _Kubernetes_ API service we need a few parameters, and we shall extract them and set them to **bash** _variables_ to make it easier to handle.

**- The API Server**

```bash
$ APISERVER=https://kubernetes.default.svc
```

If we do an `nslookup` or `dig` on `kubernetes.default.svc`, we will be bale to see the **IP**.

**- Path to Service Account data mount**

```bash
$ PATH_SRV_ACC=/var/run/secrets/kubernetes.io/serviceaccount
```

**- Namespace**

```bash
$ NAMESPACE=$(cat ${PATH_SRV_ACC}/namespace)
```

**- Server CA Certificate**

**cURL** needs this to make a secure **HTTPS** call (as this is a local **CA Certificate** and not in the _list_ of **default CAs** that comes with **cURL** ).

```bash
# Path to `ca.crt` for the 'service account'
$ CACERT=${PATH_SRV_ACC}/ca.crt
```

**- The Token**

The **JWT** **token** issued for the **Service Account**

```bash
$ TOKEN=$(cat ${PATH_SRV_ACC}/token)
```

**Make cURL Request**

Finally we are ready to now make an API request to the _Kubernetes_ API server.

```bash
# HTTPS request to 'list pods'
$ curl --cacert ${CACERT} --header "Authorization: Bearer $TOKEN" -s ${APISERVER}/api/v1/namespaces/$NAMESPACE/pods/

{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:accounting:acc-app-sa\" cannot list resource \"pods\" in API group \"\" in the namespace \"accounting\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```

And, we get a `403` error. As the error message says we can see that _Kubernetes_ has correctly **authenticated** the request (based on the **token**) as the **Service Account** `acc-app-sa` in the **Namespace** `accounting`, and responded with an **authorisation** failure, since this **Service Account** does not have *permission* to *list* **Pods** yet (in fact it has no permission at all yet). To give it _permissions_ we need to do the same thing as we did with **User** (`Alice Smith`). We need to create a **Role** (with _permissions_ needed by the **Service Account**), and link it using **RoleBinding**. We can do that using the following YAMLs.

```bash
# imperatively create YAML file for Role
$ k create role rl-sa-acc --resource=pods --verb=get,list --namespace=accounting --dry-run=client -o yaml > rl-sa-acc.yaml
```

Now we check the YAML file for **Role**, and then `apply` it.

```yaml
  1 apiVersion: rbac.authorization.k8s.io/v1
  2 kind: Role
  3 metadata:
  4   name: rl-sa-acc
  5   namespace: accounting
  6 rules:
  7 - apiGroups:
  8   - ""
  9   resources:
 10   - pods
 11   verbs:
 12   - get
 13   - list
```

Apply the configuration file to create the **Role**.

```bash
$ k apply -f rl-sa-acc.yaml
```

Then, do the same for the **RoleBinding**.

```bash
# Imperatively create rolebinding YAML
$ k create rolebinding rlb-rl-sa-acc -n accounting --role=rl-sa-acc --serviceaccount=accounting:acc-app-sa --dry-run=client -o yaml > rlb-rl-sa-acc.yaml
```

The **RoleBinding** YAML should look like:

```yaml
  1 apiVersion: rbac.authorization.k8s.io/v1
  2 kind: RoleBinding
  3 metadata:
  4   name: rlb-rl-sa-acc
  5   namespace: accounting
  6 roleRef:
  7   apiGroup: rbac.authorization.k8s.io
  8   kind: Role
  9   name: rl-sa-acc
 10 subjects:
 11 - kind: ServiceAccount
 12   name: acc-app-sa
 13   namespace: accounting
```

It _links_ the **subject** `acc-app-sa` (which is a **Service Account**) to the **Role** `rl-sa-acc`.

Now let us `apply` the YAML.

```bash
$ k apply -f rlb-rl-sa-acc.yaml 
```

If we now try to _list_ the **Role**s & **RoleBinding**s in this namespace, we should get:

```bash
$ k get role,rolebinding -n accounting 
NAME                                       CREATED AT
role.rbac.authorization.k8s.io/acc-pods    2022-05-30T12:32:18Z
role.rbac.authorization.k8s.io/rl-sa-acc   2022-06-02T17:13:33Z

NAME                                                  ROLE             AGE
rolebinding.rbac.authorization.k8s.io/acc-alice-rb    Role/acc-pods    3d4h
rolebinding.rbac.authorization.k8s.io/rlb-rl-sa-acc   Role/rl-sa-acc   103s
```

So we can see the **Role**s and **RoleBinding**s for the **user** `Alice Smith` and for our **Service Account** `acc-app-sa`.

Now if we go back into the **Pod** (`exec`) and try calling the API to list **Pods** we should be successful.

```bash
$ curl --cacert ${CACERT} --header "Authorization: Bearer $TOKEN" -s ${APISERVER}/api/v1/namespaces/$NAMESPACE/pods/ 
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "9041098"
  },
  "items": [
    {
      "metadata": {
        "name": "dummy-acc-app",
        "namespace": "accounting",
        "uid": "5a1....a8ff01c",
        "resourceVersion": "9003777",
        "creationTimestamp": "2022-06-02T11:44:24Z",
        ...
        }
     ....
}
# It gives a lot of details

# We can get fancy with 'jq' to filter out only 'pod-names' or any other info
$ curl --cacert ${CACERT} --header "Authorization: Bearer $TOKEN" -s ${APISERVER}/api/v1/namespaces/$NAMESPACE/pods/ | jq '
.items[].metadata.name'

 "dummy-acc-app"
```

Instead of using **cURL**, this could have been an application running in this **Pod** and that can use the _mounted_ **Service Account** to interact with the _Kubernetes_ cluster depending on what its **Role** allows it to do. This is a common pattern used to _extend_ the capabilities of the _cluster_ with _controllers_, or _webhooks_.

