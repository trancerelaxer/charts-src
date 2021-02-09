# KeyDB

[KeyDB](https://keydb.dev/) is an advanced key-value cache and store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets, sorted sets, bitmaps and hyperloglogs.

## TL;DR;

```bash
$ helm repo add trancerelaxer https://trancerelaxer.github.io/charts

$ helm install sb trancerelaxer/keydb-cluster

## Or locally just
$ helm install sb ./keydb-cluster
```

## Introduction

This chart bootstraps a [KeyDB](https://keydb.dev/) highly available distributed cluster in a [Kubernetes](http://kubernetes.io) cluster using the Helm package manager.

## Prerequisites

- Kubernetes 1.8+ with Beta APIs enabled
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart

```bash
$ helm repo add trancerelaxer https://trancerelaxer.github.io/charts
$ helm install trancerelaxer/keydb-cluster
```

The command deploys KeyDB Cluster on the Kubernetes cluster in the default configuration. By default this chart install 3 shards and one replica for each shard. The [configuration](#configuration) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the deployment:

```bash
$ helm delete <chart-name>
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Note

By default this chart install 3 shards and one replica for each shard:

- In this chart, one statefulset means one shard (include master and replicas), the statefulset would be dynamic created when shard number changes.
- The shard number must be equal or greater than 3, or the cluter will create failed.
- By now, the chart support scale-out (add shards), but the data in cluster won't resharding automatically. You must reshard the data manually.
- If you want to scale-in the cluster, you must reshard the data and remove the node from cluster carefully first, then run `helm upgrade` command to decrease the shard number.

## Configuration

The following table lists the configurable parameters of the KeyDB chart and their default values.

| Parameter                       | Description                                                                           | Default                 |
| :------------------------------ | :------------------------------------------------------------------------------------ | :---------------------- | --- | ------- |
| `image.name`                    | KeyDB image                                                                           | `eqalpha/keydb`         |
| `image.tag`                     | KeyDB image tag                                                                       | `alpine_x86_64_v6.0.16` |
| `image.pullPolicy`              | KeyDB image pullPolicy                                                                | `Always`                |
| `shards`                        | Shards of KeyDB Cluster, must be equal or greater than 3                              | `3`                     |
| `shardReplicas`                 | Replicas of each shard                                                                | `1`                     |
| `labels`                        | Custom labels for the keydb pod                                                       | `{}`                    |
| `keydb.port`                    | Port to access the keydb service                                                      | `6379`                  |
| `keydb.customConfig`            | Allows for custom redis.conf files to be applied.                                     | ``                      |
| `keydb.resources`               | CPU/Memory for keydb server resource requests/limits                                  | `{}`                    |
| `auth`                          | Configures keydb with AUTH (`redisPassword` to be set)                                | ``                      |
| `nodeSelector`                  | Node labels for pod assignment                                                        | `{}`                    |
| `tolerations`                   | Toleration labels for pod assignment                                                  | `[]`                    |
| `hardAntiAffinity`              | Whether the KeyDB server pods in one shard should be forced to run on separate nodes. | `false`                 |
| `persistentVolume.storageClass` | Type of persistent volume claim                                                       | `-`                     |
| `persistentVolume.accessModes`  | ReadWriteOnce or ReadOnly                                                             | `ReadWriteOnce`         |
| `persistentVolume.size`         | Size of persistent volume claim                                                       | `1Gi`                   |
| `persistentVolume.annotations`  | Annotations of persistent volume claim                                                | `{}`                    |
| `podAnnotations`                | Annontations of pod                                                                   | `{}`                    |
| `sysctl.enabled`                | Enable an init container to modify Kernel settings                                    | `false`                 |
| `sysctl.command`                | sysctl command to execute                                                             | []                      |
| `sysctl.image`                  | sysctl Init container image name                                                      | `busybox`               |
| `sysctl.tag`                    | sysctlImage Init container image tag                                                  | `1.33.0`                |
| `sysctl.pullPolicy`             | sysctlImage Init container image pull policy                                          | `Always`                |
| `sysctl.mountHostSys`           | Whether Mount the host `/sys` folder to `/host-sys`                                   | `false`                 |     | `false` |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install \
  --set image=eqalpha/keydb \
  --set tag=alpine_x86_64_v6.0.16 \
    incubator/keydb-cluster
```

The above command sets the KeyDB server within `default` namespace.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install -f values.yaml stable/keydb-cluster
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Custom KeyDB options

This chart allows for most keydb or sentinel config options to be passed as a key value pair through the `values.yaml` under `redis.customConfig`. See links below for all available options.

[Example redis.conf](http://download.redis.io/redis-stable/redis.conf)

For example `repl-timeout 60` would be added to the `keydb.customConfig` section of the `values.yaml` as:

```yml
redis:
  customConfig: |
    repl-timeout 60
```

## Host Kernel Settings

KeyDB may require some changes in the kernel of the host machine to work as expected, in particular increasing the `somaxconn` value and disabling transparent huge pages.
To do so, you can set up a privileged initContainer with the `sysctl` config values, for example:

```
sysctl:
  enabled: false
  image: busybox
  tag: 1.33.0
  pullPolicy: Always
  command: ["sh", "-c"]
  args:
  - |-
    set -e
    set -o pipefail
    sysctl -w net.core.somaxconn=10000
    echo never > /rootfs/sys/kernel/mm/transparent_hugepage/enabled
    echo never > /rootfs/sys/kernel/mm/transparent_hugepage/defrag
    grep -q -F [never] /sys/kernel/mm/transparent_hugepage/enabled
    grep -q -F [never] /sys/kernel/mm/transparent_hugepage/defrag
```
## Start a local cluster using the hyperkit driver 

```bash

$ minikube start --driver=hyperkit

## To make hyperkit the default driver:

$ minikube config set driver hyperkit
```
