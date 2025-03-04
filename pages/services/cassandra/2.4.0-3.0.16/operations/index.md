---
layout: layout.pug
navigationTitle: Operations
excerpt: Managing, repairing, and monitoring your DC/OS Apache Cassandra service
title: Operations
menuWeight: 40
model: /services/cassandra/data.yml
render: mustache
---

#include /services/include/operations.tmpl

## Performing {{ model.techShortName }} Cleanup and Repair Operations

You may manually trigger certain `nodetool` operations against your {{ model.techShortName }} instance using the CLI or the HTTP API.

### Cleanup

You may trigger a `nodetool cleanup` operation across your {{ model.techShortName }} nodes using the `cleanup` plan. This plan requires the following parameters to run:
- `CASSANDRA_KEYSPACE`: the {{ model.techShortName }} keyspace to be cleaned up.

To initiate this plan from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan start cleanup -p CASSANDRA_KEYSPACE=space1
```

To view the status of this plan from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan status cleanup
cleanup (IN_PROGRESS)
└─ cleanup-deploy (IN_PROGRESS)
   ├─ node-0:[cleanup] (COMPLETE)
   ├─ node-1:[cleanup] (STARTING)
   └─ node-2:[cleanup] (PENDING)
```

When the plan is completed, its status will be `COMPLETE`.

The above `plan start` and `plan status` commands may also be made directly to the service over HTTP. To see the queries involved, run the above commands with an additional `-v` flag.

For more information about `nodetool cleanup`, see the {{ model.techShortName }} documentation.

### Repair

You may trigger a `nodetool repair` operation across your {{ model.techShortName }} nodes using the `repair` plan. This plan requires the following parameters to run:
- `CASSANDRA_KEYSPACE`: the {{ model.techShortName }} keyspace to be cleaned up.

To initiate this command from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan start repair -p CASSANDRA_KEYSPACE=space1
```

To view the status of this plan from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan status repair
repair (STARTING)
└─ repair-deploy (STARTING)
   ├─ node-0:[repair] (STARTING)
   ├─ node-1:[repair] (PENDING)
   └─ node-2:[repair] (PENDING)
```

When the plan is completed, its status will be `COMPLETE`.

The above `plan start` and `plan status` commands may also be made directly to the service over HTTP. To see the queries involved, run the above commands with an additional `-v` flag.

For more information about `nodetool repair`, see the {{ model.techShortName }} documentation.

## Seed nodes

{{ model.techShortName }} seed nodes are those nodes with indices smaller than the seed node count.  By default, {{ model.techShortName }} is deployed
with a seed node count of two (node-0 and node-1 are seed nodes). When a replace operation is performed on one these
nodes, all other nodes must be restarted to be brought up to date regarding the ip address of the new seed node. This
operation is performed automatically.

For example if `node-0` needed to be replaced you would execute:

```bash
dcos {{ model.packageName }} --name=<service-name> pod replace node-0
```

which would result in a recovery plan like the following:

```bash
$ dcos {{ model.packageName }} --name=<service-name> plan show recovery
recovery (IN_PROGRESS)
└─ permanent-node-failure-recovery (IN_PROGRESS)
   ├─ node-0:[server] (COMPLETE)
   ├─ node-1:[server] (STARTING)
   └─ node-2:[server] (PENDING)
   ...
```

<p class="message--note"><strong>NOTE: </strong>Only the seed node is being placed on a new node, all other nodes are restarted in place with no loss of data.</p>


# Back up and Restore

## Backing Up to S3

You can back up an entire cluster's data and schema to Amazon S3 or S3 compatible storage using the `backup-s3` plan. This plan requires the following parameters to run:
- `SNAPSHOT_NAME`: the name of this snapshot. Snapshots for individual nodes will be stored as S3 folders inside of a top level `snapshot` folder.
- `CASSANDRA_KEYSPACES`: the {{ model.techShortName }} keyspaces to back up. The entire keyspace, as well as its schema, will be backed up for each keyspace specified.
- `AWS_ACCESS_KEY_ID`: the access key ID for the AWS IAM user running this backup
- `AWS_SECRET_ACCESS_KEY`: the secret access key for the AWS IAM user running this backup
- `AWS_REGION`: the region of the S3 bucket being used to store this backup
- `AWS_SESSION_TOKEN`: needed if you’re including temporary security credentials in the file
- `S3_BUCKET_NAME`: the name of the S3 bucket in which to store this backup
- `HTTPS_PROXY`: specifications for the {{ model.TechName }} backup plan, taken from `config.yaml`.

