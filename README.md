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

## Assigning resources (using keyword resources->requests||limits)
```bash
kubectl apply -f myresources.yaml
```
##### Above command should help create a container with specific CPU/Memory resources, to validate, we can run the following and it can display the limits in json output.
```bash
kubectl describe pod myresourcepod
``` 

## Create a secret and validate if you can see it
```bash
kubectl apply -f mysecret.yaml
kubectl get secret mysecret && kubectl describe secret mysecret 
```
##### The first command will let us create the secret and second command helps find the type and description of the secret. To validate if the secret is readable, run the following command
```bash
kubectl apply -f mysecretValidate.yaml
```
**To ensure that the value was indeed read, retrieve the logs for the pod, which is the name of the pod**
```bash
kubectl logs mysecretvalidator 
```
## Service accounts
**Are created to communicate with Kubernetes api-services with limited privileges assigned for these service accounts**
```bash
kubectl create serviceaccount myserviceaccount
```
##### To validate, just execute the following
```bash
kubectl apply -f myserviceaccount.yaml
```

## MultiContainer Pod "Workout2"
**Objective of this workout is to spin up two containers in the same pod** 
```bash
kubectl apply -f fruit-service-ambassador-config.yaml
kubectl apply -f fruit-service.yaml
```
**Since pod is a single entity, haproxy config "listen http-in" and the relevant entries "server server1 127.0.0.1:8775 ..." stats that haproxy will listen to the localhost on port 8775 (internally), since the only container port available by default is via haproxy pod exposed on containerport 80**

##### To quickly test this, we can run the following OR the second command if you are using minikube
```bash
kubectl apply -f fruite-service-check.yaml
curl $(kubectl get pod fruit-service-pod -o=custom-columns=IP:.status.podIP --no-headers):80
```

##### This just retrieves the IP address of the pod
```bash
kubectl get pod fruit-service-pod -o=custom-columns=IP:.status.podIP --no-headers
```

## Liveness and Readiness Probe
```bash
kubectl apply -f livenesspod.yaml
kubectl apply -f readinesspod.yaml
```
**Pretty self explannatory, liveness checks if the container is ready and readiness checks if the service is ready and service in the second container i.e, nginx**

## Ingesting both types of probes in an existing image 
```bash
kubectl apply -f candy-service-probes.yaml
kubectl logs candy-service 
```
**if you open the file you can see that either of the probes are introduce using httpGet method for both liveness and readiness. Run the second command to see the output**

## Debugging Pointers 
- Logs
```bash
kubectl get logs <pod-name>
```

- To get the definition of a pod into a file named "output.yaml"
```bash
kubectl get pod <pod-name> -n <namespace> -o yaml > output.yaml
```

- To list all pods in all namespaces and then finding the most resource intesnsive pod
```bash
kubectl get pods --all-namespaces
kubectl top pods --all-namespaces
```

## How to use labels and list pods/containers based on assigned labels
- Create the pods 
```bash
kubectl apply -f dev-label-pods.yaml
kubectl apply -f prod-label-pods.yaml
```

- Listing the objects based on selectors specified by "-l"
```bash
kubectl get pods -l environ=production
kubectl get pods -l environ=development
kubectl get pods -l 'environ in (production,development)'
```
**To list environments individually based on actual ecosystem and using multiple keys specified under labels like 'environ' in this example**

## Deployments 
**One of the key topics from a future perspective. Deployment is the specification of the state one wants for their deployments/pods/containers, for instance one can desire to keep 3 replicas of a pod, deployment enables that**

- Create the deployment with replication factor of 3. The second command will help you find out the number of pods running after the deployment
```bash
kubectl apply -f depoloyment-1.yaml
kubectl get pods
```

- Check the desired, current state of your deployment
```bash
kubectl get deployments -o wide
```

- Deployments can be edited and immediately take effect after being edited
```bash
kubectl edit deployment nginx-deployment 
kubectl get pods
kubectl get deployments -o wide
```

## Performing rolling updates and reversions to deployments

- Create a deployment. This is also going to create an nginx pod of image 1.7.5
```bash
kubectl apply -f deployments-rolling.yaml
```

- Check the state of deployments after the first initiation. Revision with a number is going to show more details about the pods within the deployment 
```bash
kubectl rollout history deployment/deployment-rolling 
kubectl rollout history deployment/deployment-rolling --revision=1
```

