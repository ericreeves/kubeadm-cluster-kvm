## kubeadm cluster on KVM with NGINX Ingress

[Read full blog article here](https://fabianlee.org/2022/05/25/kvm-kubeadm-cluster-on-kvm-using-ansible/)

![kubeadm cluster](https://github.com/fabianlee/kubeadm-cluster-kvm/raw/main/diagrams/kubeadm-3node.png)

Modify any variables for environment:
  * vi group_vars/all

Install Prerequisites OS packages and pip modules for Ansible:
  * ./install_requirements.sh

Create ssh keypair for guest VMs
```
cd tf-libvirt
ssh-keygen -t rsa -b 4096 -f id_rsa -C tf-libvirt -N "" -q
cd ..
```

Create local KVM guest VM instances:
  * ansible-playbook playbook_terraform_kvm.yml

Deploy kubeadm:
  * ansible-playbook playbook_kubeadm_dependencies.yml
  * ansible-playbook playbook_kubeadm_controlplane.yml
  * ansible-playbook playbook_kubeadm_workers.yml

Wait for all pods to be ready:

```
export KUBECONFIG=/tmp/kubeadm-kubeconfig
kubectl get pods -A
```

MetalLB with NGINX Ingress:
  * ansible-playbook playbook_metallb.yml
  * ansible-playbook playbook_certs.yml
  * ansible-playbook playbook_nginx_ingress.yml 

Wait for NGINX Ingress to have its IP address:

```
kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath="{.status.loadBalancer.ingress}"
```

Hello World deployment:
  * ansible-playbook playbook_deploy_myhello.yml


Validate ingress locally:
  * add entry to local /etc/hosts
```
    x.y.z.145 kubeadm.local
```

  * test myhello deployment exposed via NGINX Ingress
```
curl https://kubeadm.local/myhello/
```
