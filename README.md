### Practice questions for CKA exam
These questions were collected and used to prepare for CKA exam.

#### - Upgrade the current version of kubernetes from 1.20 to 1.21.0 exactly using the kubeadm utility. Make sure that the upgrade is carried out one node at a time starting with the master node. To minimize downtime, the deployment nginx-deploy should be rescheduled on an alternate node before upgrading each node. Upgrade master node first and drain node worker01 before upgrading it. Pods for ```nginx-deploy``` should run on the master node subsequently.
<details><summary>show</summary>
<p>
On the master node:

```
root@master:~# kubectl drain master --ignore-daemonsets
root@master:~# apt update
root@master:~# apt-get install -y kubeadm=1.21.0-00
root@master:~# kubeadm upgrade plan v1.21.0
root@master:~# kubeadm upgrade apply v1.21.0
root@master:~# apt-get install -y kubelet=1.21.0-00
root@master:~# kubectl uncordon master 
root@master:~# kubectl drain worker01 --ignore-daemonsets
```
On the worker01 node:

```
apt update
apt-get install -y kubeadm=1.21.0-00
kubeadm upgrade node --kubelet-version v1.21.0
apt-get install -y kubelet=1.21.0-00
```
Back on the master node:

```
root@master:~# kubectl uncordon worker01
root@master:~# kubectl get pods -o wide | grep nginx (make sure this is scheduled on node)
```
</p>
</details>

#### - Print the names of all deployments in the admin namespace in the following format: 
```
DEPLOYMENT	  CONTAINER_IMAGE  READY_REPLICAS NAMESPACE
<deployment name> <container image> <ready replica count> <namespace>
```
#### The data should be sorted by the increasing order of the deployment name.
#### Example:
````
DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
deploy0    nginx:alpine    1              admin
````
#### Write the result to the file /opt/admin_data.
<details><summary>show</summary>
<p>
Run the below command to get the correct output:

```
$ kubectl -n admin get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin_data
```
</p>
</details>

#### - A kubeconfig file called ```admin.kubeconfig``` has been created in ```/root```. There is something wrong with the configuration. Troubleshoot and fix it.
<details><summary>show</summary>
<p>
Make sure the port for the kube-apiserver is correct. Check if the port is 6443.
Run the below command to know the cluster information:

```
$ kubectl cluster-info --kubeconfig /root/admin.kubeconfig
```
</p>
</details>

#### - Create a new deployment called ```nginx-deploy```, with image ```nginx:1.16``` and 1 replica. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.
<details><summary>show</summary>
<p>
Make use of the kubectl create command to create the deployment and explore the --record option while upgrading the deployment image.
Run the below command to create a deployment nginx-deploy:

```
$ kubectl create deployment  nginx-deploy --image=nginx:1.16
```

Run the below command to update the new image for nginx-deploy deployment and to record the version:

```
$ kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
```
</p>
</details>

#### - A new deployment called alpha-deploy has been deployed in the alpha namespace. However, the pods are not running. Troubleshoot and fix the issue. The deployment should make use of the persistent volume ```alpha-pv``` to be mounted at ```/var/lib/mysql``` and should use the environment variable ```MYSQL_ALLOW_EMPTY_PASSWORD=1``` to make use of an empty root password. Important: Do not alter the persistent volume.
<details><summary>show</summary>
<p>
Use the command kubectl describe and try to fix the issue. Get the data of the PV alpha-pv and create appropriate PVC in namespace alpha.
Solution manifest file to create a pvc called alpha-pvc as follows:

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpha-pvc
  namespace: alpha
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow
```
</p>
</details>

#### - Take the backup of ETCD at the location ```/opt/etcd-backup.db``` on the master node.
<details><summary>show</summary>
<p>
Take a help of command etcdctl snapshot save --help options. Use documentaion which provide exact example.

```
export ETCDCTL_API=3
etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db
```
</p>
</details>

#### - Create a pod called ```secret-1401``` in the admin1401 namespace using the busybox image. The container within the pod should be called secret-admin and should sleep for 4800 seconds. The container should mount a read-only secret volume called secret-volume at the path ```/etc/secret-volume```. The secret being mounted has already been created for you and is called ```dotfile-secret```.
<details><summary>show</summary>
<p>

Use the command kubectl run to create a pod definition file. Add secret volume and update container name in it. Alternatively, run the command:

```
kubectl run secret-1401 --image=busybox --dry-run=client -oyaml --command -- sleep 4800 > admin.yaml
```

Add the secret volume and mount path to create a pod called secret-1401 in the admin1401 namespace as follows:

```
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-1401
  name: secret-1401
  namespace: admin1401
