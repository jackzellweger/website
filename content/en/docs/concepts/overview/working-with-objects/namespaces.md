---
reviewers:
- derekwaynecarr
- mikedanese
- thockin
title: Namespaces
content_type: concept
weight: 45
---

<!-- overview -->

In Kubernetes, _namespaces_ provide a mechanism for isolating groups of resources within a cluster. 

Namespace-based scoping is applicable only to namespaced objects {{< glossary_tooltip text="objects" term_id="object" >}} _(e.g. Deployments, Services)_ only, but not to cluster-wide objects _(e.g. StorageClass, Nodes, PersistentVolumes)_.

<!-- body -->

## How organizations use namespaces

Namespaces are versatile and used by organizations both small and large for different purposes.

For example, small organizations might use namespaces to set boundaries between resources running in the cluster and limit blast radius of any errors or security issues. Larger organizations may use namespaces to separate resources that handle different kinds of workloads, or resources managed by different departments or teams (for example, via [resource quotas](/docs/concepts/policy/resource-quotas/)).

Namespaces are not strictly necessary for Kubernetes to function, and not every team needs the boundaries that namespaces provide. Start using namespaces when you need what they offer!

{{< note >}}
Namespaces should be used to separate significantly different resources. For finer granularity, such as different versions of the same software, use {{< glossary_tooltip text="labels" term_id="label" >}} within the same namespace.
{{< /note >}}

## How namespace scoping works

- Resources in different namespaces can have the same name, but Kubernetes requires that each resource name is unique within a namespace. 

- Namespaces cannot be nested in other namespaces, and each Kubernetes resource can only be in one namespace.

## Out-of-the-box namespaces

Without any other configuration Kubernetes starts with four namespaces:

`default`
: Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.

`kube-node-lease`
: This namespace holds [Lease](/docs/concepts/architecture/leases/) objects associated with each node. Node leases allow the kubelet to send [heartbeats](/docs/concepts/architecture/nodes/#heartbeats) so that the control plane can detect node failure.

`kube-public`
: This namespace is readable by *all* clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.

`kube-system`
: The namespace for objects created by the Kubernetes system.

## Some resources are namespaced, some are not

Most Kubernetes resources—like pods, services, replication controllers, and others—are in some namespace (i.e. are _namespaced_). However, low-level resources, such as [nodes](/docs/concepts/architecture/nodes/), [persistentVolumes](/docs/concepts/storage/persistent-volumes/), as well as namespaces themselves, are not namespaced.

To see which Kubernetes resources are and are not namespaced, we can use `kubectl` with the `--namespaced` flag:

```shell
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

## Working with namespaces

### Namespace creation and deletion

Creation and deletion of namespaces are described in the
[Admin Guide documentation for namespaces](/docs/tasks/administer-cluster/namespaces).

{{< note >}}
    Avoid creating namespaces with the prefix `kube-`, since it is reserved for Kubernetes system namespaces.
{{< /note >}}

### Viewing namespaces

You can list the current namespaces in a cluster using:

```shell
kubectl get namespace
```

```
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```

### Specifying the namespace in `kubectl`

`kubectl` commands apply to the `default` namespace unless otherwise configured. However, certain `kubectl` subcommands take a `--namespace` flag that sets the namespace for the operation. For example:

```shell
# Launch a new pod named 'nginx' using the 'nginx' image in the specified namespace
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>

# Retrieve a list of all pods within the specified namespace
kubectl get pods --namespace=<insert-namespace-name-here>
```

<!-- TODO: Improve writing, make easier to understand -->
You can also save a 'standard' namespace for all subsequent `kubectl` commands in your current context.

```shell
# Save a standard namespace
kubectl config set-context --current --namespace=<insert-namespace-name-here>

# Validate the save of the standard namespace
kubectl config view --minify | grep namespace:
```

## Namespaces and DNS

When you create a [Service](/docs/concepts/services-networking/service/), Kubernetes creates a corresponding [DNS entry](/docs/concepts/services-networking/dns-pod-service/) of the form `<service-name>.<namespace-name>.svc.cluster.local`.

### Referencing services in local namespaces using short DNS names

When communicating with services in their local namespace, containers can use the services short name: `<service-name>` (instead of the entire full domain name `<service-name>.<namespace-name>.svc.cluster.local`), and Kubernetes automatic DNS resolution will resolve to the correct service in the local namespace. This shorthand and automatic DNS resolution is useful when using the same configuration across multiple namespaces—Development, Staging and Production—for example. If containers want to communicate with services outside of the local namespace, it can use the full domain name, which includes the service name, namespace name, and the standard `svc.cluster.local` suffix.

```shell
# When communicating with services in their local namespace
<service-name>

# When communicating with services outside their local namespace
<service-name>.<namespace-name>.svc.cluster.local
```

### Namespace name requirements

As a result of the above domain conventions, all namespace names must be valid [RFC 1123 DNS labels](/docs/concepts/overview/working-with-objects/names/#dns-label-names). This ensures the names are valid when used in Kubernetes DNS records.

{{< warning >}}
As reviewed above, services are addressed in the following way: `<service-name>.<namespace-name>.svc.cluster.local`

Therefore, it's best to avoid namespace names that conflict with any [public top-level-domains](https://data.iana.org/TLD/tlds-alpha-by-domain.txt), for example: `com`, `org`, `gov`, or `yoga`.

For example, let's say you named your namespace `com`. Within `com` is a local service `example`, and a pod `xyz` that wants to connect to `example`. Since pods can use short DNS names to access local resources, `xyz` might just use `example` (without `com.svc.cluster.local`) to try to connect to our `example` service.

If your Kubernetes DNS resolver is configured to append the namespace (`com` in this case) in its search for resolution, it is possible that Kubernetes will, in its search for the proper service to connect to, attempt to resolve to `<service-name>.<namespace-name>.`, or in this case, `example.com`, which is an actual domain name that exists outside the Kubernetes cluster. This could inadvertently lead the DNS query outside of the intended internal cluster network, potentially causing the pod to connect to an external service on the internet rather than the intended internal service.

To mitigate this, limit privileges for creating namespaces to trusted users. If required, configure third-party security controls, such as [admission webhooks](/docs/reference/access-authn-authz/extensible-admission-controllers/) to block the creation of any namespace with the name of [public top-level-domains](https://data.iana.org/TLD/tlds-alpha-by-domain.txt).
{{< /warning >}}

## Automatic labelling of namespaced resources

{{< feature-state for_k8s_version="1.22" state="stable" >}}

The Kubernetes control plane sets an immutable {{< glossary_tooltip text="label" term_id="label" >}} `kubernetes.io/metadata.name` on all namespaces. The value of that label is the namespace name.


## {{% heading "whatsnext" %}}

* Learn more about [creating a new namespace](/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace).
* Learn more about [deleting a namespace](/docs/tasks/administer-cluster/namespaces/#deleting-a-namespace).

