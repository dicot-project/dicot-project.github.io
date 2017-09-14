---
title: About Dicot
layout: default
---

## Background

The Kubernetes project has established itself as the leading clustered platform
for managing containers. While applications are increasingly being deployed in
containers, not everyone is ready or willing to make the jump from deploying
in virtual machines. Thus the KubeVirt project extends the Kubernetes APIs to
facilitate management of virtual machines alongside containers. This provides
cloud administrators with converged infrastructure to deploy and manager, and
tenant users with a converged API for managing workloads. The convergance will
facilitate interoperability between applications running in containers and
virtual machines and smooth the migration path towards containers.

While the long term goal is full convergence for cloud admins and tenant users
between container and virtual machine management, in the short-to-medium term
there is a clear need for interoperability with existing cloud management
systems. In particular OpenStack is most widely used open source cloud based
virtual machine management system that exists today. There are many projects
in its ecosystem that continue to be interesting to users in a Kubernetes
environment, and effort expended by administrators, developers and users to
write tools around its REST API.

There are a number of ways to provide interoperability between OpenStack and
Kubernetes that have been outlined in a previous document. The two most
interesting options will be outlined here in more detail

### Nova virt driver for KubeVirt

Convergance: single hardware pool

The minimal change approach would be to write a new Nova virt driver that
talked to Kubernetes to accomplish its tasks. Such a driver would be similar
to how the Nova VCenter driver works, where a single compute node in Nova is
in fact an entire cluster of nodes. In such a model, Kubernetes is essentially
only used for virtual machine lifecycle management, everything else in Nova and
OpenStack continues as it exists today. In this model there is very little
convergance between OpenStack and Kubernetes infrastructure. The cloud
administrator has to manage all the OpenStack infrastruture that exists (REST
API, message bus, mysql database, and OpenStack services), as well as the
Kubernetes infrastructure (REST API, etcd, and Kubernetes services). The main
benefit is that they only need a single pool of physical machines and each
machine can dynamically be used for either virtual machines or containers.

The challenges with making this approach successful revolve around how to deal
with overlapping functionality. For example, Kubernetes has dynamic storage
provisioning features that to some extent overlap with Cinder, and a network
plugin concept that overlaps with Neutron. One option is to integrate those
OpenStack services into the Kubernetes deployment. For example, Kubernetes
could use Cinder as its storage provider. The problems with the such an approach
are that it requires a greenfield Kubernetes deployment and makes OpenStack and
Kubernetes co-dependant. This adds complexity for the administrator likely
impacting on reliabilty. A goal of KubeVirt is that it can be deployed onto
any pre-existing Kubernetes cluster, without requiring any changes to it.

The less invasive approach is to strictly run the OpenStack components above
Kubernetes, so the integration is strictly one way. This allows facilitates the
deployment onto Kubernetes, but limits the scope for convergance. Storage and
network concepts in the OpenStack API are invisible if using the Kubernetes
API. Only the low level virtual machine configuration is visible via the
Kubernetes APIs. Thus from the tenant user's POV there is essentially no
convergance between virtual machine and container management APIs.

In theory this minimal change approach could provide a more seemless upgrade
path from existing OpenStack deployments using libvirt and KVM directly, to
one using KubeVirt from Nova. In practice, promising a seemless uggrade path
is fraught with difficulty. Achieving the exact same virtual machine hardware
config between a system using libvirt directly vs KubeVirt would be challenging.
The behaviour of the scheduling policies would be radically different, as there
would be two levels of scheduling taking place. First the OpenStack scheduler
would pick a compute "node", which actually represents an entire Kubernetes
cluster. Second the Kubernetes scheduler would pick the real compute node to
run the VM on. This would make it very challenging, if not impossible, to
ensure that scheduling decisions encoded in flavour extra specs would have the
same effect on the KubeVirt based system.

Conceptually the picture would look like

```
                  /-> Cinder  -> RBD
 VM tenant user --+-> Neutron -> OpenVSwitch
                  \-> Nova -----+
                                |
                                V
 Container tenant user ---> Kubernetes
                                |
                                +- KubeVirt VM driver
                                +- Docker container driver
```

### OpenStack compatible API for KubeVirt

Convergance: single hardware pool, single software stack, single API

The more radical approach is to completely ignore the existing OpenStack
codebase, and instead build a shim above the Kubernetes API that exposes an
OpenStack compatible REST API service, for some subset of OpenStack projects.
This mirrors the approach taken to provide Amazon EC2 API compatibility on
OpenStack.

The cloud administrator gets highly converged infrastructure, since they now
only have to deploy Kubernetes, KubeVirt and an API shim service. Usage of
the rabbit message bus, mysql and countless openstack services is eliminated.
While tenant users & app developers would continue with OpenStack API usage
in the short-to-medium term, all the OpenStack API concepts would have a
representation in the Kubernetes API. This enables them to incrementally switch
over to using the Kubernetes API exclusively, thus gaining full convergance
between virtual machines and containers.

The decision of which OpenStack proejcts to rewrite in terms of API shims, vs
which projects to simply run "as is" has a bearing on the design of the system.
For example, if Cinder is to be run as-is, then the Nova API shim would have
to be built using the Cinder client API for dealing with disk setup. If Cinder
were to also be an API shim, however, the Nova API shim would be able to make
assumptions about the storage backend of the Cinder shim and thus deal with
the Kubernetes API objects exclusively for setting up disks. Even if there was
a simple Cinder API shim and the Nova API shim worked in terms of Kubernetes
APIs, it would not preclude usage of the real Cinder impl too, as Kubernetes
has an internal driver that can utilize Cinder. Similar considerations apply
for for Neutron networking API shims.

The other core component is Keystone which provides authentication. It may be
desirable to have a single Keystone API shim that maps onto Kubernetes APIs,
but still have the other Nova/Cinder/Neutron API shims work in terms of the
Keystone APIs. 

Conceptually the picture would look like

```
                  /-> Cinder API shim  --+
 VM tenant user --+-> Neutron API shim --+
                  \-> Nova API shim    --+
                                         |
                                         V
 Container tenant user ----------> Kubernetes
                                      |
                                      +- KubeVirt VM driver
                                      +- Docker container driver
```

With the option to have a setup like


                  /-> Cinder API shim  --+
 VM tenant user --+-> Neutron API shim --+
                  \-> Nova API shim    --+
                                         |
                                         V
 Container tenant user ----------> Kubernetes
                                      |
                                      +- KubeVirt VM driver
                                      +- Docker container driver
                                      +- Cinder PV driver
                                      +- Keystone auth driver

if there was a desire to continue running Cinder/Keystone as backends inside
Kubernetes.
