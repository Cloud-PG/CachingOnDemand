# On-Demand XCache cluster

## What's XCache

XCache description is available in this article [here](https://iopscience.iop.org/article/10.1088/1742-6596/513/4/042044/pdf).

You can look at the [official XrootD documentation](http://xrootd.org/docs.html) for detailed information about the XRootD tool:

* [basic configuration](http://xrootd.org/doc/dev47/xrd_config.htm)
* [cmsd configuration](http://xrootd.org/doc/dev45/cms_config.htm)
* [proxy file cache](http://xrootd.org/doc/dev47/pss_config.htm)

## XCache components

The setup infrastructure is shown in Fig. B8, where the clients that run the payload can be instructed to request data to a cache system deployed on the same cloud provider and thus with low latency. The cache stack consists in:
a proxy server to function as bridge between the private network of the cache and the client. This server will simply tunnel the request from cache servers.
a cache redirector for federating each cache server deployed. If a new server is added, it will be automatically configured to contact this redirector for registration
a configurable number of cache servers, the core of the tool that are responsibles for reading-ahead from remote site while caching.

![Schema of the components deployed for using a caching on-demand system on cloud resources](https://github.com/Cloud-PG/CachingOnDemand/blob/master/docs/img/arch.png)

This setup has been tested on different cloud providers. It is also been tested at a scale of 2k concurrent jobs on Open Telekom Cloud resources in the context of HelixNebulaScience Cloud [6] project.
In the context of the eXtreme Data-Cloud project [7], a collection of recipes have been produced for the automatic deployment of a cache service on demand using different automation technology. For bare metal installation an Ansible [8] playbook is available that can deploy either directly on host or through docker container the whole stack. For those who use docker swarm for container orchestration a docker-compose [9] recipe is also available as for Kubernetes where an Helm [10] chart is provided. All these solutions have been integrated in DODAS [11] and thus with very few changes the same setup can be automatically replicated in different kind of cloud resources.

### Deployment on Kubernetes with Helm

```bash
helm init --upgrade
helm repo add  cloudpg https://dodas-ts.github.io/docker-img_cms/
helm repo update
helm install cloudpg/cachingondemand
```

More details in this [demo](demo/DEMO.md)

## Ansible deployment

### Requirements

* Ansible 2.4
* OS: Centos7
* valid CMS /etc/vomses
* Port: one open service port
* Valid grid host certifate
* Valid service certificate that is able to read from remote origin (to be stored in /etc/grid-security/xrd/xrdcert.pem, /etc/grid-security/xrd/xrdkey.pem)

### Role Variables

``` yaml
BLOCK_SIZE: 512k # size of the file block used by the cache
CACHE_LOG_LEVEL: info # server log level
CACHE_PATH: /data/xrd # folder for cached files
CACHE_RAM_GB: 12 # amount of RAM for caching in GB. Suggested ~50% of the total
HI_WM: "0.9" # higher watermark of used fs space
LOW_WM: "0.8" # lower watermark of used fs space
N_PREFETCH: "0" # number of blocks to be prefetched
ORIGIN_HOST: origin # hostname or ip adrr of the origin server
ORIGIN_XRD_PORT: "1094" # xrootd port to contact origin on
REDIR_HOST: xcache-service # hostname or ip adrr of the cache redirector
REDIR_CMSD_PORT: "31213" # cmsd port of the cache redirector
metricbeat_polltime: 60s # polling time of the metricbeat sensor
metric_sitename: changeme # sitename to be displayed for monitoring
elk_endpoint: localhost:9000 # elasticsearch endpoint url
elastic_username: dodas # elasticsearch username
elastic_password: testpass # elasticsearch password
```

### Example Playbook

```yaml
---
- hosts: localhost
  remote_user: root
  roles:
    - role: dciangot.xcache 
```

### Deployment example: CMS XCache

[https://xcache.readthedocs.io/en/latest/automated-grid.html](https://xcache.readthedocs.io/en/latest/automated-grid.html)

## Deployment with Docker

[https://hub.docker.com/r/cloudpg/xrootd-proxy/](https://hub.docker.com/r/cloudpg/xrootd-proxy/)

### On-demand XCache docker image and deployment with docker compose

Please follow the instruction [here](https://github.com/Cloud-PG/CachingOnDemand/blob/master/docker/README.md)


## Deployment with dodas

- [Kubernetes cluster](https://github.com/Cloud-PG/CachingOnDemand/blob/master/toscaTemplates/DODAS-TS/kube_deploy.yml)
  - [Kubernetes deployment charts](https://github.com/Cloud-PG/CachingOnDemand/tree/master/toscaTemplates/k8s)
- [Real case example: TOSCA template for XCache in CMS experiment with Marathon](https://github.com/Cloud-PG/CachingOnDemand/blob/master/toscaTemplates/DODAS-TS/cms_marathon_cluster.yml)
