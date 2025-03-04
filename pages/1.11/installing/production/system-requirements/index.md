---
layout: layout.pug
navigationTitle:  System Requirements
title: System Requirements
menuWeight: 5
enterprise: false
excerpt: Hardware and software requirements for DC/OS  deployments

render: mustache  
---
A DC/OS cluster consists of two types of nodes **master nodes** and **agent nodes**.  The agent nodes can be either **public agent nodes** or **private agent nodes**. Public agent nodes provide north-south (external to internal) access to services in the cluster through load balancers. Private agents host the containers and services that are deployed on the cluster. In addition to the master and agent cluster nodes, each DC/OS installation includes a separate **bootstrap node** for DC/OS installation and upgrade files. Some of the hardware and software requirements apply to all nodes. Other requirements are specific to the type of node being deployed.

# Hardware prerequisites

The hardware prerequisites are a single bootstrap node, Mesos master nodes, and Mesos agent nodes.

## Bootstrap node

*  DC/OS installation is run on a single **Bootstrap node** with two cores, 16 GB RAM, and 60 GB HDD. 
*  The bootstrap node is only used during the installation and upgrade process, so there are no specific recommendations for high performance storage or separated mount points.

<p class="message--note"><strong>NOTE: </strong>The bootstrap node must be separate from your cluster nodes.</p>

<a name="CommonReqs">

## All master and agent nodes in the cluster

