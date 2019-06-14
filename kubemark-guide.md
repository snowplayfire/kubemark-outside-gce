## Introduction
This document serves to understand how to set up kubemark cluster outside gce.

## Precondition
You need two k8s clusters(cluster one and cluster two) to set up one kubemark cluster. 

The functions of the two clusters are as follows:

- cluster one: its masters will be the kubemark cluster's masters.
- cluster two: it's used to create hollow nodes for the kubemark cluster.

## Steps:
1. Build kubemark image
1.1 pull kubernetes code
``
cd $GOPATH/src/k8s.io/
git clone git@github.com:kubernetes/kubernetes.git
``
1.2 build kubemark binary 
``
./hack/build-go.sh cmd/kubemark/
cp $GOPATH/src/k8s.io/kubernetes/_output/bin/kubemark $GOPATH/src/k8s.io/kubernetes/cluster/images/kubemark/
``
1.3 build kubemark image
``
cd $GOPATH/src/k8s.io/kubernetes/cluster/images/kubemark/
make build
``

Then you can get the image: staging-k8s.gcr.io/kubemark:latest

2. Create hollow nodes
2.1 create namesapce, configmap and secret

Copy cluster one's kubeConfig, put it on a master of cluster two, rename it as config.
``
kubectl create ns kubemark
kubectl create configmap node-configmap -n kubemark --from-literal=content.type="test-cluster"
kubectl create secret generic kubeconfig --type=Opaque --namespace=kubemark --from-file=kubelet.kubeconfig=config --from-file=kubeproxy.kubeconfig=config
``
2.2 use yaml to create hollow nodes

You can adjust the replicas according to your need.
``
kubectl create -f hollow-node-sts.yaml
``

Waiting for pods to be running. Then you can see these pods have registered as cluster one's nodes.

Finally, cluster one and cluster two set up a kubemark cluster.
