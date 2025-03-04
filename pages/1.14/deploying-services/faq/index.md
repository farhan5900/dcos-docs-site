---
layout: layout.pug
navigationTitle:  Frequently Asked Questions
title: Frequently Asked Questions
menuWeight: 120
excerpt: Frequently asked questions about deploying Marathon services
render: mustache
model: /1.14/data.yml
enterprise: false
---


We have collected some questions we often encounter concerning the usage of DC/OS. Do you have a new question you would like answered, or do you have the answer to a question? Use the `Contribute` button at the top of this page to suggest it, or check out how you can [contribute](https://dcos.io/contribute/) the answer to it.

## Why is my Marathon app stuck in Waiting?

This most commonly occurs when an application being launched has higher system requirements than any of the available offers coming to Marathon via Mesos. The deployment will eventually fail; check system requirements and increase the resources to the application if you want the deployment to succeed.

## Why is my Marathon app launching on a private agent instead of a public agent?

By default apps are launched on private nodes. For more information, see the [documentation][5].

## What is meant by service ports in a Marathon app?

A service port is the globally-unique port assigned to the app when using automatic app discovery through a proxy or load balancer system, as described in [Service Discovery and Load Balancing][1].

## Why can't I start more tasks? I have free resources in my cluster.

The most common causes for this are requesting ports or resource roles that are not available in the cluster. Tasks cannot launch unless they find agents with the required port available, and they will not accept offers that do not contain their accepted resource roles.

## How can I add automatically more agents to the cluster?

DC/OS cannot automatically spin up new nodes in response to load on hardware unless the cloud provider autoscaling groups have been configured with standby hosts and `dcos_install.sh` placed on the standby node. This is an involved process that requires setting up Autoscaling groups with your Cloud Provider (AWS, GCE, Azure) and placing an install file on each node. An overview is available [here](/1.14/deploying-services/scale-service/). Please reach out to Mesosphere Support for more guidance if you need to set this up.

## What is your best practice for service discovery?

A comprehensive overview of a few common service discovery implementations is available at [Service Discovery][2] in Marathon.

## Is it possible to span my cluster over different cloud providers?

This is not currently supported. 

## How can I upload files to Spark driver/executor?

Here is the example of a command you should launch to make it work:

    ```bash
    dcos spark run --submit-args='--conf spark.mesos.uris=https://path/to/pi.conf --class JavaSparkPiConf https://path/to/sparkPi_without_config_file.jar /mnt/mesos/sandbox/pi.conf'
    ```

More info:

- `--conf spark.mesos.uris=...` A comma-separated list of URIs to be downloaded to the sandbox when driver or executor is launched by Mesos. This applies to both coarse-grained and fine-grained mode.

- `/mnt/mesos/sandbox/pi.conf` A path to the downloaded file which your main class receives as a 0th parameter (see the code snippet below). /mnt/mesos/sandbox/ is a standard path inside a container which is mapped to a corespondent mesos-task sandbox.

## How does the installer work?

DC/OS is installed in your environment using a dynamically generated setup file. This file is generated using specific parameters that are set during configuration. This installation file contains a Bash install script and a Docker container that is loaded with everything you need to deploy a customized DC/OS build.

For more information, see the installation [documentation](/1.14/installing/).

## What versions of kernel, local OS, Docker Engine, Union Mount are recommended?

We recommend using CoreOS, matched with its correct versions and sensible defaults of Docker, filesystem, and other settings.

[1]: /1.14/networking/load-balancing-vips/
[2]: /1.14/networking/
[4]: https://support.mesosphere.com/hc/en-us/articles/206474745-How-to-reserve-resources-for-certain-frameworks-in-Mesos-cluster-
[5]: /1.14/administering-clusters/convert-agent-type/