Optional parameters:
- `S3_ENDPOINT_URL`: URL of S3 compatible storage. If you want to backup to S3 compatible storage, please provide this parameter in `backup-s3` plan.
- `AWS_SESSION_ID`: It may also be necessary to set the `AWS_SESSION_ID` depending on how you authenticate with AWS.
- `AWS_SESSION_TOKEN`: It may also be necessary to set the `AWS_SESSION_TOKEN` depending on how you authenticate with AWS. 

Make sure that you provision your nodes with enough disk space to perform a backup. {{ model.TechName }} backups are stored on disk before being uploaded to S3, and will take up as much space as the data currently in the tables, so you will need half of your total available space to be free to back up every keyspace at once.

As noted in the documentation for the backup/restore strategy configuration option, it is possible to run transfers to S3 either in serial or in parallel, but care must be taken not to exceed any throughput limits you may have in your cluster. Throughput depends on a variety of factors, including uplink speed, proximity to region where the backups are being uploaded and downloaded, and the performance of the underlying storage infrastructure. You should perform periodic tests in your local environment to understand what you can expect from S3.

You can configure whether snapshots are created and uploaded in serial (default) or in parallel. The serial backup/restore strategy is recommended.

You can initiate this plan from the command line:
```
SNAPSHOT_NAME=<my_snapshot>
CASSANDRA_KEYSPACES="space1 space2"
AWS_ACCESS_KEY_ID=<my_access_key_id>
AWS_SECRET_ACCESS_KEY=<my_secret_access_key>
AWS_REGION=us-west-2
AWS_SESSION_TOKEN=AQoDYXdzEJr...<remainder of security token>
S3_BUCKET_NAME=backups
dcos {{ model.packageName }} --name=<service-name> plan start backup-s3 \
    -p SNAPSHOT_NAME=$SNAPSHOT_NAME \
    -p "CASSANDRA_KEYSPACES=$CASSANDRA_KEYSPACES" \
    -p AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
    -p AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
    -p AWS_REGION=$AWS_REGION \
    -p S3_BUCKET_NAME=$S3_BUCKET_NAME
    -p HTTPS_PROXY=http://internal.proxy:8080
```

If you are backing up multiple keyspaces, they must be separated by spaces and wrapped in quotation marks when supplied to the `plan start` command, as in the example above. If the `CASSANDRA_KEYSPACES` parameter isn't supplied, then every keyspace in your cluster will be backed up.

<p class="message--warning"><strong>WARNING: </strong>To ensure that sensitive information such as your AWS secret access key remains secure, make sure that you've set the <tt>core.dcos_url</tt> configuration property in the DC/OS CLI to an HTTPS URL.</p>

To view the status of this plan from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan status backup-s3
backup-s3 (IN_PROGRESS)
├─ backup-schema (COMPLETE)
│  ├─ node-0:[backup-schema] (COMPLETE)
│  ├─ node-1:[backup-schema] (COMPLETE)
│  └─ node-2:[backup-schema] (COMPLETE)
├─ create-snapshots (IN_PROGRESS)
│  ├─ node-0:[snapshot] (STARTED)
│  ├─ node-1:[snapshot] (STARTED)
│  └─ node-2:[snapshot] (COMPLETE)
├─ upload-backups (PENDING)
│  ├─ node-0:[upload-s3] (PENDING)
│  ├─ node-1:[upload-s3] (PENDING)
│  └─ node-2:[upload-s3] (PENDING)
└─ cleanup-snapshots (PENDING)
   ├─ node-0:[cleanup-snapshot] (PENDING)
   ├─ node-1:[cleanup-snapshot] (PENDING)
   └─ node-2:[cleanup-snapshot] (PENDING)
