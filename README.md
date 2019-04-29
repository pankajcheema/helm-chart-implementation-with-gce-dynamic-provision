# Hyperkube

[Hyperkube](https://squareops.com) is the project for deploying [Hyperledger](https://www.hyperledger.org/) Fabric permissioned blockchain framework over kubernetes.

For simplicity, this project is divided in 2 stages:
- Dev Setup
- Production Setup

## Install Kubernetes
You can either use managed kubernetes , or install one for yourself. Follow Instructions in clouds directory to setup a kubernetes on a cloud provider

## Dev Setup
Follow instructions in env/dev directory to setup a minimal hyperledger setup with 1 orderer, 1 peer backed by couchDB and no TLS

## Production setup