spec:
  volumes:
  - name: secret-volume
    # secret volume
    secret:
      secretName: dotfile-secret
  containers:
  - command:
    - sleep
    args:
    - "4800"
    image: busybox
    name: secret-admin
    # volumes' mount path
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```
The same example is given in documentation, just copy/paste POD definition file  
</p>
</details>

#### - Deploy a pod named ```nginx-pod``` using the ```nginx:alpine``` image.
<details><summary>show</summary>
<p>
Use the command: 

```
kubectl run nginx-pod --image=nginx:alpine
```
</p>
</details>

#### - Deploy a messaging pod using the ```redis:alpine``` image with the labels set to ```tier=msg```.
<details><summary>show</summary>
<p>
Use the command:

```
kubectl run messaging --image=redis:alpine -l tier=msg
```
</p>
</details>

#### - Create a namespace named ```apx-1234```.
<details><summary>show</summary>
<p>
Run the command: 

```
kubectl create ns apx-1234
```
</p>
</details>

#### - Get the list of nodes in JSON format and store it in a file at ```/opt/outputs/nodes.json```
<details><summary>show</summary>
<p>
Run the command: 

```
kubectl get nodes -o json > /opt/outputs/nodes.json
```
</p>
</details>

#### - Create a service ```messaging-service``` to expose the messaging application within the cluster on port 6379. Use imperative commands.
<details><summary>show</summary>
<p>
Run the command: 

```
kubectl expose pod messaging --port=6379 --name messaging-service
```
</p>
</details>

#### - Create a deployment named ```web-app``` using the image ```nginx``` with 2 replicas.
<details><summary>show</summary>
<p>
Run the command: 

```
kubectl create deployment web-app --image=nginx --replicas=2
```
</p>
</details>

#### - Create a static pod named ```static-box``` on the master node that uses the ```busybox``` image and the command sleep 1000.
<details><summary>show</summary>
<p>
Create a pod definition file in the manifests directory. For that use command 

```
kubectl run --restart=Never --image=busybox static-box --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static.yaml
```
</p>
</details>

#### - Create a POD in the ```admin``` namespace named ```temp-bus``` with the image ```redis:alpine```.
<details><summary>show</summary>
<p>
Run the command: 

```
kubectl run temp-bus --image=redis:alpine --namespace=admin --restart=Never
```
</p>
</details>

#### - Expose the ```web-app``` as service ```web-app-service``` application on port ```30082``` on the nodes on the cluster. The web application listens on port ```8080```.
<details><summary>show</summary>
<p>
Run the command: 

```
kubectl expose deployment web-app --type=NodePort --port=8080 --name=web-app-service --dry-run=client -o yaml > web-app-service.yaml 
```
to generate a service definition file. Now, in generated service definition file add the nodePort field with the given port number under the ports section and create a service.
</p>
</details>

#### - Use JSON PATH query to retrieve the ```osImages``` of all the nodes and store it in a file ```/opt/outputs/nodes_os.txt```. The ```osImages``` are under the nodeInfo section under status of each node.
<details><summary>show</summary>
<p>
Run the command: 

```
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt
```
</p>
</details>

#### - Create a Persistent Volume with the given specification.

```
* Volume Name: pv-test
* Storage: 100Mi
* Access modes: ReadWriteMany
* Host Path: /pv/data-test
```

<details><summary>show</summary>
<p>
Solution manifest file to create a persistent volume pv-test as follows:

```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-test
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
      path: /pv/data-test
```
</p>
</details>

#### - Create a Pod called ```redis-storage``` with image ```redis:alpine``` with a Volume of type emptyDir that lasts for the life of the Pod. Spec sare:

```
* Pod named 'redis-storage' created
* Pod 'redis-storage' uses Volume type of emptyDir
* Pod 'redis-storage' uses volumeMount with mountPath = /data/redis
```
<details><summary>show</summary>
<p>
Use the command kubectl run and create a pod definition file for redis-storage pod and add volume.
Alternatively, run the command:

```
kubectl run redis-storage --image=redis:alpine --dry-run=client -oyaml > redis-storage.yaml
```
and add volume emptyDir in it.
Solution manifest file to create a pod redis-storage as follows:

```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: redis-storage
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: temp-volume
  volumes:
  - name: temp-volume
    emptyDir: {}
