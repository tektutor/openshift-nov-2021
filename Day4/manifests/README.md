### Cloning the TekTutor openshift repository as rps user
```
cd ~
git clone https://github.com/tektutor/openshift-nov-2021.git
```

### Creating an nginx deployment in declarative style

Make sure your old nginx deployments are deleted
```
kubectl get deploy
kubectl delete deploy/nginx
```

```
cd ~/openshift-nov-2021
git pull
cd Day4/manifests
kubectl apply -f nginx-deployment.yml
```
You may list the deploy,rs,po as shown below
```
kubectl get deploy,rs,po
```

### Creating a NodePort external service declarative way
Make sure you have deleted the existing nginx nodeport service, just in case 
```
kubectl delete svc/nginx
```

```
cd ~/openshift-nov-2021
git pull
cd Day4/manifests
kubectl apply -f nginx-nodeport-service.yml
```
You may list and describe details about the nodeport service you just now created
```
kubectl describe svc/nginx
```
You may then acess the NodePort service as
```
curl http://master:30958
curl http://worker-1:30958
curl http://worker-2:30958
```

### Creating a ClusterIP internal service declarative way
Make sure you have deleted the existing nginx nodeport service, just in case 
```
kubectl delete svc/nginx
```

```
cd ~/openshift-nov-2021
git pull
cd Day4/manifests
kubectl apply -f nginx-clusterip-service.yml
```
You may list and describe details about the clusterip service you just now created
```
kubectl describe svc/nginx
```
You may then acess the ClusterIP service as
```
kubectl exec -it your-nginx-podname sh
curl http://nginx:80
```
