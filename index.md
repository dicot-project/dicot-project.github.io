---
layout: default
title: Introduction to Dicot
---

The high level goal of the project is to build a REST API service which allows
[OpenStack](https://openstack.org) compatible clients to work with
[KubeVirt](https://kubevirt.github.io) and [Kubernetes](https://kubernetes.io)
to run [KVM](https://www.linux-kvm.org) virtual machines.

The goal is that the combination of Kubernetes and KubeVirt provide converged
infrastructure and API services for managing application container workloads
and virtual machines, while Dicot provides compatibility for applications
built against the widely used OpenStack API. It will target two core use cases

* Organizations with existing deployments of OpenStack who wish to run both
  OpenStack and Kubernetes/KubeVirt in parallel for a period of time while
  migrating to Kubernetes/KubeVirt.

* Organizations with a local private cloud based on Kubernetes/KubeVirt who
  wish to additionally make use of public clouds based on OpenStack, or
  vica-verca.

Read more about the [motivation and architecture for Dicot](/about).
