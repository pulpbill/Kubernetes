## Kubectl

### Important:

-Remember to always specify your namespace for most of kubectl commands execution.

-Though I use yaml in the examples, kubectl also takes json. 

### Install Kubectl at Amazon Linux2 AMI (RedHat based): 

Use this script: https://github.com/pulpbill/daily-bash-scripts/blob/master/kubectl.sh

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

## KOPS

### Creating a K8s cluster with kops:

I have an user (AWS user created via IAM) with S3/R53/EC2/VPC/IAM admin access and created a set of keys.

Create ssh key pairs with ssh-keygen and then set it:
```
kops create secret --name yoursubdomain.example.com sshpublickey admin -i ~/.ssh/yourkey.pub
```

1. Install Kops (you must have kubectl installed):
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64 && chmod +x kops-linux-amd64 && mv kops-linux-amd64 /usr/local/bin/kops
```

2. Delegate authority for the subdomain you choose (get your subdomain nameservers and add a NS record to the parent hosted zone that contains these 4 ns records so it can know where to resolve your subdomain).

3. Create a S3 bucket (at your AWS console or via CLI) and set the environment variable so Kops can manage files:
```
export KOPS_STATE_STORE=s3://your-bucket
```

4. Create cluster configuration (it's like a dry-run):
```
kops create cluster --node-count=2 --node-size=t2.small --node-volume-size=8 --master-volume-size=8 --zones=us-east-1a --name=yoursubdomain.example.com --master-size=t2.small --master-count=1 --dns-zone=yoursubdomain.example.com --ssh-public-key=~/.ssh/yourkey.pub
```
 
 #### Note:

I had an issue last week (mid sept. 2018) where the cluster failed, I don't know why (didn't have the time to get into that) I use t2.micro/nano because I'm just doing some tests, and after changing node/master size to t2.small, worked.
 
5. Create the cluster (it only adds --yes to the previous command (that's why I called it dry run)):
```
kops update cluster yoursubdomain.example.com --yes
```

Optional: Check every 1s the status of the cluster creation:
```
watch -n1 'kops validate cluster'
```

#### Edit the cluster with Kops  (ig stands for instance group):
This will adjust the ig configuration but not the launch configuration at AWS:
```
kops edit ig 
```
Apply these changes:
```
kops update cluster --yes
```
After that kops rolling-update cluster should show you that updates could be performed.

Execute rolling-update with the --yes parameter and your master instance size should be changed:
```
kops rolling-update --yes
```

### Further reading / notes:

- A service account provides an identity for processes that run in a Pod.
When you access the cluster (for example, using kubectl), you are authenticated by the API Server as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default)

Reference : https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

### Additional commands:

SSH to cluster:

ssh -i ~/.ssh/yourkey admin@api.yoursubdomain.example.com
