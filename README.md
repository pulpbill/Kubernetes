# k8s-basics
Cheatsheet and basic concepts about Kubernetes (Including KOPS and Helm)

## Kubectl

### Important:

-Remember to always specify your namespace for most of kubectl commands execution.

-Though I use yaml in the examples, kubectl also takes jsons. 

### Install kubectl at Amazon Linux2 AMI (RedHat based): 

Use this script: https://github.com/pulpbill/daily-bash-scripts/blob/master/kubectl.sh

### Useful commands: 

Create a resource from file (or STDIN):
```
kubectl create -f myfile.yaml
```
Apply a configuration to a resource (will create if it doesn't exists):
```
kubectl apply -f myfile.yaml
```
List objects:
```
kubectl get <object> -n <your-namespace>
```  
Get detailed info about a kind of objects:
```
kubectl describe <object> -n <your-namespace>
```
Get detailed info about a specific object (I picked pod in this example):
```
kubectl describe pod <your-pod> -n <your-namespace>
```
Check pod logs (IE: to check actual nginx log files inside our container=pod):
```
kubectl logs <pod-ID> -f -n <your-namespace>
```
Connect to the shell’s container:
```
kubectl exec -it <pod-ID> <your-containers-shell> -n <your-namespace>
```
Delete (terminate) a pod:
```
kubectl delete pod <pod-ID> -n <your-namespace>
```
Edit an object:
```
kubectl edit pod/<pod-ID> -n <your-namespace>
```
Create a configmap:
```
kubectl create configmap <cm-name> --from-file=<my-folder> -n <your-namespace>
```
Get info of a configmap:
```
kubectl describe cm <configmap-name> -n <your-namespace>
```
Delete:
```
kubectl delete cm <configmap-name> -n <your-namespace>
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
Add this to a json file:
```
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "monitoring",
    "labels": {
      "name": "monitoring"
    }
  }
}
```
## KOPS

Creating a K8s cluster with kops:

I have a user (AWS user created via IAM) with S3/R53/EC2/VPC/IAM admin access and created a set of keys.

Create ssh key pairs with ssh-keygen and then set it:
```
kops create secret --name yoursubdomain.example.com sshpublickey admin -i ~/.ssh/yourkey.pub
```

1. Install Kops (you must have kubectl installed):
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64 && chmod +x kops-linux-amd64 && mv kops-linux-amd64 /usr/local/bin/kops
```

2. Delegate authority for the subdomain you choose (get your subdomain nameservers and add a NS record to the parent hosted zone that contains these 4 ns records so it can know where to resolve your subdomain).

3. Create a S3 bucket (at your AWS consola or via CLI) and set the environment variable so Kops can manage files:
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

Optional: Check every 5s the status of the cluster creation:
```
watch -n 5 kops validate cluster
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
## HELM / Tiller

1. Install Helm:
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh && chmod 700 get_helm.sh && ./get_helm.sh
```

2. Create Tiller serviceaccount:
```
kubectl --namespace kube-system create serviceaccount tiller 
```

3. Bind the tiller serviceaccount to the Kubernetes cluster-admin role:
```
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

4. Install Tiller in your Kubernetes cluster:
```
helm init --service-account tiller
```

Optional, run this and should Tiller's pod running at kube-system namespace:
```
kubectl get pods --namespace kube-system
```

Bonus track, set up tiller account at once, use it carefully:
```
kubectl --namespace kube-system create serviceaccount tiller && kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller && helm init --service-account tiller
```


### Charts:

Once you have everything set up, go look for a chart to test on your K8s cluster (IE: Kibana):
```
helm search | grep kibana
```
Install a chart:
```
helm install stable/kibana --name kibana-stable
```
Delete a chart:
```
helm delete kibana-stable
```
Test a chart with debug:
```
helm install --dry-run --debug mychart
```
Install a chart at monitoring namespace:
```
helm install stable/kibana --name kibana-stable --namespace monitoring
```

#### Note:
I picked the release name passing --name kibana-stable Otherwise, Helm would have picked one for us.

In case of tiller issues: http://zero-to-jupyterhub.readthedocs.io/en/latest/setup-helm.html

Reference:

https://github.com/helm/helm/blob/master/docs/rbac.md

https://docs.gitlab.com/ee/install/kubernetes/preparation/tiller.html


### Further reading / notes:

- A service account provides an identity for processes that run in a Pod.
When you access the cluster (for example, using kubectl), you are authenticated by the API Server as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default)

Reference : https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

### Additional commands:

SSH to cluster:

ssh -i ~/.ssh/yourkey admin@api.yoursubdomain.example.com
