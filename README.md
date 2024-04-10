# Install Kubernetes with Kubespray


## Links

- https://github.com/morrismusumi/kubernetes/tree/main/clusters


## Kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin


#or
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install kubectl
```


## Repo



```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
git checkout tags/v2.24.1

virtualenv venv
# or
python3 -m venv kubespray-venv

source kubespray-venv/bin/activate
source venv/bin/activate

pip3 install -r requirements.txt
pip3 install -U -r requirements.txt

```

```
cp -rfp inventory/sample inventory/mycluster
cp -rfp inventory/sample inventory/proxmox01
```


```
declare -a IPS=(172.16.0.153 172.16.0.167 172.16.0.213 172.16.0.250 172.16.0.206 172.16.0.126 172.16.0.128)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

rename node names in config: inventory/proxmox01/hosts.yaml
Also configure 3 Master and 3 etcd

create a variable file to pass when we execute
vi inventory/proxmox01/sample/cluster-variable.yaml
kube_version: v1.27.5
helm_enabled: true
kube_proxy_mode: iptables
# Metallb comments

ansible-playbook -i inventory/proxmox01/hosts.yaml -e @inventory/proxmox01/custer-variable.yaml --become --become-user=root -u ansible cluster.yml




ansible-playbook -i inventory/mycluster/hosts.yaml  --become --user=root cluster.yml

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


# Install Metallb

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/manifests/metallb-native.yaml
```

```
mkdir inventory/proxmox01/metallb
```


vi IPAddressPool.yaml
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.16.43.1-172.16.43.128
```

vi I2advertisement.yaml
```
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default-advertisement
  namespace: metallb-system
```

```
kubectl apply -f ./inventory/proxmox01/metallb/

kubectl get svc
```








# Update Ansible inventory file with inventory builder
```
declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

ansible-playbook -i inventory/mycluster/hosts.yaml  --user root upgrade-cluster.yml
```


# Review and change parameters under ``inventory/mycluster/group_vars``
```
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml

# Deploy Kubespray with Ansible Playbook - run the playbook as root
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!

pip3 uninstall ansible ansible-base
pip3 install ansible==2.9.14
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml

ansible-playbook -i inventory/mycluster/hosts.yaml  --user root cluster.yml


cat contrib/inventory_builder/inventory.py

mkdir ~/.kube
scp the config from master node /etc/Kubernetes/admin.conf


ansible-playbook -i inventory/mycluster/hosts.yaml  --user root upgrade-cluster.yml

vim ./inventory/group_vars/k8s-cluster.yml
-kube_network_plugin: calico
+kube_network_plugin: flannel


curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update

helm install nginx nginx-stable/nginx-ingress
```


