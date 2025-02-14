###### Q1
In the `ckad-multi-containers` namespaces, create a `ckad-sidecar-pod` pod that matches the following requirements.  
Pod has an `emptyDir` volume named `my-vol`.  
The first container named `main-container`, runs `nginx:1.16` image. This container mounts the `my-vol` volume at `/usr/share/nginx/html` path.  
The second container named `sidecar-container`, runs `busybox:1.28` image. This container mounts the `my-vol` volume at `/var/log` path.  
Every 5 seconds, this container should write the current date along with greeting message `Hi I am from Sidecar container` to `index.html` in the `my-vol` volume.
###### solution
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad-sidecar-pod
  namespace: ckad-multi-containers
spec:
  volumes:
  - name: my-vol
    emptyDir:
      sizeLimit: 500Mi
  containers:
  - name: main-container
    image: nginx:1.16
    volumeMounts:
    - name: my-vol
      mountPath: /usr/share/nginx/html
  - name: sidecar-container
    image: busybox:1.28
    volumeMounts:
    - name: my-vol
      mountPath: /var/log
    command: ["/bin/sh","-c","while true; do echo \"$(date) Hi I am from Sidecar container\" >> /var/log/index.html; sleep 5; done"]
```
###### Q2
In the `ckad-multi-containers` namespace, create pod named `dos-containers-pod` which has 2 containers matching the below requirements:
- The first container named `alpha` runs the `nginx:1.17` image and has the `ROLE=SERVER` ENV variable configured.
- The second container named `beta`, runs `busybox:1.28` image. This container will print message `Hello multi-containers` (command needs to be run in shell).
**NOTE:** all containers should be in a running state to pass the validation.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  name: dos-containers-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - image: nginx:1.17
    name: alpha
    env:
    - name: ROLE
      value: SERVER
  - image: busybox:1.28
    name: beta
    command: ["/bin/sh", "-c", "echo \"Hello multi-containers\"; sleep 3600;"]
```
###### Q3
In the `ckad-job` namespace, create a job called `alpine-job` that runs `top` command inside the container; use `alpine` image for this task.
###### solution
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: alpine-job
  namespace: ckad-job
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - image: alpine
        name: alpine-job
        command: ["/bin/sh", "-c", "top"]
```
###### Q4
In the `ckad-pod-design` namespace, start a `ckad-redis-wiipwlznjy` pod running the `redis` image; the container should be named `redis-custom-annotation`.  
Configure a custom `annotation` to that pod as below:  
```
KKE: https://engineer.kodekloud.com/
```
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    KKE: https://engineer.kodekloud.com/
  name: ckad-redis-wiipwlznjy
  namespace: ckad-pod-design
spec:
  containers:
  - image: redis
    name: redis-custom-annotation
```
###### Q5
Create a new deployment called `ocean-apd` in the default namespace using the image `kodekloud/webapp-color:v1`.  
Use the following specs for the deployment:    
**1.** Replica count should be `2`.  
**2.** Set the Max Unavailable to `45%` and Max Surge to `55%`.  
**3.** Create the deployment and ensure all the pods are ready.  
**4.** After successful deployment, upgrade the deployment image to `kodekloud/webapp-color:v2` and inspect the deployment rollout status.  
**5.** Check the rolling history of the deployment and on the `student-node`, save the `current` revision count number to the `/opt/ocean-revision-count.txt` file.  
**6.** Finally, perform a rollback and revert the deployment image to the older version.
###### solution
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ocean-apd
  name: ocean-apd
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ocean-apd
  template:
    metadata:
      labels:
        app: ocean-apd
    spec:
      containers:
      - image: kodekloud/webapp-color:v1
        name: webapp-color
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 45%
      maxSurge: 55%
```

```sh
kubectl edit deploy ocean-apd
kubectl rollout history deployment/ocean-apd
kubectl rollout undo deployment/ocean-apd --to-revision=1
```
###### Q6
In the initial phase, our team lead wants to deploy Kubernetes Dashboard using the popular Helm tool. Kubernetes Dashboard is a powerful and versatile web-based management interface for Kubernetes clusters. It allows users to manage and monitor cluster resources, deploy applications, troubleshoot issues, and more.  
This deployment will be facilitated through the `kubernetes` Helm chart on the `cluster1-controlplane` node to fulfill the requirements of our new client

The chart URL and other specifications are as follows: -  
**1.** The chart URL link - `https://kubernetes.github.io/dashboard/`  
**2.** The chart repository name should be `kubernetes-dashboard`.  
**3.** The release name should be `kubernetes-dashboard-server`.  
**4.** All the resources should be deployed on the `cd-tool-apd` namespace.  
**NOTE: -** You have to perform this task from the `student-node`.
###### solution
> helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
> helm repo ls
> helm search repo kubernetes-dashboard
> kubectl create ns cd-tool-apd
> helm upgrade --install kubernetes-dashboard-server kubernetes-dashboard/kubernetes-dashboard -n cd-tool-apd
###### Q7
Create a deployment called `space-20` using the `redis` image to the `aerospace` namespace.
###### solution
>kubectl create deploy space-20 --image=redis --namespace aerospace
###### Q8
Create a pod with name `pod21-ckad-svcn` using the `nginx:alpine` image in the default namespace and also expose the pod using service `pod21-ckad-svcn` on `port 80`. check again
###### solution
>kubectl run pod21-ckad-svcn --image=nginx:alpine --namespace default --port=80
>kubectl expose pod pod21-ckad-svcn --port=80 --name=pod21-ckad-svcn
###### Q9
We have already deployed an application that consists of frontend, backend, and database pods in the `app-ckad` namespace. **Inspect them.**  
Your task is to create:  
- A service `frontend-ckad-svcn` to expose the frontend pods outside the cluster on `port 31100`.
- A service `backend-ckad-svcn` to make backend pods to be accessible within the cluster.  
- A policy `database-ckad-netpol` to limit access to database pods only to backend pods.
###### solution
>kubectl create service nodeport frontend-ckad-svcn --tcp=80:80 --node-port=31100 # but remeber to change the labels
```yml
apiVersion: v1
kind: Service
metadata:
  name: frontend-ckad-svcn
  namespace: app-ckad
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 31100
  selector:
    app: frontend
---
apiVersion: v1
kind: Service
metadata:
  name: backend-ckad-svcn
  namespace: app-ckad
spec:
  selector:
    app: backend
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-ckad-netpol
  namespace: app-ckad
spec:
  podSelector:
    matchLabels:
      app: database
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
```
###### Q10
Create a deployment named `hr-web-app-ckad05-svcn` using the image `kodekloud/webapp-color` with `2` replicas.  
Expose the `hr-web-app-ckad05-svcn` with service named `hr-web-app-service-ckad05-svcn` on port `30082` on the nodes of the cluster.
###### solution
> kubectl create deploy hr-web-app-ckad05-svcn --image=kodekloud/webapp-color --replicas=2 --port=8080
```yml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: hr-web-app-ckad05-svcn
  name: hr-web-app-service-ckad05-svcn
spec:
  ports:
  - name: 80-80
    nodePort: 30082
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: hr-web-app-ckad05-svcn
  type: NodePort
```
###### Q11
  
Please use the namespace `nginx-deployment` for the following scenario.  
Create a deployment with name `nginx-ckad11` using `nginx` image with `2` replicas. Also expose the deployment via ClusterIP service .i.e. `nginx-ckad11-service` on port 80. Use the label `app=nginx-ckad` for both resources.  
Now, create a NetworkPolicy .i.e. `ckad-allow` so that only pods with label `criteria: allow` can access the deployment on port 80 and apply it.
###### solution

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ckad11
  namespace: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-ckad
  template:
    metadata:
      labels:
        app: nginx-ckad
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ckad11-service
  namespace: nginx-deployment
spec:
  selector:
    app: nginx-ckad
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ckad-allow
  namespace: nginx-deployment
spec:
  podSelector:
    matchLabels:
      app: nginx-ckad
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          criteria: allow
    ports:
    - protocol: TCP
      port: 80
```
###### Q12
Update pod `ckad06-cap-aecs` in the namespace `ckad05-securityctx-aecs` to run as root user and with the `SYS_TIME` and `NET_ADMIN` capabilities.  
Note: Make only the necessary changes. Do not modify the name of the pod.
###### solution
```yml
securityContext:
  runAsUser: 0
  capabilities:
	add:
	- SYS_TIME
	- NET_ADMIN
```
###### Q13
Define a Kubernetes custom resource definition (CRD) for a new resource **kind** called `Foo` (plural form - `foos`) in the `samplecontroller.k8s.io` group.  
This CRD should have a version of `v1alpha1` with a schema that includes two properties as given below:  
> `deploymentName` (a string type) and `replicas` (an integer type with minimum value of 1 and maximum value of 5).

It should also include a `status` **subresource** which enables retrieving and updating the status of **Foo** object, including the `availableReplicas` property, which is an `integer` type.  
The **Foo** resource should be `namespace` **scoped**.    
`Note`: We have provided a template `/root/foo-crd-aecs.yaml` for your ease. There are few issues with it so please make sure to incorporate the above requirements before deploying on cluster.
###### solution
```yml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: foos.samplecontroller.k8s.io
  annotations:
    "api-approved.kubernetes.io": "unapproved, experimental-only"
