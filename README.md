
# IPFS cluster in Kubernetes

Configures a small cluster-local ipfs swarm, with focus on [http API](https://github.com/ipfs/http-api-spec) access.

A useful introduction, though with command line focus: https://medium.com/@ConsenSys/an-introduction-to-ipfs-9bba4860abd0#.amiv2jsoe.

## Bootstrapping

If pod logs `Error: failure writing to dagstore: fsync: invalid argument` you're probably on NFS or something. With minikube `kubectl create -f ./bootstrap/pv-template.yml` will get you a temp dir that ipfs is ok with if you run `sudo chown -R 1000 /tmp/k8s-data/ipfs-data-0` in `minikube ssh`.

## Remote clusters

New in Kubernetes [1.3](http://blog.kubernetes.io/2016/07/kubernetes-1.3-bridging-cloud-native-and-enterprise-workloads.html). We should test that.
