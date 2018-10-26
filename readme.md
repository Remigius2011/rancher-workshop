
rancher workshop - baselone 2018
================================

welcome to the workshop.

***note:*** using strictly lower case here is *intended*.

important notes
---------------

this section contains important information about updates of this documentation or related artifacts by third parties.

***warning:*** the image for the demo webapp used in the workshop is now available as a
[public image on docker hub](https://hub.docker.com/r/remigius65/webapp-hello-java/).
it will still be available foir a few weeks on the private registry used during
BaselOne, but ***please update the image in your rancher workload*** - otherwise future
upgrades of any kind may fail.

***warning:*** newer versions of the postgresql helm chart, namely 1.x and upwards, are completely different and have other
parameters. the description given here only works for 0.x.x. See also [this github issue](https://github.com/helm/charts/pull/8004).
also, upgrading your deployment to the latest version of the helm chart will kill your deployment - i haven't tested whether the rollback
works in this case, but feel free to do so at your own risk (obviously test with non-production workloads first)

introduction
------------

this repo contains the walk-through for the hands-on sessions.

rancher and kubernetes
----------------------

rancher and kubernetes are tools for container orchestration and cluster management. a short description is given here:

**[rancher](https://rancher.com)** is an open source project that provides means to manage infrastructure for hosting container based applications.
in 2018, the new version 2.0 has been released, which is entirely based on k8s (kubernetes) to manage
the containers - other options available in previous versions like *cattle* and *swarm* have been dropped.

**[kubernetes](https://kubernetes.io/)** - aka ***k8s*** - is an open source project that was initiated by google and is now part of the [cncf](https://www.cncf.io/).
it provides a set of components for container deployment, scaling and management.

outline
-------

the following list shows the topics we are going to cover during the workshop:

1. [setup rancher](00-setup-rancher.md)
1. [create the first cluster](01-create-cluster.md)
1. [run stateless workloads](02-stateless-workloads.md)
1. [run stateful workloads](03-stateful-workloads.md)
1. [maintenance tasks](04-maintenance.md)

there are links to the relevant documentation in each section. a general list of resources related to *rancher* and *k8s* is given here:

* **docker:**  https://www.docker.com/
* **k8s:** https://kubernetes.io/
* **k8s setup:** https://kubernetes.io/docs/setup/pick-right-solution/
* **k8s network providers:** https://chrislovecnm.com/kubernetes/cni/choosing-a-cni-provider/
* **k8s examples:**
  * https://github.com/kubernetes/kubernetes/tree/release-1.7/examples
  * https://github.com/kubernetes/examples
* **rancher:** https://rancher.com/
* **cncf:** https://www.cncf.io/
* **12 factor:** https://12factor.net/
* **app architecture:** https://www.digitalocean.com/community/tutorials/architecting-applications-for-kubernetes
