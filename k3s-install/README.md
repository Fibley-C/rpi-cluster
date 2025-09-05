# K3s install

For reference my setup consists of the following:

```
192.168.1.30 k3s-master k3s-master.local
192.168.1.31 k3s-node1 k3s-node1.local
192.168.1.32 k3s-node2 k3s-node2.local
```

## Master

In my case **k3s-master** (duh) is the primary node

To install k3s and initalise this node as the master run

```
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644 --disable servicelb --token some_random_password --node-taint CriticalAddonsOnly=true:NoExecute --bind-address 192.168.1.30 --disable-cloud-controller --disable local-storage
```

Some explanations:

- **--write-kubeconfig-mode 644** - This is the mode that we want to use for the kubeconfig file. Its optional, but needed if you want to connect to Rancher manager later on.
- **--disable servicelb** - This is the flag that we want to use to disable the service load balancer. (We will use metallb instead)
- --token - This is the token that we want to use to connect to the K3s master node. Choose a random password, but keep remember it.
- **--node-taint** - This is the flag that we want to use to add a taint to the K3s master node. This will mark the node to not run any containers except the ones that are critical.
- **--bind-address** - This is the flag that we want to use to bind the K3s master node to a specific IP address.
- **--disable-cloud-controller** - This is the flag that we want to use to disable the K3s cloud controller. I don't think I need it.
- **--disable local-storage** - This is the flag that we want to use to disable the K3s local storage. I'm going to setup longhorn storage provider instead.

Once the command has ran we can verify with

```
root@k3s-master:~/gitops/apps# kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
k3s-master   Ready    control-plane,master   240d   v1.33.4+k3s1
```

## Workers

We need to join some workers now; in our case k3s-node1 and k3s-node2. Furthermore, we are going to execute the join command on each node with Ansible.

```
ansible workers -b -m shell -a "curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.30:6443 K3S_TOKEN=some_random_password sh -"
```

Give it some time for the install to take place on the nodes, we can watch progress with

`watch kubectl get nodes`

After the install is complete we should see

```
NAME         STATUS   ROLES                  AGE    VERSION
k3s-master   Ready    control-plane,master   6m   v1.33.4+k3s1
k3s-node1    Ready    <none>                 72s   v1.33.4+k3s1
k3s-node2    Ready    <none>                 73s   v1.33.4+k3s1
```

## Setting role/labels

> k3s by default allow pods to run on the control plane, which can be OK, but in production it would not. However, in our case, we already tagged the master node when we installed the K3s. I still want to control a bit more where workload is deployed, and it's also good to know how it's done. So, We will be using labels to tell pods/deployment where to run.

Let's add this tag key:value: kubernetes.io/role=worker to worker nodes. This is more cosmetic, to have nice output from kubectl get nodes.

```
kubectl label nodes k3s-node1 kubernetes.io/role=worker
kubectl label nodes k3s-node2 kubernetes.io/role=worker
```

Another label/tag. I will use this one to tell deployments to prefer nodes where `node-type` equals `workers`. The node-type is our chosen name for key, you can call it whatever.

```
kubectl label nodes k3s-node1 node-type=worker
kubectl label nodes k3s-node2 node-type=worker
```

You can verify with

```
root@k3s-master:~/gitops/apps# kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
k3s-master   Ready    control-plane,master   240d   v1.33.4+k3s1
k3s-node1    Ready    worker                 236d   v1.33.4+k3s1
k3s-node2    Ready    worker                 236d   v1.33.4+k3s1
```

And to check taints you can run

```
root@k3s-master:~/gitops/apps# kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints --no-headers
k3s-master   [map[effect:NoExecute key:CriticalAddonsOnly value:true]]
k3s-node1    <none>
k3s-node2    <none>
```

Lastly, add following into `/etc/environment` (this is so the Helm and other programs know where the Kubernetes config is.)

`echo "KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> /etc/environment`

or use Ansible

`ansible cube -b -m lineinfile -a "path='/etc/environment' line='KUBECONFIG=/etc/rancher/k3s/k3s.yaml'"`