```

The above `plan start` and `plan status` commands may also be made directly to the service over HTTP. To see the queries involved, run the above commands with an additional `-v` flag.

## Backing up to Azure

You can also back up to Microsoft Azure using the `backup-azure` plan. This plan requires the following parameters to run:

- `SNAPSHOT_NAME`: the name of this snapshot. Snapshots for individual nodes will be stored as gzipped tarballs with the name `node-<POD_INDEX>.tar.gz`.
- `CASSANDRA_KEYSPACES`: the {{ model.techShortName }} keyspaces to backup. The entire keyspace, as well as its schema, will be backed up for each keyspace specified.
- `CLIENT_ID`: the client ID for the Azure service principal running this backup
- `TENANT_ID`: the tenant ID for the tenant that the service principal belongs to
- `CLIENT_SECRET`: the service principal's secret key
- `AZURE_STORAGE_ACCOUNT`: the name of the storage account that this backup will be sent to
- `AZURE_STORAGE_KEY`: the secret key associated with the storage account
- `CONTAINER_NAME`: the name of the container in which to store this backup.

You can initiate this plan from the command line in the same way as the Amazon S3 backup plan:
```
dcos {{ model.packageName }} --name=<service-name> plan start backup-azure \
    -p SNAPSHOT_NAME=$SNAPSHOT_NAME \
    -p "CASSANDRA_KEYSPACES=$CASSANDRA_KEYSPACES" \
    -p CLIENT_ID=$CLIENT_ID \
    -p TENANT_ID=$TENANT_ID \
    -p CLIENT_SECRET=$CLIENT_SECRET \
    -p AZURE_STORAGE_ACCOUNT=$AZURE_STORAGE_ACCOUNT \
    -p AZURE_STORAGE_KEY=$AZURE_STORAGE_KEY \
    -p CONTAINER_NAME=$CONTAINER_NAME
```

To view the status of this plan from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan status backup-azure
backup-azure (IN_PROGRESS)
├─ backup-schema (COMPLETE)
│  ├─ node-0:[backup-schema] (COMPLETE)
│  ├─ node-1:[backup-schema] (COMPLETE)
│  └─ node-2:[backup-schema] (COMPLETE)
├─ create-snapshots (COMPLETE)
│  ├─ node-0:[snapshot] (COMPLETE)
│  ├─ node-1:[snapshot] (COMPLETE)
│  └─ node-2:[snapshot] (COMPLETE)
├─ upload-backups (IN_PROGRESS)
│  ├─ node-0:[upload-azure] (COMPLETE)
│  ├─ node-1:[upload-azure] (STARTING)
│  └─ node-2:[upload-azure] (PENDING)
└─ cleanup-snapshots (PENDING)
   ├─ node-0:[cleanup-snapshot] (PENDING)
   ├─ node-1:[cleanup-snapshot] (PENDING)
   └─ node-2:[cleanup-snapshot] (PENDING)
```

The above `plan start` and `plan status` commands may also be made directly to the service over HTTP. To see the queries involved, run the above commands with an additional `-v` flag.

# Restore

All restore plans will restore the schema from every keyspace backed up with the backup plan and populate those keyspaces with the data they contained at the time the snapshot was taken. Downloading and restoration of backups will use the configured backup/restore strategy. This plan assumes that the keyspaces being restored do not already exist in the current cluster, and will fail if any keyspace with the same name is present.

## Restoring From S3

Restoring cluster data is similar to backing it up. The `restore-s3` plan assumes that your data is stored in an S3 bucket in the format that `backup-s3` uses. The restore plan has the following required parameters:
- `SNAPSHOT_NAME`: the snapshot name from the `backup-s3` plan
- `AWS_ACCESS_KEY_ID`: the access key ID for the AWS IAM user running this restore
- `AWS_SECRET_ACCESS_KEY`: the secret access key for the AWS IAM user running this restore
- `AWS_REGION`: the region of the S3 bucket being used to store the backup being restored
- `S3_BUCKET_NAME`: the name of the S3 bucket where the backup is stored

Optional parameters:
- `S3_ENDPOINT_URL`: URL of S3 compatible storage. If you want to restore from S3 compatible storage, please provide this parameter in `restore-s3` plan.
- `AWS_SESSION_ID`: It may also be necessary to set the `AWS_SESSION_ID` depending on how you authenticate with AWS.
- `AWS_SESSION_TOKEN`: It may also be necessary to set the `AWS_SESSION_TOKEN` depending on how you authenticate with AWS. 