```
</p>
</details>

#### - Create a new pod called ```super-user``` with image ```busybox```. Allow the pod to be able to set system_time. The container should sleep for 4800 seconds.
<details><summary>show</summary>
<p>
Solution manifest file to create a pod super-user-pod as follows:

```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: super-user
  name: super-user
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox
    name: super-user
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```
</p>
</details>
 
#### - A pod definition file is created at ```/root/use-pv.yaml```. Make use of this manifest file and mount the persistent volume called ```pv-1```. Ensure the pod is running and the PV is bound. Specs are:

```
mountPath: /data
persistentVolumeClaim Name: my-pvc
```
Add a persistentVolumeClaim definition to pod definition file.
<details><summary>show</summary>
<p>
Solution manifest file to create a pvc my-pvc as follows:

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
       storage: 10Mi
```

And then, update the pod definition file as follows:

```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
    - mountPath: "/data"
      name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc
Finally, create the pod by running: kubectl create -f /root/use-pv.yaml
```
</p>
</details>

#### - Create a new user called ```john```. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the ```development``` namespace . The private key exists in the location: ```/root/john.key``` and csr at ```/root/john.csr```. Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.
<details><summary>show</summary>
<p>
Solution manifest file to create a CSR as follows:

```
---
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: <create a requested data using cmd 'cat <csr file_name> | base64 | tr -d "\n"'
  usages:
  - digital signature
  - key encipherment
  - client auth
  groups:
  - system:authenticated
```
To approve this certificate, run: 

```
kubectl certificate approve john-developer
```

Next, create a role developer and rolebinding developer-role-binding, run the command:

```
kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development
kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development
```
To verify the permission from kubectl utility tool:

```
kubectl auth can-i update pods --as=john --namespace=development
```
</p>
</details>

#### - Create a nginx pod called ```nginx-resolver``` using image ```nginx```, expose it internally with a service called ```nginx-resolver-service```. Test that you are able to look up the service and pod names from within the cluster. Use the image ```busybox``` for dns lookup. Record results in ```/root/nginx.svc``` and ```/root/nginx.pod```.
<details><summary>show</summary>
<p>

Use the command kubectl run and create a nginx pod and busybox pod. Resolve it, nginx service and its pod name from busybox pod. To create a pod nginx-resolver and expose it internally:

```
kubectl run nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP
```

Then create a pod test-nslookup. Test that you are able to look up the service and pod names from within the cluster:

```
kubectl run test-nslookup --image=busybox --rm -it --restart=Never -- nslookup nginx-resolver-service
kubectl run test-nslookup --image=busybox --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/nginx.svc
```
Get the IP of the nginx-resolver pod and replace the dots(.) with hyphon(-) which will be used below.

```
kubectl get pod nginx-resolver -o wide
kubectl run test-nslookup --image=busybox --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/nginx.pod
```
</p>
</details>

#### - Create a static pod on ```worker01``` called ```nginx-critical``` with image ```nginx``` and make sure that it is recreated/restarted automatically in case of a failure.
<details><summary>show</summary>
<p>
Use /etc/kubernetes/manifests as the Static Pod path for example
To create a static pod called nginx-critical by using below command:

```
kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > static.yaml
```
Copy the contents of this file or use scp command to transfer this file from master to worker01 node.

```
root@master:~# scp static.yaml worker01:/root/
```

To know the IP Address of the worker01 node:

```
root@master:~# kubectl get nodes -o wide
```
To rerform SSH:

```
root@master:~# ssh worker01
```
OR

```
root@master:~# ssh <IP of worker01>
```
On worker01 node:
Check if static pod directory is present which is /etc/kubernetes/manifests, if it's not present then create it.

```
root@worker01:~# mkdir -p /etc/kubernetes/manifests
```
Add that complete path to the staticPodPath field in the kubelet config.yaml file.

```
root@worker01:~# vi /var/lib/kubelet/config.yaml
```
Now, move/copy the static.yaml to path /etc/kubernetes/manifests/.

```
root@worker01:~# cp /root/static.yaml /etc/kubernetes/manifests/
```
Go back to the master node and check the status of static pod:

```
root@worker01:~# exit
logout
root@master:~# kubectl get pods -o wide
```
</p>
</details>

#### - Create a new service account with the name ```pvviewer```. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called ```pvviewer-role``` and ClusterRoleBinding called ```pvviewer-role-binding```. Next, create a pod called ```pvviewer``` with the image ```redis``` and serviceAccount: ```pvviewer``` in the default namespace.
<details><summary>show</summary>
<p>
Create a service account pvviewer:

```
kubectl create serviceaccount pvviewer
```
To create a clusterrole:

```
kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
```
To create a clusterrolebinding:

```
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
```
Solution manifest file to create a new pod called pvviewer as follows:

```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pvviewer
  name: pvviewer
spec:
  containers:
  - image: redis
    name: pvviewer
  # Add service account name
  serviceAccountName: pvviewer
```
</p>
</details>

#### - List the InternalIP of all nodes of the cluster. Save the result to a file /root/node_ips. Answer should be in the format: ```InternalIP of master<space>InternalIP of worker01``` (in a single line).
<details><summary>show</summary>
<p>
Explore the JSON PATH

```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}' > /root/node_ips
```
</p>
</details>

#### - Create a pod called ```multi-pod``` with two containers.

```
Container 1, name: alpha, image: nginx
Container 2: name: beta, image: busybox, command: sleep 4800
Environment Variables:
container 1:
name: alpha
Container 2:
name: beta
```
<details><summary>show</summary>
<p>
Solution is:

```
---
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - image: nginx
    name: alpha
    env:
    - name: name
      value: alpha
  - image: busybox
    name: beta
    command: ["sleep", "4800"]
    env:
    - name: name
      value: beta
```      
</p>
</details>

### - Create a Pod called ```non-root-pod```, image ```redis:alpine```
Specs are:

```
runAsUser: 1000
fsGroup: 2000
```
<details><summary>show</summary>
<p>
Solution manifest file to create a pod called non-root-pod as follows:

```
---
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: non-root-pod
    image: redis:alpine
```
Verify the user and group IDs by using below command:

```
kubectl exec -it non-root-pod -- id
```
</p>
</details>

#### - We have deployed a new pod called ```np-test``` and a service called ```np-test-service```. Incoming connections to this service are not working. Troubleshoot and fix it. Create NetworkPolicy, by the name ```ingress-to-nptest``` that allows incoming connections to the service over port 80. Important: Don't delete any current objects deployed.
<details><summary>show</summary>
<p>
Solution manifest file to create a network policy ingress-to-nptest as follows:

```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```
</p>
</details>

#### - Taint the worker node ```worker01``` to be Unschedulable. Once done, create a pod called ```dev-redis```, image ```redis:alpine```, to ensure workloads are not scheduled to this worker node. Finally, create a new pod called ```prod-redis``` and image ```redis:alpine``` with toleration to be scheduled on ```worker01```.

```
key: env_type, value: production, operator: Equal and effect: NoSchedule
```
<details><summary>show</summary>
<p>
To add taints on the worker01 worker node:

```
kubectl taint node worker01 env_type=production:NoSchedule
```
Now, deploy dev-redis pod and to ensure that workloads are not scheduled to this worker01 worker node.

```
kubectl run dev-redis --image=redis:alpine
```
To view the node name of recently deployed pod:

```
kubectl get pods -o wide
```
Solution manifest file to deploy new pod called prod-redis with toleration to be scheduled on worker01 worker node.

```
---
apiVersion: v1
kind: Pod
metadata:
  name: prod-redis
spec:
  containers:
  - name: prod-redis
    image: redis:alpine
  tolerations:
  - effect: NoSchedule
    key: env_type
    operator: Equal
    value: production     
```    
To view only prod-redis pod with less details:

```
kubectl get pods -o wide | grep prod-redis
```
</p>
</details>

#### - Create a pod called ```hr-pod``` in ```hr``` namespace belonging to the ```production``` environment and ```frontend``` tier, image ```redis:alpine```. Use appropriate labels and create all the required objects if it does not exist in the system already. Create a namespace if it doesn't exist:
<details><summary>show</summary>
<p>

```
kubectl create namespace hr
```
and then create a hr-pod with given details:

```
kubectl run hr-pod --image=redis:alpine --namespace=hr --labels=environment=production,tier=frontend
```
</p>
</details>

