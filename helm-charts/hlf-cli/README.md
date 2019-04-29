# Hyperledger Fabric Chaincode / CLI

[Chaincode](http://hyperledger-fabric.readthedocs.io/) is a program, written in Go, node.js, or Java that implements a prescribed interface. Chaincode runs in a secured Docker container isolated from the endorsing peer process. Chaincode initializes and manages ledger state through transactions submitted by applications.

## TL;DR;

```bash
$ helm install helm-charts/hlf-cli
```

## Introduction

This chart deploys a sample chaincode to the hyperledger network we created earlier.  

## Prerequisites

- Kubernetes 1.9+
- PV provisioner support in the underlying infrastructure.
- K8S secrets containing:
    - the crypto-materials (e.g. signcert, key, cacert, and optionally intermediatecert, CA credentials)
    - the channel transaction for the Peer
    - the certificate of the Peer Organisation Admin
    - the private key of the Peer Organisation Admin (needed to join the channel)
- A running Hyperledger fabric network

## Installing the Chart

To install the chart with the release name `sample-cc`:

```bash
$ helm install helm-charts/hlf-cli --name sample-cc
```

The command deploys the Hyperledger Fabric sample chaincode on the Kubernetes cluster in the default configuration. 