The DC/OS cluster nodes are designated Mesos masters and agents during installation. The supported operating systems and environments are listed on the [version policy page](https://docs.mesosphere.com/version-policy/).

When you install DC/OS on the cluster nodes, the required files are installed in the `/opt/mesosphere` directory. You can create the `/opt/mesosphere` directory prior to installing DC/OS, but it must be either an empty directory or a link to an empty directory. DC/OS can be installed on a separate volume mount by creating an empty directory on the mounted volume, creating a link at `/opt/mesosphere` that targets the empty directory, and then installing DC/OS.

You should verify the following requirements for all master and agent nodes in the cluster:
- Every node must have network access to a public Docker repository or to an internal Docker registry.
- If the node operating system is RHEL 7 or CentOS 7, the `firewalld` daemon must be stopped and disabled. For more information, see [Disabling the firewall daemon on Red Hat or CentOS](#FirewallDaemon).
- The DNSmasq process must be stopped and disabled so that DC/OS has access to port 53. For more information, see [Stopping the DNSmasq process](#StopDNSmasq).
- You are not using `noexec` to mount the `/tmp` directory on any system where you intend to use the DC/OS CLI.
-  You have sufficient disk to store persistent information for the cluster in the `var/lib/mesos` directory.
- You should not remotely mount the `/var/lib/mesos` or Docker storage `/var/lib/docker` directory.

<a name="FirewallDaemon">

### Disabling the firewall daemon on Red Hat or CentOS
There is a known <a href="https://github.com/docker/docker/issues/16137" target="_blank">Docker issue</a> that the `firewalld` process interacts poorly with Docker. For more information about this issue, see the <a href="https://docs.docker.com/v1.6/installation/centos/#firewalld" target="_blank">Docker CentOS firewalld</a> documentation.

To stop and disable the `firewalld`, run the following command:
```bash
sudo systemctl stop firewalld && sudo systemctl disable firewalld
```
<a name="StopDNSmasq">

### Stopping the DNSmasq process
The DC/OS cluster requires access to port 53. To prevent port conflicts, you should stop and disable the `dnsmasq` process by running the following command:
```bash
sudo systemctl stop dnsmasq && sudo systemctl disable dnsmasq.service
```

## Master node requirements

The following table lists the master node hardware requirements:

|             | Minimum   | Recommended |
|-------------|-----------|-------------|
| Nodes       | 1*         | 3 or 5     |
| Processor   | 4 cores   | 4 cores     |
| Memory      | 32 GB RAM | 32 GB RAM   |
| Hard disk   | 120 GB    | 120 GB      |
&ast; For business critical deployments, three master nodes are required rather than one master node.

There are many mixed workloads on the masters. Workloads that are expected to be continuously available or considered business-critical should only be run on a DC/OS cluster with at least three masters. For more information about high availability requirements see the [High Availability documentation][0].

[0]: /1.11/overview/high-availability/

Examples of mixed workloads on the masters are Mesos replicated logs and ZooKeeper. In some cases, mixed workloads require synchronizing with `fsync` periodically, which can generate a lot of expensive random I/O. We recommend the following:

- Solid-state drive (SSD) or non-volatile memory express (NVMe) devices for fast, locally-attached storage. To reduce the likelihood of I/O latency issues, solid-state drives should be locally attached to the physical machine, if possible. You should also be sure that solid-state drive (SSD) or non-volatile memory express (NVMe) devices are used for the file systems hosting master node replicated logs.

  In planning your storage requirements, keep in mind that you should avoid using a single storage area network (SAN) device and NFS to connect to the nodes in the cluster. This type of architecture introduces a higher possibility of latency than using local storage and introduces a single point of failure in what should otherwise be a distributed system. Network latency and bandwidth issues can cause client sessions to time out and adversely affect [DC/OS] cluster performance and reliability.

- RAID controllers with a battery backup unit (BBU).
- RAID controller cache configured in writeback mode.
- If separation of storage mount points is possible, the following storage mount points are recommended on the master node. These recommendations will optimize the performance of a busy DC/OS cluster by isolating the I/O of various services.
  | Directory Path | Description |
  |:-------------- | :---------- |
  | _/var/lib/dcos_ | A majority of the I/O on the master nodes will occur within this directory structure. If you are planning a cluster with hundreds of nodes or intend to have a high rate of deploying and deleting workloads, isolating this directory to dedicated SSD storage on a separate device is recommended. |

- Further breaking down this directory structure into individual mount points for specific services is recommended for a cluster which will grow to thousands of nodes.

  | Directory Path | Description |
  |:-------------- | :---------- |
  | _/var/lib/dcos/mesos/master_ | logging directories |
  | _/var/lib/dcos/cockroach_ | CockroachDB [enterprise type="inline" size="small" /] |
  | _/var/lib/dcos/navstar_ | for Mnesia database |
  | _/var/lib/dcos/secrets_ | secrets vault [enterprise type="inline" size="small" /] | 
  | _/var/lib/dcos/exec_ | Temporary files required by various DC/OS services. The _/var/lib/dcos/exec_ directory must not be on a volume which is mounted with the `noexec` option. |
  | _/var/lib/dcos/exhibitor_ | Zookeeper database |
  | _/var/lib/dcos/exhibitor/zookeeper/transactions_ | The ZooKeeper transaction logs are very sensitive to delays in disk writes. If you can only provide limited SSD space, this is the directory to place there. A minimum of 2 GB must be available for these logs. |

## Agent node requirements

The table below shows the agent node hardware requirements.

|             | Minimum   | Recommended |
|-------------|-----------|-------------|
| Nodes       | 1         | 6 or more   |
| Processor   | 2 cores   | 2 cores     |
| Memory      | 16 GB RAM | 16 GB RAM   |
| Hard disk   | 60 GB     | 60 GB       |

In planning memory requirements for agent nodes, you should ensure that agents are configured minimize the use of swap space. The recommended best practice is optimize cluster performance and reduce potential resource consumption issues to disable memory swapping for all agents in the cluster, if possible.

In addition to the requirements described in [All master and agent nodes in the cluster](#CommonReqs), the agent nodes must have:
- A `/var` directory with 20 GB or more of free space. This directory is used by the sandbox for both [Docker and DC/OS Universal container runtime](/1.11/deploying-services/containerizers/).

-   Do not use `noexec` to mount the `/tmp` directory on any system where you intend to use the DC/OS CLI unless a TMPDIR environment variable is set to something other than `/tmp/`. Mounting the `/tmp` directory using the `noexec` option could break CLI functionality. 

-   If you are planning a cluster with hundreds of agent nodes or intend to have a high rate of deploying and deleting services, isolating this directory to dedicated SSD storage is recommended.

    | Directory Path | Description |
    |:-------------- | :---------- |
    | _/var/lib/mesos/_ | Most of the I/O from the Agent nodes will be directed at this directory. Also, The disk space that Apache Mesos advertises in its UI is the sum of the space advertised by filesystem(s) underpinning _/var/lib/mesos_ |

-  Further breaking down this directory structure into individual mount points for specific services is recommended for a cluster which will grow to thousands of nodes.

   | Directory path | Description |
   |:-------------- |:----------- |
   | _/var/lib/mesos/slave/slaves_ | Sandbox directories for tasks |
   | _/var/lib/mesos/slave/volumes_ | Used by frameworks that consume ROOT persistent volumes |
   | _/var/lib/mesos/docker/store_ | Stores Docker image layers that are used to provision URC containers |
   | _/var/lib/docker_ | Stores Docker image layers that are used to provision Docker containers |

## <a name="port-and-protocol"></a>Port and protocol configuration

-   Secure shell (SSH) must be enabled on all nodes.
-   Internet Control Message Protocol (ICMP) must be enabled on all nodes.
-   All hostnames (FQDN and short hostnames) must be resolvable in DNS; both forward and reverse lookups must succeed. [enterprise type="inline" size="small" /]
-   All DC/OS node host names should resolve to  **locally bindable** IP addresses. Most applications require host names to resolve by binding to a local IP address to function correctly. Applications that cannot resolve the host name of a node by binding to a local IP address might fail to function or behave in unexpected ways. [enterprise type="inline" size="small" /]
-   Each node is network accessible from the bootstrap node.
-   Each node has unfettered IP-to-IP connectivity from itself to all nodes in the DC/OS cluster.
-   All ports should be open for communication from the master nodes to the agent nodes and vice versa. [enterprise type="inline" size="small" /]
-   UDP must be open for ingress to port 53 on the masters. To attach to a cluster, the Mesos agent node service (`dcos-mesos-slave`) uses this port to find `leader.mesos`. 

Requirements for intermediaries (e.g., reverse proxies performing SSL termination) between DC/OS users and the master nodes:

- No intermediary must buffer the entire response before sending any data to the client.
- Upon detecting that its client goes away, the intermediary should also close the corresponding upstream TCP connection (i.e., the intermediary
should not reuse upstream HTTP connections).

## High-speed internet access

High speed internet access is recommended for DC/OS installations. A minimum 10 MBit per second is required for DC/OS services. The installation of some DC/OS services will fail if the artifact download time exceeds the value of MESOS_EXECUTOR_REGISTRATION_TIMEOUT within the file `/opt/mesosphere/etc/mesos-slave-common`. The default value for MESOS_EXECUTOR_REGISTRATION_TIMEOUT is 10 minutes.

# Software prerequisites

* Refer to the [install_prereqs.sh](https://raw.githubusercontent.com/dcos/dcos/1.10/cloud_images/centos7/install_prereqs.sh) script for an example of how to install the software requirements for DC/OS masters and agents on a CentOS 7 host.[enterprise type="inline" size="small" /]

* When using OverlayFS over XFS, the XFS volume should be created with the -n ftype=1 flag. Please see the [Red Hat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/7.2_release_notes/technology-preview-file_systems) and [Mesos](http://mesos.apache.org/documentation/latest/container-image/#provisioner-backends) documentation for more details.

## Docker requirements

Docker must be installed on all bootstrap and cluster nodes. The supported Docker versions are listed on [version policy page](https://docs.mesosphere.com/version-policy/).

### Recommendations

- Do not use Docker `devicemapper` storage driver in `loop-lvm` mode. For more information, see [Docker and the Device Mapper storage driver](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/).

- Prefer `OverlayFS` or `devicemapper` in `direct-lvm` mode when choosing a production storage driver. For more information, see Docker's <a href="https://docs.docker.com/engine/userguide/storagedriver/selectadriver/" target="_blank">Select a Storage Driver</a>.

- Manage Docker on CentOS with `systemd`. The `systemd` handles will start Docker and helps to restart Dcoker, when it crashes.

- Run Docker commands as the root user (with `sudo`) or as a user in the <a href="https://docs.docker.com/engine/installation/linux/centos/#create-a-docker-group" target="_blank">docker user group</a>.

### Distribution-specific installation

Each Linux distribution requires Docker to be installed in a specific way:

- **CentOS/RHEL** - [Install Docker from Docker's yum repository][1].
- **CoreOS** - Comes with Docker pre-installed and pre-configured.

For more more information, see Docker's <a href="https://docs.docker.com/install/" target="_blank">distribution-specific installation instructions</a>.

## Disable sudo password prompts

To disable the `sudo` password prompt, you must add the following line to your `/etc/sudoers` file.

```bash
%wheel ALL=(ALL) NOPASSWD: ALL
```

Alternatively, you can SSH as the `root` user.

## Synchronize time for all nodes in the cluster

You must enable Network Time Protocol (NTP) on all nodes in the cluster for clock synchronization. By default, during DC/OS startup you will receive an error if this is not enabled. You can check if NTP is enabled by running one of these commands, depending on your OS and configuration:

```bash
ntptime
adjtimex -p
timedatectl
```

## Bootstrap node

Before installing DC/OS, you **must** ensure that your bootstrap node has the following prerequisites.

<p class="message--important"><strong>IMPORTANT: </strong>If you specify `exhibitor_storage_backend: zookeeper`, the bootstrap node is a permanent part of your cluster. With `exhibitor_storage_backend: zookeeper`, the leader state and leader election of your Mesos masters is maintained in Exhibitor ZooKeeper on the bootstrap node. For more information, see the <a href="/1.11/installing/production/advanced-configuration/configuration-reference/">configuration parameter documentation</a>.</p>


- The bootstrap node must be separate from your cluster nodes.

### <a name="setup-file"></a>DC/OS configuration file

- Download and save the [dcos_generate_config file](https://support.mesosphere.com/hc/en-us/articles/213198586-Mesosphere-Enterprise-DC-OS-Downloads) to your bootstrap node. This file is used to create your customized DC/OS build file. Contact your sales representative or <a href="mailto:sales@mesosphere.com">sales@mesosphere.com</a> for access to this file. [enterprise type="inline" size="small" /]

- Download and save the [dcos_generate_config file](https://downloads.dcos.io/dcos/stable/dcos_generate_config.sh) to your bootstrap node. This file is used to create your customized DC/OS build file. [oss type="inline" size="small" /]

### Docker NGINX (production installation)

For production installations only, install the Docker NGINX image with this command:

```bash
sudo docker pull nginx
```

## Cluster nodes

For production installations only, your cluster nodes must have the following prerequisites. The cluster nodes are designated as Mesos masters and agents during installation.

### Data compression (production installation)

You must have the <a href="http://www.info-zip.org/UnZip.html" target="_blank">UnZip</a>, <a href="https://www.gnu.org/software/tar/" target="_blank">GNU tar</a>, and <a href="http://tukaani.org/xz/" target="_blank">XZ Utils</a> data compression utilities installed on your cluster nodes.

To install these utilities on CentOS7 and RHEL7:

```bash
sudo yum install -y tar xz unzip curl ipset
```

### Cluster permissions (production installation)

On each of your cluster nodes, follow the below instructions:

*   Make sure that SELinux is in one of the supported modes.

    To review the current SELinux status and configuration run the following command:

    ```bash
    sudo sestatus
    ```

    DC/OS supports the following SELinux configurations:

    * Current mode: `disabled`
    * Current mode: `permissive`
    * Current mode: `enforcing`, given that `Loaded policy name` is `targeted`
      This mode is not supported on CoreOS.

    To change the mode from `enforcing` to `permissive` run the following command:

    ```bash
    sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
    ```

    Or, if `sestatus` shows a "Current mode" which is `enforcing` with a `Loaded policy name` which is not `targeted`, run the following command to change the `Loaded policy name` to `targeted`:

    ```bash
    sudo sed -i 's/SELINUXTYPE=.*/SELINUXTYPE=targeted/g' /etc/selinux/config
    ```

    <p class="message--note"><strong>NOTE: </strong>Ensure that all services running on every node can be run in the chosen SELinux configuration.</p>

*   Add `nogroup` and `docker` groups:

    ```bash
    sudo groupadd nogroup &&
    sudo groupadd docker
    ```

*   Reboot your cluster for the changes to take effect.

    ```bash
    sudo reboot
    ```

    <p class="message--note"><strong>NOTE: </strong>It may take a few minutes for your node to come back online after reboot.</p>

### Locale requirements
You must set the `LC_ALL` and `LANG` environment variables to `en_US.utf-8`.

- For information on how to set these variables in Red Hat, see [How to change system locale on RHEL](https://access.redhat.com/solutions/974273)

- On Linux:
````
localectl set-locale LANG=en_US.utf8
````

- For information on how to set on these variables in CentOS7, see [How to set up system locale on CentOS 7](https://www.rosehosting.com/blog/how-to-set-up-system-locale-on-centos-7/).

# Next steps
- [Install Docker from Docker’s yum repository][1]
- [DC/OS Installation Guide][2]

[1]: /1.11/installing/production/system-requirements/docker-centos/

[2]: /1.11/installing/production/deploying-dcos/installation/