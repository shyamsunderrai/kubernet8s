CKAD Practice Exam - Part 2

This lab is designed to help prepare for the kinds of tasks and scenarios encountered during the Certified Kubernetes Application Developer (CKAD) exam. In this lab, we will practice working with multi-container pods by creating a pod that implements an adapter design pattern. We will create a pod that generates some log output and uses an adapter pod to format the output.

Solution

Open a terminal session and log in to Kube Master server via SSH using the credentials listed on the lab page:

ssh cloud_user@<PUBLIC_IP>
Hint: When copying and pasting code into Vim from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes.

Create a ConfigMap to Store the fluentd Configuration
Become root:

sudo -i
View the contents of the config file:

cat /usr/ckad/fluent.conf
Create a descriptor for the ConfigMap:

vim fluentd-config.yml
Enter the following, making sure the indenting is correct (remember the hint at the beginning of the lab guide):

apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      type tail
      format none
      path /var/log/1.log
      pos_file /var/log/1.log.pos
      tag count.format1
    </source>

    <source>
      type tail
      format none
      path /var/log/2.log
      pos_file /var/log/2.log.pos
      tag count.format2
    </source>

    <match **>
      @type file
      path /var/logout/count
      time_slice_format %Y%m%d%H%M%S
      flush_interval 5s
      log_level trace
    </match>
Save and exit the file by pressing Escape followed by :wq!.

Create the ConfigMap in the cluster:

kubectl apply -f fluentd-config.yml
Create the Pod Descriptor
Open the /usr/ckad/adapter-pod.yml descriptor file:

vim /usr/ckad/adapter-pod.yml
Enter the following, making sure the indenting is correct:

apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: adapter
    image: k8s.gcr.io/fluentd-gcp:1.30
    env:
    - name: FLUENTD_ARGS
      value: -c /fluentd/etc/fluent.conf
    volumeMounts:
    - name: varlog
      mountPath: /var/log
    - name: config-volume
      mountPath: /fluentd/etc
    - name: logout
      mountPath: /var/logout
  volumes:
  - name: varlog
    emptyDir: {}
  - name: config-volume
    configMap:
      name: fluentd-config
  - name: logout
    hostPath:
      path: /usr/ckad/log_output
Create the Pod in the Cluster and Make Sure It Is Working
Create the pod:

kubectl apply -f /usr/ckad/adapter-pod.yml
Verify the pod starts up:

kubectl get pod counter
Verify everything is working by checking the output directory on the worker node. Log in to the worker from the master:

ssh cloud_user@10.0.1.102
The password is the same as the one we used to log in to the Kube Master server.

Make sure we see fluentd output files in the output directory:

ls /usr/ckad/log_output
Conclusion

Congratulations on successfully completing this hands-on lab!