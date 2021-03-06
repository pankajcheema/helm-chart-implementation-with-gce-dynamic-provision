# Hyperledger Fabric Peer

[Hyperledger Fabric Peer](http://hyperledger-fabric.readthedocs.io/) is the node type responsible for endorsing transactions and recording them on the Blockchain for the [Hyperledger](https://www.hyperledger.org/) Fabric permissioned blockchain framework.

## TL;DR;

```bash
$ helm install helm-charts/hlf-peer
```

## Introduction

The Hyperledger Fabric Peer can either use a `goleveldb` or a `CouchDB` database for holding the ledger data.

This Peer can receive transaction requests, which it checks and signs, endorsing them. These endorsements can then be sent to the Ordering Service (one or more Orderer nodes), which will package them and return blocks that the Peer can then commit to their own Ledger.

## Prerequisites

- Kubernetes 1.9+
- PV provisioner support in the underlying infrastructure.
- K8S secrets containing:
    - the crypto-materials (e.g. signcert, key, cacert, and optionally intermediatecert, CA credentials)
    - the channel transaction for the Peer
    - the certificate of the Peer Organisation Admin
    - the private key of the Peer Organisation Admin (needed to join the channel)
- A running [HLF-CouchDB Chart](https://github.com/kubernetes/charts/tree/master/stable/hlf-couchdb) if you are using the `CouchDB` database.

## Installing the Chart

To install the chart with the release name `peer1`:

```bash
$ helm install helm-charts/hlf-peer --name peer1
```

The command deploys the Hyperledger Fabric Peer on the Kubernetes cluster in the default configuration. The [Configuration](#configuration) section lists the parameters that can be configured during installation.

### Custom parameters

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
$ helm install helm-charts/hlf-peer --name peer1 --set peer.mspID=MyMSP
```

Alternatively, a YAML file can be provided while installing the chart. This file specifies values to override those provided in the default values.yaml. For example,

```bash
$ helm install helm-charts/hlf-peer --name peer1 -f my-values.yaml
```

## Updating the chart

To update the chart:

```bash
$ helm upgrade peer1 helm-charts/hlf-peer -f my-values.yaml
```

## Uninstalling the Chart

To uninstall/delete the `peer1` deployment:

```bash
$ helm delete peer1
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the Hyperledger Fabric Peer chart and default values.

| Parameter                          | Description                                     | Default                                                    |
| ---------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------- |
| `image.repository`                 | `hlf-peer` image repository                          | `hyperledger/fabric-peer`                                  |
| `image.tag`                        | `hlf-peer` image tag                                 | `x86_64-1.1.0`                                             |
| `image.pullPolicy`                 | Image pull policy                                    | `IfNotPresent`                                             |
| `service.portRequest`              | TCP port for requests to Peer                        | `7051`                                                     |
| `service.portEvent`                | TCP port for event service on Peer                   | `7053`                                                     |
| `service.type`                     | K8S service type exposing ports, e.g. `ClusterIP`    | `ClusterIP`                                                |
| `persistence.accessMode`           | Use volume as ReadOnly or ReadWrite                  | `ReadWriteOnce`                                            |
| `persistence.annotations`          | Persistent Volume annotations                        | `{}`                                                       |
| `persistence.size`                 | Size of data volume (adjust for production!)         | `1Gi`                                                      |
| `persistence.storageClass`         | Storage class of backing PVC                         | `default`                                                  |
| `ingress.enabled`                  | If true, Ingress will be created                     | `false`                                                    |
| `ingress.annotations`              | Ingress annotations                                  | `{}`                                                       |
| `ingress.path`                     | Ingress path                                         | `/`                                                        |
| `ingress.hosts`                    | Ingress hostnames                                    | `[]`                                                       |
| `ingress.tls`                      | Ingress TLS configuration                            | `[]`                                                       |
| `peer.databaseType`                | Database type to use (`goleveldb` or `CouchDB`)      | `goleveldb`                                                |
| `peer.couchdbInstance              | CouchDB chart name to use `cdb-peer1`                | `cdb-peer1`                                                |
| `peer.mspID`                       | ID of MSP the Peer belongs to                        | `Org1MSP`                                                  |
| `peer.gossip.bootstrap`            | Gossip bootstrap address                             | ``                                                         |
| `peer.gossip.endpoint`             | Gossip endpoint                                      | ``                                                         |
| `peer.gossip.externalEndpoint`     | Gossip external endpoint                             | ``                                                         |
| `peer.gossip.orgLeader`            | Gossip organisation leader ("true"/"false")          | `"false"`                                                  |
| `peer.gossip.useLeaderElection`    | Gossip use leader election                           | `"true"`                                                   |
| `peer.tls.server.enabled`          | Do we enable server-side TLS?                        | `false`                                                    |
| `peer.tls.client.enabled`          | Do we enable client-side TLS?                        | `false`                                                    |
| `secrets.peer.cred`                | Credentials: 'CA_USERNAME' and 'CA_PASSWORD'         | ``                                                         |
| `secrets.peer.cert`                | Certificate: as 'cert.pem'                           | ``                                                         |
| `secrets.peer.key`                 | Private key: as 'key.pem'                            | ``                                                         |
| `secrets.peer.caCert`              | CA Cert: as 'cacert.pem'                             | ``                                                         |
| `secrets.peer.intCaCert`           | Int. CA Cert: as 'intermediatecacert.pem'            | ``                                                         |
| `secrets.peer.tls`                 | TLS secret: as 'tls.crt' and 'tls.key'               | ``                                                         |
| `secrets.peer.tlsRootCert`         | TLS root CA certificate: as 'cert.pem'               | ``                                                         |
| `secrets.peer.tlsClient`           | TLS client secret: as 'tls.crt' and 'tls.key'        | ``                                                         |
| `secrets.peer.tlsClientRootCerts`  | TLS Client root CA certificate files (any name)      | ``                                                         |
| `secrets.channel`                  | Secret containing Channel tx for peer to create/join | ``                                                         |
| `secrets.adminCert`                | Secret containing Peer Org admin certificate         | ``                                                         |
| `secrets.adminCert`                | Secret containing Peer Org admin private key         | ``                                                         |
| `secrets.ordTlsRootCert`           | Secret containing Orderer TLS root CA certificate    | ``                                                         |
| `resources`                        | CPU/Memory resource requests/limits                  | `{}`                                                       |
| `nodeSelector`                     | Node labels for pod assignment                       | `{}`                                                       |
| `tolerations`                      | Toleration labels for pod assignment                 | `[]`                                                       |
| `affinity`                         | Affinity settings for pod assignment                 | `{}`                                                       |

## Persistence

The volume stores the Fabric Peer data and configurations at the `/var/hyperledger` path of the container.

The chart mounts a [Persistent Volume](http://kubernetes.io/docs/user-guide/persistent-volumes/) at this location. The volume is created using dynamic volume provisioning through a PersistentVolumeClaim managed by the chart.