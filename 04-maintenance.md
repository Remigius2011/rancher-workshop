
maintenance
===========

even though rancher is quite easy to handle, there are some maintenance tasks to do once ina while.
most frequently, it will be necessary to upgrade workloads once new versions of container images become available.
also, rancher versions themselves change quite often.

goals
-----

in this section we are looking to achieve the following goals:

* upgrade and rollback a workload (our demo webapp)
* upgrade a catalog app (`postgresql`)
* upgrade rancher

upgrading workloads
-------------------

to upgrade an existing workload to a new version of an image (typically available
as a new tag in an existing repo of the same registry), you only have to change the image
tag. navigate to the list of workloads of the `Workshop` project, then enter the `Edit` menu
via the three dot button. then change the current image tag (should be `950eeaed3c`) to `954218a9b9` in the
image field and click the `Upgrade` button. now you can watch the replacement of existing
pods by new ones created from the new image. after all is finished, the greeting will have
changed to 'Howdy' instead of 'Hello'. note that - as we had three pods running
simultaneously - the webapp was continuously available. also, rancher manages all internal
connections between the ingress and the pods as old pods are replaced by new ones. the upgrade
policy and its parameters can be changed in the workload settings on creation or upgrade.

after the workload's pods are recreated, you will see the new commit id after a refresh of the webapp.

***note:*** it may make sense to add the (short form) of the commit id from which a web application was built
to the webapp's metadata in various places, like:

* jar/war manifests
* visible in the gui (e.g. in an about page or on the footer)
* docker image tag
* etc.

this allows to associate app artifacts to the commit from which they were created wherever they appear.
there are plugins to get the commit id for various build tools
(see e.g. the poms in the [demo webapp's github repo](https://github.com/Remigius2011/webapp-hello-java))
it can also easily be determined in a shell script or windows batch file.

rollback
--------

if you need to ***rollback***, i.e. revert the deployment to the last version (or one of the last versions), just use `Rollback` from
the three dot menu of the deployment. you can select from a list of recently deployed versions and review configuration differences.

upgrading postgresql
--------------------

***warning:*** newer versions of the helm chart, namely 1.x and upwards, are completely different and have other
parameters. the description given here only works for 0.x.x. See also [this github issue](https://github.com/helm/charts/pull/8004)
even though updating the helm chart version would be easy, don't go beyond version `0.19`, as this will
break your existing deployment.

postgresql was installed as a catalog app. upgrading it is as easy as upgrading a workload, but
the gui is somewhat different. we can edit the entry for the running postgresql catalog app
and change the parameter (answer) `imageTag` to `9.6.10` to upgrade to the current version of `postgresql 9`.
upgrading directly to `posgresql 10` will fail due to incompatibility of the database file formats. in this case,
you will have to create a dump and restore it to the `postgresql 10` server.

as all our data is on a persistent volume, it survives the creation of a new pod and stays available
(provided the data format is upwards compatible).

upgrading rancher
-----------------

upgrading a running workload is - as we have seen - quite easy and can be done without downtime. upgrading
rancher itself is not quite as easy, but still quite manageable. however, there is some downtime necessary
(at least for a single node install). if you have followed the instructions given
here your rancher server version should be v2.0.8 (see the footer of the rancher gui). we will now proceed
to upgrade it to `v2.1.0`, the latest but not stable version - just to have a look at it.

upgrading rancher to a new version can be performed bny performing the following steps:

* stop the running rancher container
* create a data container that links to the volumes of the active rancher container
* create a tarball containing the data of the current rancher container in case we have to rollback
* start a container with the new version that also links to the volumes of the old one
* once the new version is established to be stable, you can remove the existing container

this leads to the following shell commands to be executed on the docker host on which rancher server is running:

```
$ docker stop rsv208
$ docker create --volumes-from rsv208 --name rancher-data rancher/rancher:v2.0.8
$ docker run --volumes-from rancher-data -v $PWD:/backup alpine tar zcvf /backup/rancher-data-backup-2.0.8-2018-10-17.tar.gz /var/lib/rancher
$ docker run -d --name rsv211 --volumes-from rancher-data --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:v2.1.1
```

then to remove the stopped old rancher container

```
$ docker rm rsv208
```

see also:

* https://rancher.com/docs/rancher/v2.x/en/upgrades/upgrades/
* https://rancher.com/docs/rancher/v2.x/en/upgrades/rollbacks/single-node-rollbacks/
