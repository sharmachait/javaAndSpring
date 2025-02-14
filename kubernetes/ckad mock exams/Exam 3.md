###### Q1  
In the `ckad-job` namespace, create a cronjob named `simple-python-job` to run every 30 minutes to list all the running processes inside a container that used `python` image (the command needs to be run in a shell).
In Unix-based operating systems, `ps -eaf` can be use to list all the running processes.
###### solution
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-python-job
  namespace: ckad-job
spec:
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - command: ["/bin/sh", "-c", "ps -eaf"]
            image: python
            name: simple-python-job
          restartPolicy: OnFailure
  schedule: '*/30 * * * *'
```
###### Q2
In the `ckad-pod-design` namespace, start a `ckad-nginx-uahsbcbdkl` pod running the `nginx:1.17` image.
Configure the pod with a label:  
```
TRAINER: KODEKLOUD
```
The pod should not be restarted in any case if it has already exited.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    TRAINER: KODEKLOUD
  name: ckad-nginx-uahsbcbdkl
  namespace: ckad-pod-design
spec:
  containers:
  - image: nginx:1.17
    name: ckad-nginx-uahsbcbdkl
```
###### Q3
In the `ckad-pod-design` namespace, create a pod called `ckad-busybox-gsqodeuykj` that runs a `busybox:1.28` image.  
The pod's container should be named `busybox-server`; the container will sleep for `3600` seconds.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ckad-busybox-gsqodeuykj
  name: ckad-busybox-gsqodeuykj
  namespace: ckad-pod-design
spec:
  containers:
  - image: busybox:1.28
    name: busybox-server
    command: ["/bin/sh","-c","sleep 3600"]
```
###### Q4
In the `ckad-multi-containers` namespace, create a pod named `cuda-pod`, which has 2 containers matching the below requirements:  
- The first container named `alpha` runs `alpine` image and has `release=stable` environment variable.  
- The second container named `beta` runs `nginx:1.17` image and is exposed at port `8080`.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - env:
    - name: release
      value: stable
    image: alpine
    name: alpha
    command: ["/bin/sh","-c","sleep 3600"]
  - image: nginx:1.17
    name: beta
    ports:
      containerPort: 8080
```
###### Q5
In the `ckad-pod-design` namespace, we created a pod named `custom-nginx` that runs the `nginx:1.17` image.  
Take appropriate actions to update the `index.html` page of this NGINX container with below value instead of default NGINX welcome page:  
```
Welcome to CKAD mock exams!
```
**NOTE:** By default NGINX web server default location is at `/usr/share/nginx/html` which is located on the default file system of the Linux.
###### solution
```sh
kubectl exec -it -n ckad-pod-design custom-nginx -- sh
# echo 'Welcome to CKAD mock exams!' > /usr/share/nginx/html/index.html
```
###### Q9
We have deployed several applications in the `ns-ckad17-svcn` namespace that are exposed inside the cluster via ClusterIP.  
Your task is to create a LoadBalancer type service that will serve traffic to the applications based on its labels. Create the resources as follows:  
- Service `lb1-ckad17-svcn` for serving traffic at `port 31890` to pods with labels `"exam=ckad, criteria=location"`.
- Service `lb2-ckad17-svcn` for serving traffic at `port 31891` to pods with labels `"exam=ckad, criteria=cpu-high"`.
###### solution
```yml
apiVersion: v1
kind: Service
metadata:
  name: lb1-ckad17-svcn
  namespace: ns-ckad17-svcn
spec:
  selector:
    exam: ckad
    criteria: location
  ports:
    - protocol: TCP
      port: 31890
      targetPort: <container-port> 
  type: LoadBalancer
---
apiVersion: v1
kind: Service
metadata:
  name: lb2-ckad17-svcn
  namespace: ns-ckad17-svcn
spec:
  selector:
    exam: ckad
    criteria: cpu-high
  ports:
    - protocol: TCP
      port: 31891
      targetPort: <container-port>
  type: LoadBalancer
```
###### Q10
###### solution
###### Q11
###### solution
###### Q12
###### solution
###### Q13
###### solution
###### Q14
###### solution
###### Q15
###### solution
###### Q16
###### solution
###### Q17
###### solution
###### Q18
###### solution
###### Q19
###### solution
###### Q20
###### solution