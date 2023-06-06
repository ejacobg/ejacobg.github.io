---
title: "Hands-On Software Engineering With Golang"
date: 2023-05-09T10:49:23-07:00
draft: true
---

Some notes and my overall experience reading through Achilleas Anagnostopoulos's _[Hands-On Software Engineering with Golang](https://www.packtpub.com/product/hands-on-software-engineering-with-golang/9781838554491)_ (2020). This won't cover everything discussed in the book, just the points I thought were most helpful. 

## Review

I went into this book thinking I could hit a lot of the keywords on my list: Docker, Kubernetes, gRPC, concurrency, software engineering, testing, and a project to add to my portfolio. In the loosest sense you could say I "hit" all of these topics, but some were more focused on than others. You cover just enough Kubernetes terminology to understand all the parts of the cluster, and you cover enough Docker to take up 4 out of ~600 pages of the book. To be fair, this book never claims to be a guide for everything; you learn just enough to continue on with the project and nothing more. The vast majority of the book was dedicated to discussing all the components and iterations of the project. If your goal was to learn each of the aforementioned topics, you may be better served reading a dedicated book on each one. This isn't to say I didn't "get anything" out of the book. In fact, I got quite a bit out of it:

* A general-purpose Dockerfile for building Go programs.
* A three-legged OAuth flow implementation.
* Basic knowledge of protocol buffers and gRPC.
* Introduction to packages like GoCheck and tools like `go generate`.

