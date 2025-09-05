# MetalLB

K3s will come with pretty much everything pre-configured like traefik.

More info about traefik: https://doc.traefik.io/traefik/ or small guide from me Traefik.

However, I would like to have LoadBalancer, and in essence to be able to give services (pods) an external IP. Just like my Kubernetes nodes and not from internal Kubernetes ranges. Normally, this is an external component, and your cloud provider should somehow magically give that to you, but since we are our own cloud provider, and we are trying to keep everything in one cluster... in short MetalLB is the answer.

## What is MetalLB

https://metallb.universe.tf/

## Deployment

> On your control node

Ensuring `helm` is installed as per `/helm`

```
# First add metallb repository to your helm
helm repo add metallb https://metallb.github.io/metallb
# Check if it was found
helm search repo metallb
# Install metallb
helm upgrade --install metallb metallb/metallb --create-namespace \
--namespace metallb-system --wait
```

This should install MetalLB into your cluster and return something like this:

```
Release "metallb" does not exist. Installing it now.
NAME: metallb
LAST DEPLOYED: Sat Apr 1 10:49:30 2023
NAMESPACE: metallb-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
MetalLB is now running in the cluster.

Now you can configure it via its CRs. Please refer to the metallb official docs
on how to use the CRs
```

## Config

MetalLB needs to know what IP range to use for external IPs. This is done via a Custom Resource. You can find the CR in the official documentation. So lets create and apply a Custom Resource with the following content:

```
cat << 'EOF' | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.40-192.168.1.61
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```

It should return something like this:

```
ipaddresspool.metallb.io/default-pool created
l2advertisement.metallb.io/default created
```

As you can see, I specified a range from 192.168.1.40 to 192.168.0.61. That will give me 21 "external" IPs to work with for now.
