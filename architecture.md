---
title: System architecture
layout: default
---

The high level integration of Dicot with Kubernetes and KubeVirt can be
illustrated with the following diagram:

<img class="diagram" src="images/k8s-architecture.png" title="Kubernetes / Dicot architecture diagram"/>

The key flows in this diagram are

* Kubernetes native clients can launch containers or virtual machines by using
  the Kubernetes API exclusively, with virtual machine based actions being
  forwarded on to KubeVirt.
* OpenStack clients can launch virtual machines using the OpenStack API exposed
  by Dicot.
* When handling an OpenStack API call, Dicot will use the Kubernetes API to
  manage resources in Kubernetes and KubeVirt.
* KubeVirt will use the Kubernetes API to manage the lifecycle of virtual
  machines it is asked to create.

A more detailed look at how Dicot provides OpenStack API services can be
illustrated with the following diagram:

<img class="diagram" src="images/os-architecture.png" title="OpenStack / Dicot architecture diagram"/>

The key points illustrated by this diagram are

* The Dicot API supports the image, compute, identity, network and block
  storage APIs.
* Other OpenStack services, such as database as a service, object store,
  baremetal, shared filesystem and dashboard can be layered above Dicot.
* OpenStack clients talk to either Dicot or the native OpenStack services
  depending on which APIs they need to use.

