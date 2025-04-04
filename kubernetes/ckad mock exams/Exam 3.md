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
- Now we know which pods use the labels, we can create the LoadBalancer type service using the imperative command.
```sh
kubectl -n ns-ckad17-svcn expose pod geo-location-app --type=LoadBalancer --name=lb1-ckad17-svcn
```
Similarly, create the another service
```sh
kubectl -n ns-ckad17-svcn expose pod cpu-load-app --type=LoadBalancer --name=lb2-ckad17-svcn
```
- Once the services are created, you can edit the services to use the correct nodePorts as per the question using `kubectl -n ns-ckad17-svcn edit svc lb2-ckad17-svcn`
###### Q10
We have created a Network Policy `netpol-ckad13-svcn` that allows traffic only to specific pods and it allows traffic only from pods with specific labels.
Your task is to edit the policy so that it allows traffic from pods with labels `access = allowed`.
###### solution
edit and add podselector
###### Q11
We have created a deployment of an application named `nginx-app-ckad` in the `default` namespace.
Configure a service named `nginx-svcn` for the application, which exposes the pods on multiple ports with different protocols.
- Expose port 80 using TCP with the name `http`
- Expose port 443 using TCP with the name `https`
###### solution
```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svcn
spec:
  selector:
    app: nginx-app-ckad
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
```
###### Q12
We have a Kubernetes namespace called `ckad12-ctm-sa-aecs`, which contains a service account and a pod. Your task is to modify the pod so that it uses the service account defined in the same namespace.  
Additionally, you need to ensure that the pod has access to the **API credentials** associated with the service account by **enabling** the automounting feature for the credentials.
###### solution
edit spec.serviceAccountName and spec.automountServiceAccountToken
###### Q13
Create a `role` named `pod-creater` in the `ckad20-auth-aecs` namespace, and grant only the **list**, **create** and **get** permissions on `pods` resources.  
Create a `role binding` named `mock-user-binding` in the same namespace, and assign the **pod-creater** role to a user named `mock-user`.
###### solution
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: ckad20-auth-aecs
  name: pod-creater
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "create", "list"]
```

```sh
kubectl create rolebinding mock-user-binding --clusterrole=pod-creater --user=mock-user --namespace=ckad20-auth-aecs
```
###### Q14
In the `ckad14-sa-projected` namespace, configure the `ckad14-api-pod` Pod to include a **projected volume** named `vault-token`.
Mount the service account token to the container at `/var/run/secrets/tokens`, with an expiration time of `7000` seconds.  
Additionally, set the intended audience for the token to `vault` and path to `vault-token`.
###### solution
A `projected` volume maps several existing volume sources into the same directory.
```yml
apiVersion: v1
kind: Pod
metadata:
  namespace: ckad14-sa-projected
  name: ckad14-api-pod
spec:
  containers:
  - name: container-test
    image: busybox:1.28
    command: ["sleep", "3600"]
    volumeMounts:
    - name: vault-token
      mountPath: "/var/run/secrets/tokens"
      readOnly: true
  serviceAccountName: default
  volumes:
  - name: vault-token
    projected:
      sources:
      - serviceAccountToken:
          audience: vault
          expirationSeconds: 7000
          path: vault-token
```
###### Q15
Create a custom resource `my-anime` of kind `Anime` with the below specifications:  
Name of Anime: `Naruto`  
Episode Count: `220`  
`TIP`: You may find the respective CRD with **anime** substring in it.
###### solution
```yml
apiVersion: "animes.k8s.io/v1alpha1"
kind: Anime
metadata:
  name: my-anime
spec:
  animeName: "Naruto"
  episodeCount: 220
```
###### Q16
Create a service account `dashboard-sa` in the namespace `levantest`.
###### solution
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: levantest
```
###### Q18
View the metrics (CPU and Memory) of the node `cluster2-node01` and copy the output to the `/root/node-metrics` file in `clustername,CPU and memory`.
###### solution
```sh
kubectl top nodes > /root/node-metrics
```
###### Q20
Store the **pod names** and their **ip addresses** from the `spectra-1267` ns at `/root/pod_ips_cka05_svcn` where the output is sorted by their IP's
Please ensure the format as shown below:  
```
POD_NAME        IP_ADDR
pod-1           ip-1
pod-3           ip-2
pod-2           ip-3
...
```
###### solution
```sh
student-node ~ ➜  kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP

POD_NAME   IP_ADDR
pod-12     10.42.0.18
pod-23     10.42.0.19
pod-34     10.42.0.20
pod-21     10.42.0.21
...

# store the output to /root/pod_ips
student-node ~ ➜  kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP > /root/pod_ips_cka05_svcn
```