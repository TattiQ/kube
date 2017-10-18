# Install Kubernetes
0. Disable and stop apparmor + disable swap permanently by commenting it out in /etc/fstab (swapoff -a does it only for the current session)
1. Copy sysctl.conf to nodes
2. [Kubernetes install steps](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
3. Copy daemon.json into `/etc/docker/` and restart to avoid subnet conflicts (only if you have them)
3. Create the cluster with a custom cidr for the subnet
- ```kubeadm init --pod-network-cidr="10.244.0.0/16"```
- copy the config as asked to use kubectl locally
- save the join command ```kubeadm join --token 82898e.ec82f35b0df0be9a 172.31.35.32:6443 --discovery-token-ca-cert-hash sha256:784781b031ab094be635ca7efbbd218557cf9fd17b64067808f300319129d776```
4. Install flannel with rbac support (since this is k8s > 1.6)
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel.yml
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml
```
5. check flannel settings for the current node: 
```
cat /var/run/flannel/subnet.env 
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=8951
FLANNEL_IPMASQ=true
```
6. install kubeadm and docker on each worker node, instructions in step 2.
- note: i had to delete contents of `/var/lib/kubelet/`
7. repeat step 5 for each node
8. apply sysctl.conf to enable forwarding `sysctl -p /etc/sysctl.conf`
