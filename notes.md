### ETCD - Commands (Optional)

(Optional) Additional information about ETCDCTL Utility

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:
```bash
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

Whereas the commands are different in version 3
```bash
etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put
```

To set the right version of API set the environment variable ETCDCTL_API command
```bash
export ETCDCTL_API=3
```

When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.

Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

```bash
--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key
```

So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:
```bash
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 
```

## Certication Tip

Here's a tip!

As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the **kubectl run** command can help in generating a YAML template. And sometimes, you can even get away with just the **kubectl run** command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the **kubectl run** command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises

Reference (Bookmark this page for exam. It will be very handy):
- https://kubernetes.io/docs/reference/kubectl/conventions/

**Create an NGINX Pod**
```bash
kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
```bash
kubectl run nginx --image=nginx --dry-run -o yaml
```

**Create a deployment**
```bash
kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
```bash
kubectl create deployment --image=nginx nginx --dry-run -o yaml
```

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
```
kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml
```
Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.


## Kubernetes Imperative Way

```bash
kubectl run --image=nginx nginx
kubectl run redis --image=redis:alpine --labels=tier=db
kubectl create deployment --image=nginx nginx
kubectl run --image=nginx custom-nginx --port=8080
kubectl run httpd --image=httpd:alpine --port 80 --expose
kubectl expose deployment nginx --port 80
kubectl expose deployment nginx --port 80 --type=NodePort nginx
kubectl expose pod redis --name redis-service --port 6379 --target-port 6379
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18
kubectl create -f nginx.yaml
kubectl replace -f nginx.yaml
kubectl replace --force -f nginx.yaml
kubectl delete -f nginx.yaml
```

## Kubernetes Declarative Way

```bash
kubectl apply -f nginx.yaml
```

## Certification Tips - Imperative Commands with Kubectl

While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save a considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

**--dry-run**: By default as soon as the command is run, the resource will be created. If you simply want to test your command, use the **--dry-run=client** option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

**-o yaml**: This will output the resource definition in YAML format on the screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

**POD**

Create an NGINX Pod
```bash
kubectl run nginx --image=nginx
```


Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
```bash
kubectl run nginx --image=nginx  --dry-run=client -o yaml
```


**Deployment**

Create a deployment
```bash
kubectl create deployment --image=nginx nginx
```


Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```


**IMPORTANT:**

**kubectl create deployment** does not have a **--replicas** option. You could first create it and then scale it using the **kubectl scale** command.


Save it to a file - (If you need to modify or add some other details)
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

You can then update the YAML file with the replicas or any other field before creating the deployment.

**Service**

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors)

Or

**kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml** (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. [https://github.com/kubernetes/kubernetes/issues/46191](You cannot pass in selectors as an option). So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)



Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
```bash
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or
```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

**Reference:**

https://kubernetes.io/docs/reference/kubectl/conventions/


## Kubectl Cheat Sheet
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## References
- https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet