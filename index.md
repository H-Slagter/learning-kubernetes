**
Masters and nodes

A Kubernetes cluster is made up of masters and nodes. These are Linux hosts running on anything from VMs, bare metal servers, all the way up to private and public cloud instances.

Masters (control plane)

The API server

The API Server (apiserver) is the frontend into the Kubernetes control plane. It exposes a RESTful API that preferentially consumes JSON. We POST manifest files to it, these get validated, and the work they define gets deployed to the cluster.

The cluster store

If the API Server is the brains of the cluster, the cluster store is its memory. The config
and state of the cluster gets persistently stored in the cluster store, which is the only
stateful component of the cluster and is vital to its operation - no cluster store, no
cluster!

The cluster store is based on etcd, the popular distributed, consistent and watchable
key-value store.

The controller manager (kube-controller-manager)

The controller manager is currently a bit of a monolith - it implements a few features and functions that’ll probably get split out and made pluggable in the future. Things like the node controller, endpoints controller, namespace controller etc. They tend to sit in loops and watch for changes – the aim of the game is to make sure the current state of the cluster matches the desired state (more on this shortly).

The scheduler (kube-scheduler)
At a high level, the scheduler watches for new workloads and assigns them to nodes. Behind the scenes, it does a lot of related tasks such as evaluating affinity and anti-affinity, constraints, and resource management.

Nodes

The only things that we care about are the kubelet, the container runtime, and the kube-proxy.

Kubelet

First and foremost is the kubelet. This is the main Kubernetes agent that runs on
all cluster nodes.  You install the kubelet on a Linux host and it registers the host with the cluster as a node. It then watches the API server for new work assignments. Any time it sees one, it carries out the task and maintains a reporting channel back to the master.
If the kubelet can’t run a particular work task, it reports back to the master and lets the control plane decide what actions to take. For example, if a Pod fails on a node, the kubelet is not responsible for restarting it or finding another node to run it on. It simply reports back to the master. The master then decides what to do.
The kubelet exposes an endpoint on port 10255 where you can inspect it.

Container runtime

The Kubelet needs to work with a container runtime to do all the container management stuff – things like pulling images and starting and stopping containers.
More often than not, the container runtime that Kubernetes uses is Docker. In the
case of Docker, Kubernetes talks natively to the Docker Remote API.
More recently, Kubernetes has released the Container Runtime Interface (CRI). This
is an abstraction layer for external (3rd-party) container runtimes to plug in to.
Basically, the CRI masks the internal machinery of Kubernetes and exposes a clean
documented container runtime interface.
The CRI is now the default method for container runtimes to plug-in to Kubernetes.
The containerd CRI project is a community-based open-source project porting the
CNCF containerd runtime to the CRI interface. It has a lot of support and will
probably replace Docker as the default, and most popular, container runtime used
by Kubernetes.

Note: containerd is the container supervisor and runtime logic stripped
out of the Docker Engine. It was donated to the CNCF by Docker, Inc.
and has a lot of community support.

Kube-proxy

The last piece of the puzzle is the kube-proxy. This is like the network brains of the
node. For one thing, it makes sure that every Pod gets its own unique IP address. It
also does lightweight load-balancing on the node.

In Kubernetes, things work like this:
1. We declare the desired state of our application (microservice) in a manifest file
2. We POST it the API server
    The most common way of doing this is with the kubectl command. This sends the manifest to port 443 on the master.
3. Kubernetes stores this in the cluster store as the application’s desired state
4. Kubernetes deploys the application on the cluster
    Kubernetes inspects the manifest and identifies which controller to send it to (e.g. the Deployments controller)
5. Kubernetes implements watch loops to make sure the cluster doesn’t vary from desired state
    Kubernetes sets up background reconciliation loops that constantly monitor the state of the cluster. If the current state of the cluster varies from the desired state Kubernetes will try and rectify it.

Pods and containers

It’s true that Kubernetes runs containerized apps. But those containers always run inside of Pods! You cannot run a container directly on a Kubernetes cluster.

Pod anatomy

At the highest-level, a Pod is a ring-fenced environment, an area of the host OS which has a network stack, a bunch of kernel namespaces, which has one or more containers running in it. The Pod itself doesn’t actually run anything.

Pods as the atomic unit

Pods are also the minimum unit of scaling in Kubernetes. If you need to scale your
app, you do so by adding or removing Pods.

Pod lifecycle

Pods are mortal. They’re born, they live, and they die. If they die unexpectedly, we
don’t bother trying to bring them back to life! Instead, Kubernetes starts another one
in its place – but it’s not the same Pod.

Deploying Pods

We normally deploy Pods indirectly as part of something bigger, such as a ReplicaSet
or Deployment.

Deploying Pods via ReplicaSets

A ReplicaSet is a higher-level Kubernetes object that wraps around a Pod and adds
features. As the names suggests, they take a Pod template and deploy a desired
number of replicas of it. They also instantiate a background reconciliation loop that
checks to make sure the right number of replicas are always running – desired state
vs actual state.
ReplicaSets can be deployed directly. But more often than not, they are deployed
indirectly via even higher-level objects such as Deployments.


Deployments

Deployments build on top of ReplicaSets, add a powerful update model, and make
versioned rollbacks simple. As a result, they are considered the future of Kubernetes
application management.


Services

Pods are mortal and can die. If they are deployed via ReplicaSets or Deployments, when they fail, they get replaced with new Pods somewhere else in the cluster - these Pods have totally different IPs! This also happens when we scale an app - the new Pods all arrive with their own new IPs. It also happens
when performing rolling updates - the process of replacing old Pods with new Pods
results in a lot of IP churn.
The moral of this story is that we can’t rely on Pod IPs. But this is a problem.
Assume we’ve got a microservice app with a persistent storage backend that other
parts of the app use to store and retrieve data. How will this work if we can’t rely on
the IP addresses of the backend Pods?
This is where Services come in to play. Services provide a reliable networking
endpoint for a set of Pods.

A Service is a fully-fledged object in the Kubernetes
API just like Pods, ReplicaSets, and Deployments. They provide stable DNS, IP
2: Kubernetes principles of operation 28
addresses, and support TCP and UDP (TCP by default). They also perform simple
randomized load-balancing across Pods, though more advanced load balancing
algorithms may be supported in the future. This adds up to a situation where Pods
can come and go, and the Service automatically updates and continues to provide
that stable networking endpoint.
The same applies if we scale the number of Pods - all the new Pods, with the new
IPs, get seamlessly added to the Service and load-balancing keeps working.
So that’s the job of a Service – it’s a stable network abstraction point for multiple
Pods that provides basic load balancing.

Connecting Pods to Services

The way that a Service knows which Pods to load-balance across is via labels.






















