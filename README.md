# Install Kubernetes with Kubespray


## Links

- https://github.com/morrismusumi/kubernetes/tree/main/clusters

## Repo



```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray

virtualenv venv
source venv/bin/activate

pip3 install -r requirements.txt

```

```
cp -rfp inventory/sample inventory/mycluster
```


```
declare -a IPS=(172.16.0.153 172.16.0.167 172.16.0.213 172.16.0.250 172.16.0.206 172.16.0.126 172.16.0.128)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
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


