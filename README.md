## Install Kubectl:
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
&& yum install -y kubectl
```
### Important:

-Remember to always specify your namespace for most of kubectl commands execution.

-Though I use yaml in the examples, kubectl also takes json. 

### Useful commands: 

Create an object from file (or STDIN):
```
kubectl create -f myfile.yaml
```
Apply a configuration to an object (will create if it doesn't exists):
```
kubectl apply -f myfile.yaml
```
List objects:
```
kubectl get <object> -n <your-namespace>
```  
Get detailed info about an object:
```
kubectl describe <object> <object-name> -n <your-namespace>
```
Exmaple of object information (I picked pod):
```
kubectl describe pod <your-pod> -n <your-namespace>
```
Check pod logs (IE: to check nginx log files inside our container (-f stands for follow, as tail does):
```
kubectl logs <pod-ID> -f -n <your-namespace>
```
Check container logs for a pod with 2 or more containers (IE: to check node log files inside our container):
```
kubectl logs <pod-ID> -f -c node -n <your-namespace>
```
Connect to shell’s container:
```
kubectl exec -it <pod-ID> <your-containers-shell> -n <your-namespace>
```
Connecting to a container example:
```
kubectl exec -it <pod-ID> -n <your-namespace> sh
```
Delete (terminate) a pod:
```
kubectl delete pod <pod-ID> -n <your-namespace>
```
Edit an object:
```
kubectl edit <object> <object-ID> -n <your-namespace>
```

### Advanced commands:
```
kubectl get nodes -o json |
       jq ".items[] | {name:.metadata.name} + .status.capacity"
```
```
IP=$(kubectl get svc <some-service> -o go-template --template '{{ .spec.clusterIP }}')
```
```
NODEPORT=$(kubectl get svc/registry -o json | jq .spec.ports[0].nodePort)
```
```
REGISTRY=127.0.0.1:$NODEPORT
```
```
kubectl get deploy -o json |
       jq ".items[] | {name:.metadata.name} + .spec.strategy.rollingUpdate"
```
### Adding auto complete to your bash:
```
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

## Create a K8s namespace (I'll use monitoring as an example):
Run this at your cluster:
```
kubectl create -f https://raw.github.com/pulpbill/Kubernetes/master/ns-monitoring.yaml
```
### Further reading / notes:

- A service account provides an identity for processes that run in a Pod.
When you access the cluster (for example, using kubectl), you are authenticated by the API Server as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default)

Reference : https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

### Additional commands:

SSH to cluster:

ssh -i ~/.ssh/yourkey admin@api.yoursubdomain.example.com