Do note that this book was published in 2020, and at the time of writing, the current Go version was **1.12.5**. Needless to say, a lot of the packages, software, and tools used in the book have changed and updated since the book was first written. The official [code repository](https://github.com/PacktPublishing/Hands-On-Software-Engineering-with-Golang) for the book has been updated to require Go 1.18+, however you will still find that it relies on older packages and tools (e.g. the Makefile targets for minikube make use of Helm v2 as opposed to the current v3). I've done my best to modernize the repository, however I never actually got the Kubernetes cluster to be deployed. You can find my changes here: (link)

Overall, the author does a good job of explaining everything that's going on, using both high level architecture diagrams and low-level implementation details. I never felt "lost" while looking at any particular piece of code. Even though I wasn't able to deploy my Kubernetes cluster, the program does work when running it in memory.  

## Notes

These will mainly go over the software engineering parts of the book. For a discussion of the _Links 'R' Us_ project, see its associated blog post.

### SOLID Principles

**Single responsibility** - In any well-designed system, objects should only have a single responsibility.

**Open/closed** - A software module should be open for extension but closed for modification.

In Go, you are not allowed to redefine the behavior of types you've imported from other packages. You can satisfy the closed principle (for packages) for free.

Even when you are inside a package and have control over the implementation of your classes, you can still make use of the open/closed principle. Use composition to obtain the methods from other types, and override any methods as needed.

```go
type Sword struct {
    name string
}

// Damage returns the damage dealt by this sword.
func (Sword) Damage() int {
    return 2
}

type EnchantedSword struct {
    // Embed the Sword type
    Sword
}

// Damage returns the damage dealt by the enchanted sword.
func (EnchantedSword) Damage() int {
    return 42
}
```

**Liskov substitution** - Two types _S_ and _T_ are substitutable if the behavior of programs _P<sub>S</sub>_ and _P<sub>T</sub>_ are the same. Program to interfaces rather than concrete types.

**Interface segregation** - Clients should not be forced to depend on the interfaces they do not use. Define only the smallest interfaces needed by your clients.

**Dependency inversion** - High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

The 4 preceding principles all set the stage for this final one. If you've done things correctly, then you will get dependency inversion for free.
* The ISP allows both high- and low-level modules to rely on abstractions.
* The open/closed principle allows abstractions (interfaces) to be decoupled from details (implementations).
* The LSP allows our implementation details to rely on abstractions.

---

What you gain in flexibility, you lose in complexity and size. Always consider whether defining an abstraction is worth it (it surprisingly isn't for a lot of cases).

Encountering difficulties when testing may be a sign that one or more of your SOLID principles have been violated and need refactoring.

SOLID principles apply outside of coding. Your microservices should follow the SRP. They should communicate using clearly defined contracts and boundaries (ISP).

### Linters

There are certain static linkers that can help you write your code. These can detect common errors or vulnerabilities. Here are some checks that you might want to add to your `Makefile`:

* `bodyclose` - check if your `http.Response` bodies get closed.
* `aligncheck` - check if your structs use more padding than needed.
* `prealloc` - identify slice declarations that could be preallocated.

* You can use a "meta-linter" like `golangci-lint` to run your linters in parallel, and normalize all of their output.

### Key-value stores

Popular key-value stores include AWS DynamoDB, LevelDB, and RocksDB.

Key-value stores typically support insertions, deletions, and lookups. Some may also support range queries, which return all the key-value pairs whose keys fall within a given range.

Key-value stores can accept many kinds of data as their keys and values.

Key-value stores can easily be partitioned across multiple nodes. They scale very well horizontally.

Common use cases for key-value stores include:

* Caches (eg. web pages, database responses).
* Storing session data. If we tag each session with a unique ID, then we don't have to mess with a relational database and a load balancer; the key-value store does this naturally.
* Storage layers for database systems. Some databases like CockroachDB, and Apache Cassandra are backed by key-value stores.

The main drawback of key-value stores is that while it is easy to search the keys, it is not easy to search through the values.

### Relational databases

One of the most important features of relational databases is their support for transactions. In order for a RDBMS to execute transactions effectively, it must satisfy the so-called ACID properties.

* **Atomicity** - transactions are executed in full or not at all.
* **Consistency** - transactions must leave the database in a valid state.
* **Isolation** - transactions should occur independently of one another. Running transactions concurrently should produce the same result as running them sequentially.
* **Durability** - once a transaction is committed, changes should persist even when the database is restarted.

RDBs tend to scale well vertically, but not horizontally.

For write-heavy workloads, *sharding* is typically used, where the database tables are split across multiple nodes. This comes at the cost of read speed, since multiple nodes need to be queried to obtain the full dataset.

For read-heavy workloads, horizontal scaling is achieved through *read-replicas*, where each node is a mirror of one or more "primary" nodes. Writes are always handled by the primary nodes, while reads can occur on any node.

RDBs are good for transactional workloads and complex queries. They are *not* good if your data is hierarchical or if you are managing very high volumes.

### OAuth

OAuth is an authentication standard proposed as an alternative to basic authentication.

OAuth allows us to have service A access some of our account information hosted on service B, but without us having to give service A our login information to service B.

Some common use cases for this include:

* **Single Sign-On (SSO)** - this allows us to manage multiple accounts with different services, but without having to manage different logins for all of them. Some providers also provide controls for revoking a service's access to your information.
* **API automation** - this is common for things like CI pipelines, where you might need to pull data from a repo and integrate with many other services.

We will implement a "three-legged" OAuth flow which involves 4 parties:

* The **Resource Owner** - the user who wishes for service A to access their data on service B.
* The **OAuth client** - the service who wishes to perform an action on behalf of a user (in this case, service A).
* The **Resource Server** - the service who hosts the resource that the user wishes to share (in this case, service B).
* The **Authorization Server** - a part of service B that authorizes access tokens into service B.

The OAuth flow looks like this:

* The resource owner visits service A and clicks the "Login with B" button.
* Service A returns an authorization URL to the user that points to service B's authorization server.
  * This URL contains information such as service A's client ID, its access requests, and a return URL to visit after the permissions have been granted.
* The resource owner visits the URL, which returns a consent form showing what data and permissions are being granted. If the user grants these permissions, then they will be redirected to the return URL specified in the previous step.
  * The authorization server will append two more pieces of information attached to the return URL: a nonce value and an access code.
* When the return URL is visited, service A will confirm the nonce and make a request to the authorization server with the access code.
* The server will respond to service A's access code with two tokens: a short-lived token that provides access to the requested resources, and a long-lived token that allows service A to re-generate any expired access tokens.
* Service A is now able to perform actions on service B using its access token.

The `golang.org/x/oauth2` package provides some primitives for working with a three-legged OAuth flow.

### Kubernetes Basics

Kubernetes the de facto standard for deploying applications. It is a platform for managing containerized workloads.

There are two types of nodes in a Kubernetes cluster: the **master** nodes, which manage the "control plane", and the **worker** nodes which execute tasks assigned by the masters.

There are typically multiple master nodes, each with a (live) copy of the control plane.

All master nodes run the following processes:

* The **kube-api-server** - an API that allows workers and cluster operators to access the cluster's control plane.
* **etcd** - a key-value store that represents the cluster's current state. Clients can subscribe to certain keys to receive notifications when their values change.
* The **scheduler** - monitors incoming workloads and assigns tasks to workers.
* The **cloud controller manager** - acts as the interface between the cloud provider hosting the cluster and the cluster itself. It will manage any cloud-specific services like storage, load balancing, and routing.

Kubernetes is a *container* manager, so all workers in a cluster must have some sort of container runtime, usually Docker, but some other runtimes like containerd or rkt may be used.

Work is assigned to a worker in the form of a **pod**, which represents one or more container images to execute on that worker.

Worker nodes also run the following processes alongside their workloads:

* The **kubelet** agent - connects to the master's **kube-api-server** and watches for incoming workloads. It also restarts any containers that die while running.
* The **kube-proxy** - routes any internal or external traffic to other pods hosted on this worker.

Operators interact with the Kubernetes cluster through a CRUD-like interface. The cluster provides several "resources" that you can operate on.

The **config map** resource allows us to inject configuration settings into a pod at runtime. It is typically bad practice to hard-code our configuration settings since doing so can make them hard to change. The **secret** resource works similarly to a config map, but for sensitive data like keys and credentials.

The **namespace** resource allows us to group several nodes together and control access and resources to them. This can be useful if multiple teams need to use the same cluster.

The **persistent volume (PV)** and **persistent volume claim (PVC)** resources allow for data to be persisted in the event of a pod crash.

* A PV is simply a piece of block storage made available to the cluster.
* A PVC is a *claim* for a particular piece of block storage (eg. it must have a certain size, IOPS, must be SSD, etc.). The control plane handles assigning block storage to specified claims.

The **deployment** resource is the recommended method for deploying a Kubernetes cluster. The deployment resource specifies a **template** for instantiating a pod and the number of replicas, and Kubernetes tries to synchronize the cluster to satisfy the requests laid out in the template.

* Deployment pods are given a random hostname, and each of them share the same PVC with all other pods in the deployment.

Some applications require "stateful" deployments where the hostnames for each pod are known and each pod requires its own PVC. For example, the hostname of the pod hosting the database should always be known. The **StatefulSet** resource is similar to the deployment resource, where we define a pod template and its replicas. However, instead of being given a random hostname, we define a name for the StatefulSet itself, and each pod is identified by its index in the set (eg. web-0 and web-1).

The **service** resource allows a pod to communicate with other resources in the cluster. There are two types of service resources:

* The **load balancer** resource acts as a gateway to a group of pods. This functionality is provided by the kube-proxy process of the worker node.
  * Load balancers are given an IP address and DNS record so that it can be discoverable by clients.
* The **headless** service allows you to define your own service discovery mechanism. Headless services are not given an IP address, but are given DNS records for the pods behind the service.

The **ingress** service exposes HTTP and HTTPS endpoints to handle traffic originating outside of the cluster.