#### - We have created a new deployment called ```nginx-deploy```. Scale the deployment to 3 replicas. Has the replica's increased? Troubleshoot the issue and fix it.
<details><summary>show</summary>
<p>
Use the command kubectl scale to increase the replica count to 3.

```
kubectl scale deploy nginx-deploy --replicas=3
```
The controller-manager is responsible for scaling up pods of a replicaset. If you inspect the control plane components in the kube-system namespace, you will see that the controller-manager is not running.

```
kubectl get pods -n kube-system
```
The command running inside the controller-manager pod is incorrect. That's 'added' error.
After fix all the values in the file and wait for controller-manager pod to restart.
At last, inspect the deployment by using below command:

```
kubectl get deploy
```
</p>
</details>

#### - Create a job that calculates pi to 2000 decimal points using the container with the image named ```perl``` and the following commands issued to the container:  ```["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]```. Once the job has completed, check the logs to and export the result to ```pi-result.txt```.
<details><summary>show</summary>
<p>

```
kubectl job pi2000 --image=perl -o yaml --dry-run > pi2000.yaml
```
Then edit the file, edit the name, remove any ID references and include the command argument under container spec.

```
command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
```
So the manifest looks like:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi2000
spec:
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: perl
        name: pi2000
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
        resources: {}
      restartPolicy: Never
```
Then crate an object:

```
kubectl -f pi2000.yaml
```
And get the output from the logs and export them to text file

```
kubectl logs pi2000 > pi-result.txt
```
</p>
</details>

#### - Create a yaml file called ```nginx-deploy.yaml``` for a deployment of three replicas of nginx, listening on the container's port 80. They should have the labels ```role=webserver``` and ```app=nginx```. The deployment should be named ```nginx-deploy```. Expose the deployment with a load balancer and use a curl statement on the IP address of the load balancer 
to export the output to a file titled ```output.txt```.

<details><summary>show</summary>
<p>

```
kubectl run nginx-deploy --labels="role=webserver,app=nginx" --image=nginx --replicas=3 --port=80 -o yaml > nginx-deployment.yaml
```

Expose the deployment with a loadbalancer type, call it nginx-service

```
kubectl expose deployment nginx-deploy --type=LoadBalancer --name=nginx-service
```

Use a curl statement that connects to the IP endpoint of the nginx-service and save the output to a file called output.txt

```
curl IP > output.txt
```
</p>
</details>

#### - Scale the deployment you just made down to 2 replicas
<details><summary>show</summary>
<p>

```
kubectl scale deployment nginx-deploy --replicas=2
```
</p>
</details> 
 
#### - Create a pod called ```haz-docs``` with an ```nginx``` image listening on port 80. Attach the pod to ```emptyDir``` storage, mounted to /tmp in the container. Connect to the pod and create a file with zero bytes in the ```/tmp``` directory called ```my-doc.txt```. 
<details><summary>show</summary>
<p>

```
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  labels:
    run: haz-docs
  name: haz-docs
spec:
  replicas: 1
  selector:
    matchLabels:
      run: haz-docs
  strategy: {}
  template:
    metadata:
      labels:
        run: haz-docs
    spec:
      containers:
      - image: nginx
        name: haz-docs
        volumeMounts:
        - mountPath: /tmp
          name: tmpvolume
        ports:
        - containerPort: 80
        resources: {}
      volumes:
      - name: tmpvolume
        emptyDir: {}
```

```
kubectl exec -it haz-docs-5b49cb4d87-2lm5g /bin/bash
root@haz-docs-5b49cb4d87-2lm5g:/# cd /tmp/
root@haz-docs-5b49cb4d87-2lm5g:/tmp# touch my-doc.txt
root@haz-docs-5b49cb4d87-2lm5g:/tmp# ls 
my-doc.txt
```
</p>
</details>

#### - Label the worker node of your cluster with ```rack=qa```.
<details><summary>show</summary>
<p>

```
kubectl label node worker01 rack=qa
```
</p>
</details>
 
#### - Create a file called ```counter.yaml``` in your home directory and paste the following yaml into it:

```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c, 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```
#### - Start this pod. Once its logs exceed a count of 20 (no need to be precise — any time after it has reached 20 is fine), save the logs into a file in your home directory called ```count.result.txt```. 
<details><summary>show</summary>
<p>

```
kubectl apply -f counter.yaml
kubectl logs counter > count.result.txt
```
</p>
</details>

#### - Create a deployment with two replicas of ```nginx```. The container listens on port 80. It should be named ```web-dep``` and be labeled with ```tier=frontend``` with an annotation of ```AppVersion=3.4```. The containers must be running with the UID of 1000.
<details><summary>show</summary>
<p>

```
kubectl run web-dep --labels="tier=frontend" --image=nginx --replicas=2 --port=80 -o yaml > web-dep.yaml
```
Edit the file to add the annotation in the metadata section:

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  annotations:
    AppVersion: "3.4"
  creationTimestamp: 2019-03-02T18:17:19Z
  generation: 1
  labels:
    tier: frontend
```
Create the deployment with ```kubectl apply``` command.
Output the description of the deployment to the file web-dep-description.txt

