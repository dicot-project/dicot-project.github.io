---
title: About Dicot
layout: default
---

The [Kubernetes](https://kubernetes.io) project has established itself as the
leading clustered platform for managing containers. While applications are
increasingly being deployed in containers, not everyone is ready or willing to
make the jump from deploying in virtual machines. Thus the
[KubeVirt](https://kubevirt.github.io) project extends the Kubernetes APIs to
facilitate management of virtual machines alongside containers. This enables
cloud administrators to deploy and manage a single infrastructure platform for
both containers and virtual machines. Tenant users will benefit from a converged
API for managing workloads regardless of what they are running in. The
convergance will facilitate interoperability between applications running in
containers and virtual machines and smooth the migration path towards use of
containers for all applications.

While the intended idea / goal is for tenant users to use the Kubernetes &
KubeVirt APIs exclusively for managing containers & virtual machines, the real
world may impose some constraints.

* Organizations with existing deployments of OpenStack may wish to run both
  OpenStack and Kubernetes/KubeVirt in parallel for a period of time.
* Organizations with a local private cloud based on Kubernetes/KubeVirt may
  wish to additionally make use of public clouds based on OpenStack, or
  vica-verca.

The Dicot project aims to be a facilitating technology in those usage scenarios.
At a high level it will assist tenant users in taking tools written to work with
the OpenStack public REST APIs and reuse them to run virtual machine workloads
on Kubernetes using KubeVirt.

It is important to note here that Dicot is not attempting to be the final or
exclusive solution for tenants to use when managing VMs. The expectation is that
tenants will use the Kubernetes / KubeVirt APIs for the majority of their
day-to-day work, in order to see the advantages of having a converged API for
containers and virtual machines. Dicot is a bridging technology to deal with
the scenarios where some degree of interoperability with OpenStack is required,
either as part of the infrastructure migration strategy or for hybrid public /
private cloud usage across multiple vendors.

This constrains the scope of the project to become a more tractable problem.
For example, there is no attempt to provide a seemless data upgrade path from a
running OpenStack cloud to a Kubernetes/KubeVirt cloud. Those with existing
usage of OpenStack would most likely run both solutions in parallel for a period
of time, gradually using Kubernetes for new workloads, leaving legacy workloads
on OpenStack. It also means that while Dicot aims to provide an OpenStack
compatible REST API, it would not need to promise 100% semantic compatibility.
The aim will be to provide strong enough compatibility to allow the majority of
common usage to work, but some scenarios may require users to make adaptations
to tools to take account of semantic differences or missing features.

Overall, Dicot can be considered to be broadly equivalent to the OpenStack
project to build an [Amazon EC2 compatible
API](https://github.com/openstack/ec2-api/blob/master/README.rst) on top of
OpenStack.