spec:
  group: samplecontroller.k8s.io
  scope: Namespaced
  names:
    kind: Foo
    plural: foos
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        # schema used for validation
        openAPIV3Schema:
          type: object
          properties:
            spec:
              # Spec for schema goes here !
              type: object
              properties:
                deploymentName:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 5
            status:
              type: object
              properties:
                availableReplicas:
                  type: integer
      # subresources for the custom resource
      subresources:
        # enables the status subresource
        status: {}
```
###### Q14
We created two **ConfigMaps** ( `ckad02-config1-aecs` and `ckad02-config2-aecs` ) and one **Pod** named `ckad02-test-pod-aecs` in the `ckad02-mult-cm-cfg-aecs` namespace.  
Create two environment variables for the above pod with below specifications:  
> 1. `GREETINGS` with the data from configmap `ckad02-config1-aecs`
> 2. `WHO` with the data from configmap `ckad02-config2-aecs`.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  name: ckad02-test-pod-aecs
  namespace: ckad02-mult-cm-cfg-aecs
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - while true; do env | egrep 'GREETINGS|WHO'; sleep 10; done
    image: registry.k8s.io/busybox
    imagePullPolicy: Always
    name: test-container
    env:
    - name: GREETINGS
      valueFrom:
        configMapKeyRef:
          name: ckad02-config1-aecs
          key: greetings.how
    - name: WHO
      valueFrom:
        configMapKeyRef:
          name: ckad02-config2-aecs
          key: man
```
###### Q15
Create a Kubernetes Pod named `ckad15-memory`, with a container named `ckad15-memory` running the `polinux/stress` image, and configure it to use the following specifications:  
- Command: `stress`
- Arguments: `["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]`
- Requested memory: `10Mi`
- Memory limit: `10Mi`
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ckad15-memory
  name: ckad15-memory
spec:
  containers:
  - image: polinux/stress
    name: ckad15-memory
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]
    resources:
      requests:
        memory: "10Mi"
      limits:
        memory: "10Mi"
```
###### Q16
Create a role `configmap-updater` in the `ckad17-auth1` namespace granting the `update` and `get` permissions on `configmaps` resources but restricted to only the `ckad17-config-map` instance of the resource.
###### solution
get the api version from 
> kubectl api-resources | grep configmap

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2023-03-22T09:01:04Z"
  name: configmap-updater
  namespace: ckad17-auth1
  resourceVersion: "2799"
  uid: c152750a-198e-438e-9993-64b3e872c3e0
rules:
- apiGroups:
  - ""
  resourceNames:
  - ckad17-config-map
  resources:
  - configmaps
  verbs:
  - update
  - get
```
###### Q17
Identify the kube api-resources that use `api_version=storage.k8s.io/v1` using kubectl command line interface and store them in `/root/api-version.txt` on student-node.
###### solution
```sh
kubectl api-resources | grep -i storage.k8s.io/v1  > /root/api-version.txt
```
###### Q18
A manifest file located at `root/ckad-aom.yaml` on **cluster3-controlplane**. Which can be used to create a multi-containers pod. There are issues with the manifest file, preventing resource creation. Identify the errors, fix them and create resource.  
You can access controlplane by `ssh cluster3-controlplane` if required.
###### solution
open yaml file and check in **spec -> nginx container** you can see error with `mountPath` --> **mountPath: "/var/log"** change it to **mountPath: `/var/log/nginx`** and apply changes.
###### Q19
Find the error and fix the issue.

```yml
apiVersion: k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: mongodbs.mongodb.com
spec:
  group: mongodb.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                name:
                  type: string
                replicas:
                  type: integer
                storageSize:
                  type: string
                version:
                  type: string
                password:
                  type: string
                  format: password
                sslEnabled:
                  type: boolean
                tlsEnabled:
                  type: boolean
                nodeSelector:
                  type: object
                  additionalProperties:
                    type: string
      subresources:
        status: {}
  scope: Namespaced
  names:
    plural: mongodbs
    singular: mongodb
    kind: Mongodb
    shortNames:
      - mdb
```
###### solution

Check for apiVersion

```sh
kubectl api-resources | grep -i crd
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1
```

Replace the correct apiVersion

```sh
apiVersion: apiextensions.k8s.io/v1
```