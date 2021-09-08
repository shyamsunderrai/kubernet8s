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
### Validate if the variables named MY_VAR and MY_VAR2 took effect
```bash
kubectl logs myconfigmappod 
```
_Above command should display the values specified in data_

## Creating a pod that can read config values using volume and volume mounts
```bash
kubectl apply -f myconfigvolumespod.yaml 
```
#### The sample volumeMount contains the name of the actual volume (can be whatever we named it within volumes). Then volumes should actually contain the name of the config map we created, hence, "myconfigmap" in our case.