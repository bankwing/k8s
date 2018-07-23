# Setting Up a Single Node Kubernetes Cluster Using Kubespray

## Prepare node

* Stop and Disable firewall
```
 systemctl stop firewalld
 systemctl disable firewalld
 ```

* Disable SELINUX
```
 setenforce 0
 sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
 ```

* Disable swap
```
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
echo vm.swappiness=0 >> /etc/sysctl.conf
sysctl -p
```

## Prepare bootstrap vm
* Install epel repo
```
yum install -y epel-release.noarch
```
 
* Install ansible and python
```
yum install -y ansible python python-pip
```

* Install kubespray
```
pip install kubespray
```

* Prepare kubespray config
```
kubespray prepare --nodes node1[ansible_ssh_host=10.0.0.1]
cd ~/.kubespray && pip install -r requirements.txt 
sed -i ':a;N;$!ba;s/\s*pause:\n\s*seconds:.*restart\"/\n  command: sleep 10/g' ~/.kubespray/roles/network_plugin/flannel/handlers/main.yml
sed -i ':a;N;$!ba;s/\s*pause:\n\s*seconds:.*restart\"/\n  command: sleep 10/g' ~/.kubespray/roles/docker/handlers/main.yml

```
* Generating public/private rsa key pair. 
```
ssh-keygen
```
* Copy rsa key to nodes. 
```
ssh-copy-id root@10.0.0.1 
```
 
 ## Setup Kubernetes
```
kubespray deploy 
```

* Check cluster information
```
kubectl cluster-info
```
* Config dashboard service nodeport
```
kubectl apply -f https://raw.githubusercontent.com/bankwing/k8s/master/k8s-dashboard-svc.yml 
```

* Creating admin user
```
kubectl apply -f https://raw.githubusercontent.com/bankwing/k8s/master/k8s-dashboard-adduser.yml
```

* Get admin user token
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

* Access Kubernetes dashboard via browser and auth with admin token
