### Reference
- https://www.edureka.co/blog/install-kubernetes-on-ubuntu#SetupKubernetesEnvironment
- https://joshh.info/2018/kubernetes-dashboard-https-nodeport/

This tutorial is applied on VirtualBox, running Ubuntu server 16.04 on a NAT network.

kubectl apply -f https://docs.projectcalico.org/v3.7/manifests/calico.yaml

### Prepare steps on VMs (master + slave)
**Pre-requisites To Install Kubernetes**

Since we are dealing with VMs, we recommend the following settings for the VMs:-

- Master: 2 GB RAM, 2 Cores of CPU 
- Slave/ Node: 1 GB RAM, 1 Core of CPU

**Turn off swap** and edit fstab to comment out the line which has mention of swap partition.
```
$ sudo su
$ swapoff -a
$ vi /etc/fstab
```
**Update The Hostnames**

To change the hostname of both machines, run the below command to open the file and subsequently rename the master machine to ‘kmaster’ and your node machine to ‘knode’.

```
$ vi /etc/hostname
```

**Update The Hosts File** With IPs Of Master & Node
```
$ ifconfig
$ vi /etc/hosts
```

**Install Docker & SSH**
```
$ apt-get install openssh-server 
$ apt-get install -y docker.io
```

**Install K8s**
```
$ apt-get update && apt-get install -y apt-transport-https curl
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
  deb http://apt.kubernetes.io/ kubernetes-xenial main
  EOF
$ apt update
4 apt-get install -y kubelet kubeadm kubectl
```

### Install KMaster
Now it's time to init the master:
```
kubeadm init --apiserver-advertise-address=10.10.21.135 --pod-network-cidr=192.168.0.0/16
```

Following the initialization, run these commands as a normal user
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Check that pods are being started
```
kubectl get pods -o wide --all-namespaces
```

You will notice from the previous command, that all the pods are running except one: ‘kube-dns’. For resolving this we will install a pod network. To install the CALICO pod network, run the following command:
```
kubectl apply -f https://docs.projectcalico.org/v3.7/manifests/calico.yaml
```

Now all pods should be in running state.

It's time to install a dashboard
```
$ curl https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml -O
$ kubectl apply -f kubernetes-dashboard.yaml
```

Edit dashboard service to change: spec.type from ClusterIP to NodePort, spec.ports[0].nodePort from 32641 to whatever port you want it to be exposed on
```
$ kubectl -n kube-system edit service kubernetes-dashboard
```

Check service
```
$ kubectl -n kube-system get services
```
	
From virtualBox, map internal port (eg 31425 to host port 8001), then you can access the dashboard at ```http://localhost:8001```

Create the service account for the dashboard and get it’s credentials:
```
$ kubectl create serviceaccount dashboard -n default
$ kubectl create clusterrolebinding dashboard-admin -n default --clusterrole=cluster-admin --serviceaccount=default:dashboard
$ kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```

### Join the Slave

Get the token on master:
``` 
$ kubeadm token create  --ttl=0
$ kubeadm token list
```

Join the slave: ```$ sudo kubeadm join 10.0.2.6:6443 --token 9tyi10.t95k1ronssuabpa8 --discovery-token-unsafe-skip-ca-verification```

Check on master: ```$ kubeadm get nodes```

To reset kubeadm init or join: ```$ kubeadm reset```

### OTHERS

   58  kubectl get pods -o wide --all-namespaces
   59  service kubelet status

   45  source <(kubectl completion bash)
   46  echo "source <(kubectl completion bash)" >> ~/.bashrc
   49  alias k=kubectl
   54  complete -F __start_kubectl k
