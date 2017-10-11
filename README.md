
# IPFS cluster in Kubernetes

Configures a small cluster-local ipfs swarm, with focus on [http API](https://github.com/ipfs/http-api-spec) access.

A useful introduction, though with command line focus: https://medium.com/@ConsenSys/an-introduction-to-ipfs-9bba4860abd0.

## Bootstrapping

Optionally configure service `type` in the 3* yml:s, then do
`kubectl create -f .`.

### Bootstrap configuration

We assume that you want to start out completely local, and only later participate in public ledgers.

When you start the offical container (wihtout an `--offline` arg)
it will use with the default public [bootstrap](https://ipfs.io/docs/commands/#ipfs-bootstrap) list.

Instead you want [ipfs swarm peers](https://ipfs.io/docs/commands/#ipfs-swarm-peers) to be your local addresses:
```
$ kubectl exec ipfs-0 -- ipfs swarm peers
/ip4/[cluster-local IP here]/tcp/4001/ipfs/...

# with test content from below
$ kubectl exec ipfs-0 -- ipfs cat QmSUFD7V8MfmLYEHWw9phnGEFhrjuYxGTgzEtMJuNoB6Jq
{"test":1}
$ kubectl exec ipfs-1 -- ipfs cat QmSUFD7V8MfmLYEHWw9phnGEFhrjuYxGTgzEtMJuNoB6Jq
{"test":1}
```

This happens automatically and only locally if you clear the bootstrap list prior to first online daemon start.
The bootstrap/swarm_local.sh script tries to do such a thing.
With `debug` level logging you know that you're local if the amount of logs isn't too big for a human to follow.

### Ongoing investigation

Remains to investigate this log output (notably on debug level):
```
04:09:25.525 DEBUG       core: <peer.ID RnHFxa> no more bootstrap peers to create 3 connections bootstrap.go:141
04:09:25.525 DEBUG       core: <peer.ID RnHFxa> bootstrap error: not enough bootstrap peers to bootstrap bootstrap.go:87
```
and this, notably on warn level:
```
04:09:25.527 WARNI       core: trying peer info: {<peer.ID eSjMnW> [/ip4/172.17.0.5/tcp/4001]} core.go:201
```

## Persistence

Note that ipfs has no ambition to replicate all content across all nodes,
as that would be very impractical for next generation internet :)

TODO is distributed [pin](https://www.reddit.com/r/ipfs/comments/69l1y8/does_ipfs_add_automatically_pin/),
like in [ipfs-cluster](https://github.com/ipfs/ipfs-cluster) (which reinvents too much of Kubernetes for our taste).

## Testing

IP and port depends on your setup and the `type` you chose for services. Example with minikube NodePort and the random ports from `kubectl create -f 3[service].yml`:

```bash
ipfs_api=$(minikube service -n ipfs --url api)
ipfs_ro=$(minikube service -n ipfs --url readonly)
echo '{"test":1}' > test1
curl -F "data=@./test1" $ipfs_api/api/v0/add
# {"Name":"test1","Hash":"QmSUFD7V8MfmLYEHWw9phnGEFhrjuYxGTgzEtMJuNoB6Jq"}
curl $ipfs_ro/ipfs/QmSUFD7V8MfmLYEHWw9phnGEFhrjuYxGTgzEtMJuNoB6Jq
# {"test":1}
```

## Remote clusters

New in Kubernetes [1.3](http://blog.kubernetes.io/2016/07/kubernetes-1.3-bridging-cloud-native-and-enterprise-workloads.html). We should test that.
