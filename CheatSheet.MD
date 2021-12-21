## My Cheat Sheet
```bash
kubectl expose deployment/<deployment-name> --type="NodePort" --port 8080 // Exposing cluster IP using NodePort 
kubectl get pods -l app=kubernetes-bootcamp // Retrieve a given label from a pod or a service
kubectl label pods POD_NAME version=1 // Assign label to a running pod  
kubectl get pods -o go-template --template '{{range.items}}{{.metadata.name}}{{"\n"}}{{end}}' //Retrieve pod names
kubectl run tmp --restart=Never --rm --image=busybox -i -- wget -O- 10.12.2.15 //Run a wget to validate accessibility of a running pod
kubectl run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 10.12.2.15 //Run curl to check accessibility
kubectl exec -ti $POD_NAME -- curl localhost:8080 // Run an interactive pod and check if its listening from within
kubectl get rs //to list the total replica sets
kubectl scale deployments/kubernetes-bootcamp --replicas=4 //Increase replicasets of a given deployment 
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2  //Change image of a deployment 
kubectl get services/kubernetes-bootcamp -o go-template --template='{{(index .spec.ports 0).nodePort}}' // To extract the nodePort from a service
kubectl rollout status deployments/kubernetes-bootcamp //Check rollout status of an upgrade or change in deployments
kubectl rollout undo deployments/kubernetes-bootcamp //Undo the last rollout for a deployment 
kubectl get pods -l app=examplehttpapp -o go-template='{{(index .items 0).metadata.name}}' // Extract pod name by label and go-template
kubectl top node // To sort the biggest resource consuming node
kubectl top pod // To sort and list largest resource consuming pod
kubectl -n pluto describe deploy project-23-api | grep Liveness // Get deployment details to see if liveness exists
kubectl -n namespace get ep // To list all the endpoints for deployments within a given namespace 
```bash