To initiate this plan from the command line:
```
SNAPSHOT_NAME=<my_snapshot>
CASSANDRA_KEYSPACES="space1 space2"
AWS_ACCESS_KEY_ID=<my_access_key_id>
AWS_SECRET_ACCESS_KEY=<my_secret_access_key>
AWS_REGION=us-west-2
S3_BUCKET_NAME=backups
dcos {{ model.packageName }} --name=<service-name> plan start restore-s3 \
    -p SNAPSHOT_NAME=$SNAPSHOT_NAME \
    -p "CASSANDRA_KEYSPACES=$CASSANDRA_KEYSPACES" \
    -p AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
    -p AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
    -p AWS_REGION=$AWS_REGION \
    -p S3_BUCKET_NAME=$S3_BUCKET_NAME
```

To view the status of this plan from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan status restore-s3
restore-s3 (IN_PROGRESS)
├─ fetch-s3 (COMPLETE)
│  ├─ node-0:[fetch-s3] (COMPLETE)
│  ├─ node-1:[fetch-s3] (COMPLETE)
│  └─ node-2:[fetch-s3] (COMPLETE)
├─ restore-schema (IN_PROGRESS)
│  ├─ node-0:[restore-schema] (COMPLETE)
│  ├─ node-1:[restore-schema] (STARTED)
│  └─ node-2:[restore-schema] (PENDING)
└─ restore-snapshots (PENDING)
   ├─ node-0:[restore-snapshot] (PENDING)
   ├─ node-1:[restore-snapshot] (PENDING)
   └─ node-2:[restore-snapshot] (PENDING)
```

The above `plan start` and `plan status` commands may also be made directly to the service over HTTP. To see the queries involved, run the above commands with an additional `-v` flag.

## Restoring From Azure

You can restore from Microsoft Azure using the `restore-azure` plan. This plan requires the following parameters to run:

- `SNAPSHOT_NAME`: the name of this snapshot. Snapshots for individual nodes will be stored as gzipped tarballs with the name `node-<POD_INDEX>.tar.gz`.
- `CLIENT_ID`: the client ID for the Azure service principal running this backup
- `TENANT_ID`: the tenant ID for the tenant that the service principal belongs to
- `CLIENT_SECRET`: the service principal's secret key
- `AZURE_STORAGE_ACCOUNT`: the name of the storage account that this backup will be sent to
- `AZURE_STORAGE_KEY`: the secret key associated with the storage account
- `CONTAINER_NAME`: the name of the container in whcih to store this backup

You can initiate this plan from the command line in the same way as the Amazon S3 restore plan:
```
dcos {{ model.packageName }} --name=<service-name> plan start restore-azure \
    -p SNAPSHOT_NAME=$SNAPSHOT_NAME \
    -p CLIENT_ID=$CLIENT_ID \
    -p TENANT_ID=$TENANT_ID \
    -p CLIENT_SECRET=$CLIENT_SECRET \
    -p AZURE_STORAGE_ACCOUNT=$AZURE_STORAGE_ACCOUNT \
    -p AZURE_STORAGE_KEY=$AZURE_STORAGE_KEY \
    -p CONTAINER_NAME=$CONTAINER_NAME
```

To view the status of this plan from the command line:
```
dcos {{ model.packageName }} --name=<service-name> plan status restore-azure
restore-azure (IN_PROGRESS)
├─ fetch-azure (COMPLETE)
│  ├─ node-0:[fetch-azure] (COMPLETE)
│  ├─ node-1:[fetch-azure] (COMPLETE)
│  └─ node-2:[fetch-azure] (COMPLETE)
├─ restore-schema (COMPLETE)
│  ├─ node-0:[restore-schema] (COMPLETE)
│  ├─ node-1:[restore-schema] (COMPLETE)
│  └─ node-2:[restore-schema] (COMPLETE)
└─ restore-snapshots (IN_PROGRESS)
   ├─ node-0:[restore-snapshot] (COMPLETE)
   ├─ node-1:[restore-snapshot] (STARTING)
   └─ node-2:[restore-snapshot] (PENDING)
```

The above `plan start` and `plan status` commands may also be made directly to the service over HTTP. To see the queries involved, run the above commands with an additional `-v` flag.
