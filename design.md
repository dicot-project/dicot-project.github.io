---
title: Design constraints / implications
layout: default
---

The design of Dicot has constraints placed on it by a handful of initial
requirements or assumptions:

* An existing deployment of Kubernetes and KubeVirt is available for use
* Deployment of Dicot cannot involve changes to the configuration of either
  Kubernetes or KubeVirt
* Resources managed via the OpenStack REST API must be visible in some form
  from the Kubernetes

From these points, and the goals described in the previous section, it is
possible to rule out a number of technical directions

* Direct use of OpenStack services such as Cinder or Neutron by Kubernetes or
  KubeVirt is not acceptable. This would entail administrative reconfiguration
  of these components. Kubernetes does have a storage plugin that can talk to
  Cinder, and an authentication plugin that can talk to Keystone, but the
  decision about whether to use these is upto the Kubernetes adminsitrator. Dicot
  has to accept whatever decision was made.

* Direct use of OpenStack services such as Cinder or Neutron by Dicot itself is
  also not acceptable. In the case of Cinder, this would make the storage config
  of virtual machines launched via Dicot invisible to the Kubernetes API and thus
  diverge from behaviour seen when launching virtual machines with KubeVirt
  directly. This goes counter to the goal of Dicot which is to provide a path
  for convergance on the Kubernetes & KubeVirt APIs for all management.

* Extending Nova to have a KubeVirt compute driver integrated as an alternative
  to the current libvirt / KVM driver. In such a model Nova would ultimately call
  Kubernetes / KubeVirt APIs to manage virtual machine lifecycles, but the rest
  of OpenStack would still be in use. While virtual machine config would become
  visible via the Kubernetes APIs, the rest of OpenStack resources would still
  be hidden, doing litle to help tenants move to a purely Kubernetes/KubeVirt
  based deployment.

If accepted, all of the above directions would also have implications for the
cloud administrator, requiring them to effectively maintain multiple distinct
pieces of infrastructure for the same solution. Having distinct infrastructure
required for storage / networking / etc would run counter to the goal of reducing
the infrastructure burden on cloud administrators.

The above requirements all point towards a design where Dicot exists as a layer
above Kubernetes / KubeVirt, acting simply as an API shim to map between the
declarative OpenStack REST API and the imperative Kubernetes API. As mentioned
earlier, this is a broadly equivalent strategy to that taken by the OpenStack
project to build an [Amazon EC2 compatible
API](https://github.com/openstack/ec2-api/blob/master/README.rst).

Given this design approach, the question is which OpenStack services are required
to be within the scope of Dicot. The core goal is to enable virtual machine
management, so in scope is likely any service which is required in order to
satisfy the Nova REST API functionality. Out of the large number of OpenStack
services this largely boils down to

* Identity service - aka Keystone compatibility
* Image service - aka Glance compatibility
* Network service - aka Neutron compatibility
* Storage service - aka Cinder compatibility
* Compute service - aka Nova compatibility

A number of other services in OpenStack are in turn delivered as layers above
these services. Thus, if the Dicot implementation of these REST APIs is of high
enough accuracy, it could be possible to run other OpenStack services above the
Dicot REST API implementation, in much the same way as OpenStack clients will be
supported. Anything that directly hooks into OpenStack infrastructure databases
or message buses will be categorically unsupported though, because these concepts
will not exist in the Kubernetes based architecture of Dicot.

