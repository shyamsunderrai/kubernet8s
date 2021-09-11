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


## Testing Security Context 
#### On kubernetes node, host in case you are using minikube, run the following  
```bash
sudo useradd -u 2000 user0 
sudo groupadd -g 3000 group0
sudo useradd -u 2001 user1
sudo groupadd -g 3001 group1
sudo mkdir /etc/message && echo "Hello World" | sudo tee -a /etc/message/message.txt
sudo chown 2000:3000 /etc/message/message.txt && sudo chmod 640 /etc/message/message.txt
```

## Review the contents of ismypodsecure.yaml and run it
```bash
kubectl apply -f ismypodsecure.yaml 
```
#### At this point you can validate if your pod was able to read the contents of the mounted file, using
##### This should ideally emit "Hello World" 
```bash
kubectl logs ismypodsecure  
```
##### This is why it would be able to read, because the process is running as "root" user
```bash
kubectl exec ismypodsecure -- ps
```

## To enforce security, review contents of mypodissecure.yaml and change the runAsUser value to 2001 and fsGroup to 3001
```bash
kubectl apply -f mypodissecure.yaml
```
##### You can now validate that the container won't run and logs for your pod would show error 
##### To fix the error, just swap the runAsUser with 2000 as value and fsGroup with 3000 as the value