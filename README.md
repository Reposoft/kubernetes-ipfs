
# IPFS cluster in Kubernetes

Configures a small cluster-local ipfs swarm, with focus on [http API](https://github.com/ipfs/http-api-spec) access.

A useful introduction, though with command line focus: https://medium.com/@ConsenSys/an-introduction-to-ipfs-9bba4860abd0.

## Howto

First [create a key for your private swarm](https://github.com/ipfs/go-ipfs/blob/v0.4.11/docs/experimental-features.md#private-networks) and upload using:
```bash
kubectl apply -f ./00namespace.yml
kubectl -n ipfs create secret generic ipfs --from-file=./swarm.key
```

Then optionally configure service `type` in the 3* yml:s, then do
`kubectl apply -f .`.

### Check status of your swarm

Peers (i.e. pods) find each other automagically.

```bash
kubectl -n ipfs get pods -o wide
# Should include "/ip4/..." entries for the peers:
kubectl -n ipfs exec ipfs-0 -- ipfs swarm peers
```

### Ongoing investigation

Remains to investigate this log output
which repeats itself endlessly:
```
DEBUG       mdns: starting mdns query mdns.go:125
DEBUG       mdns: Handling MDNS entry: 172.17.0.13:4001 QmTgXq7EM2jyArCPAPKsBwdjCmbVqnypXouCxSWQNzPTrP mdns.go:147
WARNI       core: trying peer info: {<peer.ID TgXq7E> [/ip4/172.17.0.13/tcp/4001]} core.go:390
DEBUG       mdns: Handling MDNS entry: 172.17.0.14:4001 QmXuE8WGu4pGNuUyz1bsMpwpZrShVDGpokSyWvCMuYCqNA mdns.go:147
WARNI       core: trying peer info: {<peer.ID XuE8WG> [/ip4/172.17.0.14/tcp/4001]} core.go:390
DEBUG       core: <peer.ID aoBGog> no more bootstrap peers to create 1 connections bootstrap.go:141
DEBUG       core: <peer.ID aoBGog> bootstrap error: not enough bootstrap peers to bootstrap bootstrap.go:87
DEBUG       mdns: Handling MDNS entry: 172.17.0.12:4001 QmUKJ7Fze9kWaxgZR7vUzZzWFgcrAi6AQ3ciX7iCMKw4fn mdns.go:147
DEBUG       mdns: Handling MDNS entry: 172.17.0.10:4001 QmaoBGogYpHNBhnmpg5e6Hoerc45kdiwvgDEAk1wTrC1H2 mdns.go:147
DEBUG       mdns: got our own mdns entry, skipping mdns.go:155
WARNI       core: trying peer info: {<peer.ID UKJ7Fz> [/ip4/172.17.0.12/tcp/4001]} core.go:390
DEBUG       mdns: mdns query complete mdns.go:138
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
