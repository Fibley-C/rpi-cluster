# OS Settings

Most settings will already be done via the inital bootstrap DietPi install found under `/nodes`

However we need to add **Iptables**

# Add Iptables

Make sure to finish installing Ansible first (`/ansible`)

As `root` run

`ansible k3s -m apt -a "name=iptables state=present" --become`

And reboot for belt and brace

`ansible workers -b -m shell -a "reboot"`

Then manually reboot the master node
