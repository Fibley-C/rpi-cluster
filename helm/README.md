# Helm Install

```
#Make sure GIT is installed
apt -y install git
#We need to fix kubeconfig file for helm to stop complaining
export KUBECONFIG=~/.kube/config
mkdir ~/.kube 2> /dev/null
sudo k3s kubectl config view --raw > "$KUBECONFIG"
chmod 600 "$KUBECONFIG"
echo "KUBECONFIG=$KUBECONFIG" >> /etc/environment
#Switch to home directory
cd
#Create a directory for helm
mkdir helm
#Switch to helm directory
cd helm
#Download helm installer
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
#change permissions to execute
chmod 700 get_helm.sh
#install helm
./get_helm.sh
#check if helm is installed
helm version
```
