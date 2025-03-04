---
layout: layout.pug
navigationTitle:  dcos marathon pod show
title: dcos marathon pod show
menuWeight: 27
excerpt: Displaying detailed information for a specific pod
enterprise: false
render: mustache
model: /1.13/data.yml
---


# Description
The `dcos marathon pod show` command allows you to view detailed information for a specific pod.

# Usage

```bash
dcos marathon pod show <pod-id>
```

# Options

| Name |  Description |
|---------|-------------|
| `-h`, `--help` | Display info about usage of this command. |

## Positional arguments

| Name |  Description |
|---------|-------------|
| `<pod-id>`   | The pod ID. You can view a list of the pod IDs with the `dcos marathon pod list` command.|



# Examples

## Show Pod JSON
To see the pod definition, run the following command:
```
dcos marathon pod show <pod-id>
```
You can use the `show` command to read data about the pod programmatically.

# Parent command

| Command | Description |
|---------|-------------|
| [dcos marathon](/1.13/cli/command-reference/dcos-marathon/) | Deploy and manage applications to DC/OS. |
