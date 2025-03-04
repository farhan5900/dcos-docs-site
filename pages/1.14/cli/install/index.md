---
layout: layout.pug
navigationTitle:  Installing the CLI
title: Installing the CLI
menuWeight: 1
excerpt: Installing the DC/OS command line interface
enterprise: false
render: mustache
model: /1.14/data.yml
---

The recommended method to install the DC/OS CLI is by getting the preformatted set of commands from the DC/OS UI and running them in the terminal. See the prerequisites and instructions for your operating system for more information:

- [Installing on Linux](#linux)
- [Installing on macOS](#macos)
- [Installing on Windows](#windows)

# Prerequisites

## Common prerequisites
- You must have a **separate computer** that is not part of the DC/OS cluster on which you can install the CLI.
- You must have network access from the external system hosting the CLI to the DC/OS cluster.
- You must be able to open a command-line shell terminal on the external system hosting the CLI.
- You must not be using the `noexec` to mount the `/tmp` directory unless you have set a `TMPDIR` environment variable to something other than the `/tmp` directory. Mounting the `/tmp` directory with the `noexec` option can prevent CLI operations.

# Prerequisites for Linux
- You must be able to run `cURL` program on the system hosting the CLI. The `curl` command is installed by default on most Linux distributions.


# Prerequisites for macOS
-  You must be running MacOS 10.10 Yosemite (deprecated), or later. The next version of the DC/OS CLI will require macOS 10.11 El Capitan or later.
- A model running on a Haswell CPU (2014), or later.
- You must be able to run `cURL` program on the system hosting the CLI. If you don't have `cURL`, follow the instructions in [Install curl on macOS](http://macappstore.org/curl/) to install it.

# Prerequisites for Windows
- Windows 10 64-bit or later.
- You must disable any security or antivirus software before you start the installation.

# Installing the CLI using the UI

1. At the top-right corner of the DC/OS UI, click the down arrow to the right of your cluster name.

    ![open cluster popup](/1.14/img/open-cluster-popup.png)

    Figure 1. Open cluster popup menu

1. Select **Install CLI**.

    ![CLI install UI](/1.14/img/install-cli.png)

    Figure 2. Select Install CLI

1. Copy and paste the code snippets appropriate to your OS into your terminal, and press the return key.

1. Once the dcos information screen has been displayed, run the command `dcos cluster list` to verify the connection to the cluster has been established.

<a name="linux"></a>

# Installing the DC/OS CLI manually on Linux

It is strongly recommended that you copy and paste the installation commands from the UI of the cluster to which you want to connect. Following below are the instructions for step by step installation of the CLI.

1. If you do not already have a working directory for the CLI, create one. The preferred location is `/usr/local/bin` and all the instructions will reference this path.

    ```bash
    [ -d usr/local/bin ] || sudo mkdir -p /usr/local/bin
    ```

1. Download the DC/OS CLI binary to your local directory by running the following command and replacing `<target-os-type>` with the OS type (`darwin`, `linux`, `windows`), and `<dcos-version>` with the version (such as 1.13), that you want to use.

    ```bash
    curl https://downloads.dcos.io/binaries/cli/<target-os-type>/x86-64/dcos-<dcos-version>/dcos -o dcos
    ```

    For example, the CLI download for a Linux user on DC/OS 1.13 would look like this:

    ```bash
    curl https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.13/dcos -o dcos
    ```

1.  Move the CLI binary to your local bin directory.

    ```bash
    sudo mv dcos /usr/local/bin
    ```

1. Make the CLI binary executable.

    ```bash
    chmod +x /usr/local/bin/dcos
    ```

1. Set up the connection from the CLI to your DC/OS cluster. In this example, `http://example.com` is the master node URL.

    ```bash
    dcos cluster setup http://example.com
    ```

    Follow the instructions in the DC/OS CLI. For more information about security, see [Security](/1.14/security/).

    Your CLI should now be authenticated with your cluster! Enter `dcos` to get started. You can learn more about managing your cluster connections [here](/1.14/cli/command-reference/dcos-cluster/).

<a name="macos"></a>

# Installing the DC/OS CLI manually on macOS

**The first two steps of this tutorial can be replaced by a simple `brew install dcos-cli` if you have [Homebrew](https://brew.sh) installed.**

1. Download the DC/OS CLI binary `dcos` to your working directory by running the following command and replacing `<target-os-type>` with the OS type (`darwin`, `linux`, `windows`), and `<dcos-version>` with the version (such as 1.13), that you want to use:

    ```bash
    curl https://downloads.dcos.io/binaries/cli/<target-os-type>/x86-64/dcos-<dcos-version>/dcos -o dcos
    ```

    For example, the CLI download for a Mac user on DC/OS 1.13 would look like this:

    ```bash
    curl https://downloads.dcos.io/binaries/cli/darwin/x86-64/dcos-1.13/dcos -o dcos
    ```

1.  Make the CLI binary executable.

    ```bash
    chmod +x dcos
    ```

1. Set up the connection from the CLI to your DC/OS cluster. In this example, `http://example.com` is the master node URL.

    ```bash
    dcos cluster setup http://example.com
    ```
    If your system is unable to find the executable, you may need to re-open the command prompt or add the installation directory to your PATH environment variable manually.</p>

    Follow the instructions in the DC/OS CLI. For more information about security, see the [documentation](/1.14/security/).

    Your CLI should now be authenticated with your cluster!

1. Type `dcos` to view usage information and get started.

<a name="windows"></a>

# Installing the DC/OS CLI manually on Windows

1. Open the command line environment using the Administrator credentials.

1. Download the DC/OS CLI binary `dcos` to your working directory by running the following command and replacing `<target-os-type>` with the OS type (`darwin`, `linux`, `windows`), and `<dcos-version>` with the version (such as 1.13), that you want to use:

    ```bash
    curl https://downloads.dcos.io/binaries/cli/<target-os-type>/x86-64/dcos-<dcos-version>/dcos -o dcos
    ```

    For example, the CLI download for a Windows user on DC/OS 1.13 would look like this:

    ```bash
    curl https://downloads.dcos.io/binaries/cli/windows/x86-64/dcos-1.13/dcos.exe -o dcos
    ```

1. Change into the directory of the downloaded file if you are not already there.

    ```bash
    cd path/to/download/directory
    ```

1. Set up the connection from the CLI to your DC/OS cluster. In this example, `http://example.com` is the master node URL.

    ```bash
    dcos cluster setup http://example.com
    ```

    <p class="message--note"><strong>NOTE: </strong>If your system is unable to find the executable, you may need to re-open the command prompt or add the installation directory to your PATH environment variable manually.</p>

    Follow the instructions in the DC/OS CLI. For more information about security, see the [documentation](/1.14/security/).

    Your CLI should now be authenticated with your cluster! Enter `dcos` to get started.

