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

## How Organizations Use Namespaces

Namespaces are versatile and used by organizations both small and large for different purposes.

For example, small orgs may use namespaces to set boundaries between resources and limit blast radius of any errors or security issues. Larger organizations may use namespaces to separate resources that handle different kinds of workloads, or resources managed by different departments or teams (for example, via [resource quotas](/docs/concepts/policy/resource-quotas/)).

Namespaces are not strictly necessary for Kubernetes to function, and not every team needs the boundaries that namespaces provide. Start using namespaces when you need what they offer!

## How Namespace Scoping Works

- Each resource name is required to be unique within a namespace. Resources in different namespaces can have the same name.

- Namespaces cannot be nested in other namespaces, and each Kubernetes resource can only be in one namespace.

## Tips & Tricks

- It is not necessary to use multiple namespaces to separate slightly different resources, such as different versions of the same software: use {{< glossary_tooltip text="labels" term_id="label" >}} to distinguish resources within the same namespace.

<!-- Why? -->
- For a production cluster, consider _not_ using the `default` namespace. Instead, make other namespaces and use those.

## Out-of-The-Box Namespaces

Without any other configuration Kubernetes starts with four namespaces:

<!-- These are wrong -->

`default`
: Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.

`kube-node-lease`
: This namespace holds [Lease](/docs/concepts/architecture/leases/) objects associated with each node. Node leases allow the kubelet to send [heartbeats](/docs/concepts/architecture/nodes/#heartbeats) so that the control plane can detect node failure.

`kube-public`
: This namespace is readable by *all* clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.

`kube-system`
: The namespace for objects created by the Kubernetes system.

## Working with Namespaces

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


### Setting the namespace for a request

To set the namespace for a current request, use the `--namespace` flag.

For example:

```shell
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```

### Setting the namespace preference

You can permanently save the namespace for all subsequent kubectl commands in that
context.

```shell
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
kubectl config view --minify | grep namespace:
```

## Namespaces and DNS

When you create a [Service](/docs/concepts/services-networking/service/),
it creates a corresponding [DNS entry](/docs/concepts/services-networking/dns-pod-service/).
This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means
that if a container only uses `<service-name>`, it will resolve to the service which
is local to a namespace.  This is useful for using the same configuration across
multiple namespaces such as Development, Staging and Production.  If you want to reach
across namespaces, you need to use the fully qualified domain name (FQDN).

As a result, all namespace names must be valid
[RFC 1123 DNS labels](/docs/concepts/overview/working-with-objects/names/#dns-label-names).

{{< warning >}}
By creating namespaces with the same name as [public top-level
domains](https://data.iana.org/TLD/tlds-alpha-by-domain.txt), Services in these
namespaces can have short DNS names that overlap with public DNS records.
Workloads from any namespace performing a DNS lookup without a [trailing dot](https://datatracker.ietf.org/doc/html/rfc1034#page-8) will
be redirected to those services, taking precedence over public DNS.

To mitigate this, limit privileges for creating namespaces to trusted users. If
required, you could additionally configure third-party security controls, such
as [admission
webhooks](/docs/reference/access-authn-authz/extensible-admission-controllers/),
to block creating any namespace with the name of [public
TLDs](https://data.iana.org/TLD/tlds-alpha-by-domain.txt).
{{< /warning >}}

## Not all objects are in a namespace

Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are
in some namespaces.  However namespace resources are not themselves in a namespace.
And low-level resources, such as
[nodes](/docs/concepts/architecture/nodes/) and
[persistentVolumes](/docs/concepts/storage/persistent-volumes/), are not in any namespace.

To see which Kubernetes resources are and aren't in a namespace:

```shell
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

## Automatic labelling

{{< feature-state for_k8s_version="1.22" state="stable" >}}

The Kubernetes control plane sets an immutable {{< glossary_tooltip text="label" term_id="label" >}}
`kubernetes.io/metadata.name` on all namespaces.
The value of the label is the namespace name.


## {{% heading "whatsnext" %}}

* Learn more about [creating a new namespace](/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace).
* Learn more about [deleting a namespace](/docs/tasks/administer-cluster/namespaces/#deleting-a-namespace).

