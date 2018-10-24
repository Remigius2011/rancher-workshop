
setup rancher
=============

in this step we are going to run an instance of rancher server. one such instance is able to manage several clusters, once it is up and running.
the rancher server is running independently of the clusters it manages. therefore, if the rancher server stops for any reason, the clusters
are continuing normal operation.

goals
-----

in this section we are looking to achieve the following goals:

* get familiar with rancher server installation and its options
* provide some infrastructure to run rancher server on do (digitalocean)
* install docker
* run a docker container with a rancher server
* get an initial impression of rancher server by looking at its gui

install types
-------------

rancher supports the following install types:

* **single node:** install rancher on a single node. this will be appropriate for small organizations.
* **high availability:** install rancher on a k8s cluster provisioned beforehand. this is good for larger organizations where not only the workloads but also rancher itself must be highly available.
* **air gap:** install rancher on a single node disconnected from the internet.

in the workshop, we perform a *single node* install. for other install types consult the rancher documentation.

see also:

* https://rancher.com/docs/rancher/v2.x/en/installation/
* https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/

prerequisites
-------------

* **infrastructure:** a single host (metal or vm)
* **network:** internet connection, lan to hosts to be provisioned as cluster nodes 
* **os:** ubuntu server 16.04 LTS (xenial) x64
* **docker:** v. 17.03
* **ports:** see [rancher port requirements](https://rancher.com/docs/rancher/v2.x/en/installation/references/)
* **ssl certificate (*highly recommended for production*):** to run https workloads

note that the somewhat outdated versions of os and docker are mostly mandated by k8s. newer versions *may work* but are not (yet) tested
by the k8s community (***note:*** k8s 12.0 adds support for docker 18.06).

see also:

* https://rancher.com/docs/rancher/v2.x/en/installation/requirements/
* https://rancher.com/docs/rancher/v2.x/en/installation/references/
* https://kubernetes.io/docs/setup/independent/install-kubeadm/

provision the host for rancher server
-------------------------------------

in this workshop, we just install a simple ubuntu xenial cloud vm. in a production scenario it is highly recommended to
follow typical recommendations to harden the infrastructure, like mandating certificate based access, installing and
properly configuring a firewall, reducing installed packages and running processes to what's absolutely necessary etc.
it is recommended to use a configuration management tool like [***ansible***](https://www.ansible.com/)
for this purpose, to make this process repeatable. see also the repo for my talk about
[***infrastructure as code***](https://github.com/Remigius2011/iac) and its [slides](https://www.slideshare.net/remigius-stalder/iac-baselone17en).

if you want to keep the results of your activities during this workshop for further experiments, you are welcome to create
an account on digitalocean - aka ***do*** - right now. the link provided here is a referral link that gives you during the month of octiber 2018
a credit of USD 100 valid for 60 days (and me eventually a kick-back of USD 25):

[https://m.do.co/c/4d082f0c649f](https://m.do.co/c/4d082f0c649f)

(conditions: "Everyone you refer gets $100 in credit over 60 days during [Hacktoberfest!](https://hacktoberfest.digitalocean.com/) Once they've spent $25 with us, you'll get $25.")

once the do account is available, you can create a new droplet (this is the do term for a cloud vm) using the green "Create" button
on the upper right with the following parameters:

* **Ubuntu 16.04.4 x64** (make sure to select v. 16.04.4 x64 from the list below "Ubuntu")
* **size: 2GB / 2vCPUs / 60GB / 3TB - $15/mo** (rancher recommends a larger VM, but for small clusters 2GB RAM are sufficient)
* **region: Frankfurt 1** (or any other available region near you)

after short time, the VM will be provisioned and accessible over ssh. the root password will be sent to you by email and you will have to change it on the first ssh login.

***note:*** if you don't feel like using a droplet on do, just provision any other host (metal / vm) with parameters similar to the above ones.
although the example for creating a cluster in this workshop iy also using do, it is not necessary to run rancher and clusters
in the same datacenter / cloud. you can e.g. run rancher on a VMWare VM on your desktop behind a corporate firewall and
manage clusters in a public cloud with it.

install docker
--------------

as mentioned above, it is highly recommended to perform more preparation steps to secure your rancher host, but as this
is a rancher workshop and not a security workshop, we restrict ourselves here to the minimum necessary to run rancher.
you will find plenty of resources helping you to secure an ubuntu host if you search the internet.

to install docker, you will first have to log into your freshly provisioned host using ssh.

***note:*** it is recommended to use a proper ssh terminal (ssh command line / [putty](https://www.putty.org/))
instead of the online ssh login on the do web site, as its character set may not support your keyboard
(i had difficulties with the pipe character - even when pasting the command).

***note:*** mostly all shell commands in this workshop require root access. if you are not logged in as root, just prefix the commands with `sudo` or execute `$ sudo -s` to become root.

the following command installs docker 17.03:

```
$ curl https://releases.rancher.com/install-docker/17.03.sh | sh
```

to verify the installation, run the following:

```
$ docker --version
```

see also:

* https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/quickstart-manual-setup/#2-install-rancher

setup rancher
-------------

rancher enforces the use of `https` to access it (web gui and api). there are
[several possibilities](https://rancher.com/docs/rancher/v2.x/en/installation/single-node/#2-choose-an-ssl-option-and-install-rancher)
to achieve this, here we use the default, which is a self-signed certificate. this means you have to click through
the warning of performing a dangerous operation when starting the web gui. for production workloads it is advisable
a 'real' certificate - either a commercial one or a free one from [let's encrypt](https://letsencrypt.org/).

as can be expected from a container management tool, rancher is distributed as a [docker image](https://hub.docker.com/r/rancher/rancher/).
to run a rancher server container, execute the following command (as root / with sudo - see above):

```
$ docker run -d --name rsv208 --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:v2.0.8
```

immediately after starting the container, run the following command to follow the progress of the ongoing installation:

```
$ docker logs -f rsv208
```

leave this log running during the workshop (or restart it after some interruption), we can observe some possibly interesting
processes in it like creating clusters.

***note:*** the command given above to run rancher server is a slightly enhanced version of the command given in the rancher docs.
firstly, it assigns the container the name `rsv208` (use anything you like, I prefer to make it short and add the rancher version to it),
secondly it uses an explicit version tag instead of `:latest` to make sure the installed rancher version is predictable and repeatable.
btw, it is advisable to consult the available tags of the [rancher server image](https://hub.docker.com/r/rancher/rancher/tags/)
regularly, in particular before installing a new rancher server.

***note:*** when writing these instructions, the current stable version of rancher was `2.0.8` while the latest was `v2.1.0`. meanwhile,
`v2.1.1` (or even a later version - see current [image tags](https://hub.docker.com/r/rancher/rancher/tags/)) is stable. if you like to
start out with the latest stable version, you can directly launch a `v2.1.1` - change the container name accordingly.
however, if youn like to work though the rancher upgrade in the maintenance chapter, start with an earlier version and upgrade
it subsequently according to the instructions.

see also:

* https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/quickstart-manual-setup/#2-install-rancher
* https://hub.docker.com/r/rancher/rancher/tags/

walk-through
------------

after the log has come to a stable state, we can safely assume the rancher server is running.
you can login to the web gui using the url `https://<SERVER_IP>`. on first sign-in you are prompted to
enter a new password for the admin user.

let's walk through some interesting parts of rancher before creating the first cluster. they are accessible through the
toolbar at the top of the page. all terms are from the *english* localization of the gui.

* **Global:** navigation between cluster related and global toolbar items
* **Clusters:** this is the 'home' page of rancher, which will eventually list the clusters (currently empty).
* **Node Drivers:** a list of available node drivers, some of which are activated. if you need to connect to a supported cloud,
    make sure to activate its driver if necessary (e.g. [Exoscale](https://www.exoscale.com/) - a swiss cloud provider - is initially deactivated)
* **Catalogs:** the place to control which helm catalogs are available to run preconfigured workloads - we will start one of them later 
* **Users:** a list of users (currently only admin) - go here to add more users to your rancher server
* **Settings:** some system settings which don't have to be touched under normal circumstances
* **Security:** roles (initially a predefined set), pod security policies and external authentication providers (go here to enable sign-on e.g. using github, OpenLDAP, AD or similar)
* **User Menu:** on the right hand side - rancher api keys, node templates, preferences (change to the 'dark' theme here!) and a logout link
