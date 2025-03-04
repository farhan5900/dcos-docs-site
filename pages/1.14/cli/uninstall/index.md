---
layout: layout.pug
navigationTitle:  Uninstalling the CLI
title: Uninstalling the CLI
menuWeight: 4
excerpt: Uninstalling the DC/OS command line interface
enterprise: false
render: mustache
model: /1.14/data.yml
---

You can uninstall the CLI with these commands for your operating system.

- [Unix, Linux, and macOS](#unixlinuxosx)
- [Windows](#windows)

## <a name="unixlinuxosx"></a>Unix, Linux, and macOS

1.  Delete the hidden `.dcos` directory. This will delete the configuration files for your DC/OS CLI connections.

    ```bash
    rm -rf ~/.dcos
    ```

1.  Delete the `dcos` executable.  By default, this file is located in `/usr/local/bin/dcos`.

    ```bash
    rm -rf /usr/local/bin/dcos
    ```

## <a name="windows"></a>Windows

1.  Delete the hidden `.dcos` directory. This will delete the configuration files for your DC/OS CLI connections.
1.  Delete the `dcos` executable.  