```
kubectl describe deploy/web-dep > web-dep-description.txt
``` 
</p>
</details>

#### - Upgrade the image in use by the ```web-dep``` deployment to nginx:1.9.
<details><summary>show</summary>
<p>

```
kubectl --record deployment/web-dep set image deployment/web-dep nginx=nginx:1.9
```
</p>
</details>
 
#### - Roll the image in use by the web-dep deployment to the previous version. Do not set the version number of the image explicitly for this command.
<details><summary>show</summary>
<p>

```
kubectl rollout history deployment/web-dep
kubectl rollout undo deployment/web-dep
```
</p>
</details>

#### - Expose the web-dep deployment as a service using a NodePort.
<details><summary>show</summary>
<p>

```
kubectl expose deployment/web-dep --type=NodePort
```
</p>
</details>
 
#### - Configure a DaemonSet to run the ```image k8s.gcr.io/pause:2.0``` in the cluster.
<details><summary>show</summary>
<p>

```
kubectl run testds --image=k8s.gcr.io/pause:2.0 -o yaml > testds.yaml
```
Then edited it as Daemonset to get it running, you don't do replicas in a daemonset, 
it runs on all nodes
</p>
</details> 

#### - Configure the cluster to use 8.8.8.8 and 8.8.4.4 as upstream DNS servers.
<details><summary>show</summary>
<p>

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-dns
  namespace: kube-system
data:
  stubDomains: |
    {"acme.local": ["1.2.3.4"]}
  upstreamNameservers: |
    ["8.8.8.8", "8.8.4.4"]
```
</p>
</details> 

#### - An app inside a container needs the IP address of the ```web-dep``` endpoint to be passed to it as an environment variable called ```ULTIMA```. Save the yaml as ```env-ultima.yaml```.
<details><summary>show</summary>
<p>
Get the IP address of the web-dep service

```
kubectl get svc
```
Create a mainfest file:

```
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: ultima-dep
  namespace: default
spec:
  selector:
    matchLabels:
      app: ultima-app
  template:
    metadata:
      labels:
        app: ultima-app
    spec:
      containers:
      - name: pause-pod
        image: k8s.gcr.io/pause:2.0
        env:
        - name: ULTIMA
          value: 55.55.58.23
```
Finally, create a deployment

```
kubectl -f env-ultima.yaml
```
</p>
</details>
 
#### - Figure out a way to create a pod with 3 replicas using the the ```nginx``` container that can have pods deployed on a worker node and the master node if needed.
<details><summary>show</summary>
<p>

```
kubectl get nodes
kubectl describe node MASTERNODE
```
Notice the taint on the master node:

```
Taints:             node-role.kubernetes.io/master:NoSchedule
```
Add the toleration to the yaml file:

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Equal"
        effect: "NoSchedule"
```        
</p>
</details>
 
#### - Create a yaml file called db-secret.yaml for a secret called db-user-pass. The secret should have two fields: a username and password. The username should be ```superadmin``` and the password should be ```imamazing```.
<details><summary>show</summary>
<p>

```
echo -n 'superadmin' > ./username.txt
echo -n 'imamazing' > ./password.txt
```                 
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt -o yaml > db-secret.yaml

apiVersion: v1
data:
  password.txt: aWhlYXJ0a2l0dGVucw==
  username.txt: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: 2019-03-03T00:21:16Z
  name: db-user-pass
  namespace: default
  resourceVersion: "30182"
  selfLink: /api/v1/namespaces/default/secrets/db-user-pass
  uid: 42b979da-3d4a-11e9-8f41-06f514f6b3f0
type: Opaque
```
</p>
</details>
