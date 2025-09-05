# Ansible Setup

First update your host file on each node with the IP of each node on the cluster. Mine looks like

```
#/etc/hosts
192.168.1.30 k3s-master k3s-master.local
192.168.1.31 k3s-node1 k3s-node1.local
192.168.1.32 k3s-node2 k3s-node2.local
```

## Install Ansible

`apt install ansible`

Then create `/etc/ansible/hosts`

eg.

```
#/etc/ansible/hosts
[control]
k3s-master ansible_connection=local

[workers]
k3s-node1 ansible_connection=ssh
k3s-node2 ansible_connection=ssh

[k3s:children]
control
workers
```

Lastly we can make it so that root can conect to each node from k3s-master without having to enter the password everytime.

```
# Make sure you are user root
cd
mkdir -p ~/.ssh
chmod 700 ~/.ssh
# Do not fill anything in next command just enter
ssh-keygen -t rsa
# Copy keys to each node, for example:
ssh-copy-id -i ~/.ssh/id_rsa.pub root@k3s-node1
ssh-copy-id -i ~/.ssh/id_rsa.pub root@k3s-node2
```

## Test Ansible

We can run a simple ping across all nodes to ensure Ansible is setup and working

`ansible workers -m ping`

The result should look like

```
k3s-node2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
k3s-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
