###  Foreign intelligence kubernetes todo
1. Check ports for kubelet/etcd and others k8s-components
- Ports of Kubernetes components sticking out 

| Port            | Process        | Description                                                   |
|-----------------|----------------|---------------------------------------------------------------|
| 443/TCP         | kube-apiserver | Kubernetes API port                                           |
| 2379-80/TCP     | etcd           |                                                               |
| 6666/TCP        | etcd           | etcd                                                          |
| 4194/TCP        | cAdvisor       | Container metrics                                             |
| 6443/TCP        | kube-apiserver | Kubernetes API port                                           |
| 8443/TCP        | kube-apiserver | Minikube API port                                             |
| 8080/TCP        | kube-apiserver | Insecure API port                                             |
| 10250/TCP       | kubelet        | HTTPS API which allows full mode access                       |
| 10255/TCP       | kubelet        | Unauth read-only HTTP port: pods, running pods and node state |
| 10249,10256/TCP | kube-proxy     | Kube Proxy health-check server                                |
| 9099/TCP        | calcio-felix   | Health check server for Calcio                                |
| 6782-4/TCP      | weave          | Metrics and endpoints                                         |
| 30000-32767/TCP | NodePort       | Proxy to the services                                         |
| 44134/TCP       | Tiler          | Helm service listening                                        |



2. Check endpoints (kube-proxy, apiserver)
- curl –k https://host:443/api{apis}/
- curl –k https://host:10250/pods
- curl –k https://host:443/api/v1/{pods,nodes,services,namespaces...}
- curl –k https://host:443/api/v1/{namespaces}/{podname}/{logs}
- curl -k -v https://host:10249/proxyMode

1. Check kubernetes dashboard (**broken web**=**cluster access**)
- Weave Scope (4040)
- Kubernetes Dashboard (8443)
- Prometheus (9090)
- Octant (8900)