- Performing an upgrade on the deployment image
```bash
kubectl set image deployment/deployment-rolling nginx=nginx:1.7.9
kubectl rollout history deployment/deployment-rolling --revision=<5> //Last number from the history command. This will give details of the changes
```

- Performing a rollback
```bash
kubectl rollout undo deployment/deployment-rolling --to-revision=4 // Last value -1 to rever to older version
kubectl rollout history deployment/deployment-rolling 
```

**Tip: kubectl get pods -w allows you to watch the state of a pod(s) as it changes**


## Workout 3
**Prep work for hands-on of Deployment, rollout and rolling upgrade**

- Apply the candy-deployment.yaml from "Workout3" folder, this is the prep work
```bash
kubectl apply -f candy-deployment.yaml
```

- Now try performing upgrade to the image linuxacademycontent/candy-service:3 
```bash
kubectl set image deployment/candy-deployment candy-ws=linuxacademycontent/candy-service:3
kubectl rollout history deployment/candy-deployment --revision=1 // This will display the version of the app
```
**If you want to check the progress, you can run the the second command**

- If done right, it is a possibility that the upgrade deployment might succeed, but the container might not succeed. Can be checked via
```bash
kubectl get pods // OR
kubectl get deployments -o wide
```

- Try to roll back to the previous version. Either one of the options can be used.
```bash
kubectl rollout undo deployment/candy-deployment // OR
kubectl rollout undo deployment/candy-deployment --to-revision=1
```

- Check the status of deployment
```bash
kubectl rollout status deployment/candy-deployment
```

## Cronjob

- To deploy a cronjob and schedule an activity

```bash
kubectl apply -f cronjob.yaml
```

- To list the cronjobs. Pay close heed to the "ACTIVE" colum count and "LAST SCHEDULE" which should show the last execution and number of iterations
```bash
kubectl get cronjobs
```

## Services

- Create a deployment to work with first. This should create 3 replicas for nginx 
```bash
kubectl apply -f service-deployment.yaml
``` 

- Create a service named "my-service" of type "ClusterIP" that tags with app named "nginx" as we used in our deployment
```bash
kubectl apply -f myu-service.yaml
```

- Assuming the deployment is successful, use the first command to list the newly created service and then the next one to list the end points
```bash
kubectl get svc // Short for services
kubectl get ep // Short for endpoints OR
kubectl get ep my-service // To list the endpoints as specified in my-service
```

- Another crucial service is "NodePort" which exposes the port and IP (if specified or auto allocated) for external world. I removed the ClusterIP service to ensure that there is no collission. 
```bash
kubectl apply -f my-nodeport.yaml
``` 

- Now list the Cluster IP allocated by the service "my-nodeport-service" listed under heading "CLUSTER-IP"
```bash
kubectl get svc
curl <cluster-ip>:80
```
**I had some confusion figuring this out, so I used googles reference and used this command to figure out**
```bash
kubectl expose deployment/nginx-service-deployment --type="NodePort" --port=80 //This creates a service by the same name as deployment
kubectl get svc
kubectl get svc -o yaml nginx-service-deployment > <file>.yaml // Write it to a file to see what all is specified
```

## Network Policy
**One of the prereqs is to install the networking policies as specified in kubernetes documentation https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/**

- Setup prerequisite for netowrk policy to work
```bash
curl -O ttps://docs.projectcalico.org/manifests/canal.yaml && kubectl apply -f canal.yml
```

- Create a pod (and/or) deployment which you'd want to access // This would be before network policy is implemented
```bash
kubectl apply -f securenginxpod.yaml 
kubectl apply -f netpolicy-server-deployment
```

- Create a client to access the service(s) from 
```bash
kubectl apply -f insecure-client.yaml
```

- Get the IPs of the serving pod i.e. in our case it is either "securenginx" Or "np-server-deployment-???????". Note down the IP and run the second command to test out
```bash
kubectl get pods -o wide
kubectl exec insecure-client-pod -- curl <ip of the np-server> // Should return the nginx welcome page
```

- Apply the network policy 
```bash
kubectl apply -f network-policy.yaml
```

**After this application, accessing the pods via curl, as used before, should fail or timeout**

-- To fix this, we need to fix our client by adding the label within insecure-client-pod.yaml and redeploy














 


