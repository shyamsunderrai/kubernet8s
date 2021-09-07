# kubernet8s
Tutorial for Kubernetes

### Creating a simple pod using nginx.yaml file
```bash
kubectl apply -f nginx.yaml 
```
### Create a config map named myconfigmap with two keys named myKey and myKey2
```bash
kubectl apply -f myconfigmap.yaml
```
### Create a pod and use the keys from cofig map created in myconfigmap
```bash
kubectl apply -f myconfigmappod.yaml
```

