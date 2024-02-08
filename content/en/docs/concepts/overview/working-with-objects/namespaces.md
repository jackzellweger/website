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

In Kubernetes, _namespaces_ provides a mechanism for isolating groups of resources within a cluster. 

Namespace-based scoping applies to namespaced {{< glossary_tooltip text="objects" term_id="object" >}} _(e.g. Deployments, Services, etc)_ only, but not to cluster-wide objects _(e.g. StorageClass, Nodes, PersistentVolumes, etc)_.

<!-- body -->

## How organizations use namespaces

Namespaces are versatile and used by organizations both small and large for different purposes.

For example, small organizations might use namespaces to set boundaries between resources running in the cluster and limit blast radius of any errors or security issues. Larger organizations may use namespaces to separate resources that handle different kinds of workloads, or resources managed by different departments or teams (for example, via [resource quotas](/docs/concepts/policy/resource-quotas/)).

Namespaces are not strictly necessary for Kubernetes to function, and not every team needs the boundaries that namespaces provide. Start using namespaces when you need what they offer!

## How namespace scoping works

- Resources in different namespaces can have the same name, but Kubernetes requires that each resource name is unique within a namespace. 

- Namespaces cannot be nested in other namespaces, and each Kubernetes resource can only be in one namespace.

## Out-of-the-box namespaces

Without any other configuration Kubernetes starts with four namespaces:

<!-- TODO: These are wrong -->

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

### Tips and tricks

- Namespaces should only be used to separate resources that differ sufficiently. For example, you can use use {{< glossary_tooltip text="labels" term_id="label" >}} to reference different versions of the same software running <!-- TODO: Rewrite this to be more clear about using labels to label different versions of the same software, that it should be in the same namespace --> resources within the same namespace.

<!-- TODO: Why? -->
- For a production cluster, consider _not_ using the `default` namespace. Instead, make other namespaces and use those.

## Namespaces and DNS

When you create a [Service](/docs/concepts/services-networking/service/), Kubernetes creates a corresponding [DNS entry](/docs/concepts/services-networking/dns-pod-service/) of the form `<service-name>.<namespace-name>.svc.cluster.local`.

### Shorthand to reference services in local namespaces

When communicating with services in their local namespace, containers can use the shorthand `<service-name>` (without the rest of the full domain name `<namespace-name>.svc.cluster.local`), and Kubernetes automatic DNS resolution will resolve to the correct service in the local namespace. This shorthand and automatic DNS resolution is useful when using the same configuration across multiple namespaces—Development, Staging and Production—for example. If containers want to communicate with services outside of the local namespace, it can use the full domain name, which includes the service name, namespace name, and the standard `svc.cluster.local` suffix.

```shell
# When communicating with services in their local namespace
<service-name>

# When communicating with services outside their local namespace
<service-name>.<namespace-name>.svc.cluster.local
```

### Namespace name requirements

As a result of the above domain conventions, all namespace names must be valid [RFC 1123 DNS labels](/docs/concepts/overview/working-with-objects/names/#dns-label-names). This ensures that the names are valid when used in Kubernetes DNS records.

{{< warning >}}
By creating namespaces with the same name as [public top-level domains](https://data.iana.org/TLD/tlds-alpha-by-domain.txt), services with short DNS names run the risk of collision with public DNS records. Workloads from any namespace performing a DNS lookup without a [trailing dot](https://datatracker.ietf.org/doc/html/rfc1034#page-8) will be redirected to those services, taking precedence over public DNS.

<!-- Without a trailing dot, DNS lookups by workloads may first attempt to resolve names using internal DNS configurations. If no internal match is found, and the queried name matches a public domain, the resolver might then query external DNS, potentially leading to responses from public services. Including a trailing dot in the query explicitly bypasses internal search domains, directing the resolver to treat the query as an absolute name and consult the appropriate DNS records directly.-->

To mitigate this, limit privileges for creating namespaces to trusted users. If required, you could additionally configure third-party security controls, such as [admission webhooks](/docs/reference/access-authn-authz/extensible-admission-controllers/) to block creating any namespace with the name of [public top-level-domains](https://data.iana.org/TLD/tlds-alpha-by-domain.txt).
{{< /warning >}}

## Automatic labelling of namespaced resources

{{< feature-state for_k8s_version="1.22" state="stable" >}}

The Kubernetes control plane sets an immutable {{< glossary_tooltip text="label" term_id="label" >}} `kubernetes.io/metadata.name` on all namespaces. The value of that label is the namespace name.


## {{% heading "whatsnext" %}}

* Learn more about [creating a new namespace](/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace).
* Learn more about [deleting a namespace](/docs/tasks/administer-cluster/namespaces/#deleting-a-namespace).

