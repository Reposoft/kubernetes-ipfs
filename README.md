
# IPFS cluster in Kubernetes

Configures a small cluster-local ipfs swarm, with focus on [http API](https://github.com/ipfs/http-api-spec) access.

A useful introduction, though with command line focus: https://medium.com/@ConsenSys/an-introduction-to-ipfs-9bba4860abd0#.amiv2jsoe.

## Bootstrapping

If pod logs `Error: failure writing to dagstore: fsync: invalid argument` you're probably on NFS or something. With minikube `kubectl create -f ./bootstrap/pv-template.yml` will get you a temp dir that ipfs is ok with if you run `sudo chown -R 1000 /tmp/k8s-data/ipfs-data-0` in `minikube ssh`.

## Testing

IP and port depends on your setup and the `type` you chose for services. Example with minikube NodePort and the random ports from `kubectl create -f 3[service].yml`:

```
$ ipfs_api=192.168.99.101:30115
$ ipfs_ro=192.168.99.101:30715
$ curl -F "data=@./test1" http://$ipfs_api/api/v0/add
{"Name":"test1","Hash":"QmSUFD7V8MfmLYEHWw9phnGEFhrjuYxGTgzEtMJuNoB6Jq"}
$ $ curl http://$ipfs_ro/ipfs/QmSUFD7V8MfmLYEHWw9phnGEFhrjuYxGTgzEtMJuNoB6Jq
{"test":1}
```

## Remote clusters

New in Kubernetes [1.3](http://blog.kubernetes.io/2016/07/kubernetes-1.3-bridging-cloud-native-and-enterprise-workloads.html). We should test that.
