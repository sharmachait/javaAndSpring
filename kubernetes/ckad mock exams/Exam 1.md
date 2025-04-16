###### Q1. 
In the `ckad-multi-containers` namespace, create a pod named `tres-containers-pod`, which has 3 containers matching the below requirements:  
- The first container named `primero` runs `busybox:1.28` image and has `ORDER=FIRST` environment variable.    
- The second container named `segundo` runs `nginx:1.17` image and is exposed at port `8080`.  
- The last container named `tercero` runs `busybox:1.31.1` image and has `ORDER=THIRD` environment variable.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: tres-containers-pod
  name: tres-containers-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - image: busybox:1.28
    name: primero
    command: ["sleep", "3600"]
    env:
    - name: ORDER
      value: FIRST
  - image: nginx:1.17
    name: segundo
    ports:
    - containerPort: 8080
  - image: busybox:1.31.1
    name: tercero
    command: ["sleep", "3600"]
    env:
    - name: ORDER
      value: THIRD
```
###### Q2
Create a storage class with the name `banana-sc-ckad08-str` as per the properties given below:  
**-** Provisioner should be `kubernetes.io/no-provisioner`,  
**-** Volume binding mode should be `WaitForFirstConsumer`.  
**-** Volume expansion should be enabled.
###### solution
```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: banana-sc-ckad08-str
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```
###### Q3
In the `ckad-job` namespace, create a cronjob named `simple-node-job` to run every 30 minutes to list all the running processes inside a container that used `node` image (the command needs to be run in a shell).
In Unix-based operating systems, `ps -eaf` can be use to list all the running processes.
###### solution
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-node-job
  namespace: ckad-job
spec:
  schedule: "*/30 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: simple-node-job
            image: node
            imagePullPolicy: IfNotPresent
            command: ["/bin/sh", "-c", "ps -eaf"]
          restartPolicy: OnFailure
```
###### Q4
In the `ckad-pod-design` namespace, create a pod called `ckad-ubuntu-qwfefefwe` that runs a `ubuntu` image.  
The pod's container should be named `ubuntu-server`; the container will sleep for `3600` seconds.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ckad-ubuntu-qwfefefwe
  name: ckad-ubuntu-qwfefefwe
spec:
  containers:
  - image: ubuntu
    name: ubuntu-server
    command: ["sleep", "3600"]
```
###### Q5
In this task, we have to create two identical environments that are running different versions of the application. The team decided to use the Blue/green deployment method to deploy a total of 10 application pods which can mitigate common risks such as downtime and rollback capability.  
Also, we have to route traffic in such a way that `30%` of the traffic is sent to the `green-apd` environment and the rest is sent to the `blue-apd` environment. All the development processes will happen on `cluster 3` because it has enough resources for scalability and utility consumption.  
Specification details for creating a `blue-apd` deployment are listed below: -  
1. The name of the deployment is `blue-apd`.  
2. Use the label `type-one: blue`.  
3. Use the image `kodekloud/webapp-color:v1`.  
4. Add labels to the pod `type-one: blue` and `version: v1`.  
Specification details for creating a `green-apd` deployment are listed below: -  
1. The name of the deployment is `green-apd`.  
2. Use the label `type-two: green`.  
3. Use the image `kodekloud/webapp-color:v2`.  
4. Add labels to the pod `type-two: green` and `version: v1`.  
We have to create a service called `route-apd-svc` for these deployments. Details are here: -  
1. The name of the service is `route-apd-svc`.  
2. Use the correct service type to access the application from outside the cluster and application should listen on port `8080`.  
3. Use the selector label `version: v1`.  
**NOTE**: - We do not need to increase replicas for the deployments, and all the resources should be created in the `default` namespace.
###### solution
```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-one: blue
  name: blue-apd
spec:
  replicas: 7
  selector:
    matchLabels:
      type-one: blue
      version: v1
  template:
    metadata:
      labels:
        version: v1
        type-one: blue
    spec:
      containers:
        - image: kodekloud/webapp-color:v1
          name: blue-apd
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-two: green
  name: green-apd
spec:
  replicas: 3
  selector:
    matchLabels:
      type-two: green
      version: v1
  template:
    metadata:
      labels:
        type-two: green
        version: v1
    spec:
      containers:
        - image: kodekloud/webapp-color:v2
          name: green-apd
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: route-apd-svc
  name: route-apd-svc
spec:
  type: NodePort
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    version: v1
```
###### Q6
On the `student-node`, a Helm chart repository is given under the `/opt/` path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: -
1. The release name should be `webapp-color-apd`.
2. All the resources should be deployed on the `frontend-apd` namespace.
3. The service type should be `node port`.
4. Scale the deployment to `3`.
5. Application version should be `1.20.0`.
`NOTE`: - Remember to make necessary changes in the `values.yaml` and `Chart.yaml` files according to the specifications, and, to fix the issues, inspect the template files.
You can start new terminal to open `student-node` command.

Chart.yml
```yml
apiVersion: v2
name: webapp-color-apd
description: A Helm chart for Webapp Color Application
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "v1"
```

values.yml
```yml
replicaCount: 0

image:
  repository: kodekloud/webapp-color
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "v1"

imagePullSecrets: []

name: webapp-color-apd

serviceAccount:
  # Specifies whether a service account should be created
  create: false
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: webapp-sa-apd
  labels:
    app: webapp-color-apd

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000
service:
  name: webapp-color-svc
  type: ClusterIP
  port: 8080


environment: development


configMap:
  name: webapp-configmap-apd

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi
nodeSelector: {}

tolerations: []

affinity: {}
```

templates
```yml
apiVersion: v1
metadata:
  name: {{ .Values.configMap.name }}
kind: ConfigMap
data:
  APP_COLOR: darkblue
---
apiVersion: v1
kind: Deployment
metadata:
  labels:
    app: webapp-color-apd
  name: {{ .Values.name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: webapp-color-apd
  template:
    metadata:
      labels:
        app: webapp-color-apd
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: webapp-color-apd
        envFrom:
         - configMapRef:
                name: {{ .Values.configMap.name }}
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ .Values.serviceAccount.name }}
  labels:
    app: webapp-color-apd
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service.Name }}
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: webapp-color-apd
  type: {{ .Values.service.type }}
```

###### solution
> kubectl create ns frontend-apd

a.) Update the value of the `appVersion` to `1.20.0` in the `Chart.yaml` file.  
  
b.) Update the value of the `replicaCount` to `3` in the `values.yaml` file.  
  
c.) Update the value of the `type` to `NodePort` in the `values.yaml` file.
>helm lint ./webapp-color-apd/

But in our case, there are some issues with the given templates.  
1. Deployment apiVersion needs to be correctly written. It should be `apiVersion: apps/v1`.  
2. In the service YAML, there is a typo in the template variable `{{ .Values.service.name }}` because of that, it's not able to reference the value of the name field defined in the `values.yaml` file for the Kubernetes service that is being created or updated.
> helm upgrade --install webapp-color-apd ./webapp-color-apd -n frontend-apd --create-namespace

###### Q7 
We have deployed a pod `pod22-ckad-svcn` in the default namespace. Create a service `svc22-ckad-svcn` that will expose the pod at port `6335`.
###### solution
> kubectl expose pod pod22-ckad-svcn --name=svc22-ckad-svcn --port=6335
###### Q8
create a network policy named `deny-all-svcn` that denies all incoming and outgoing traffic to `ckad12-svcn` namespace.
Note: The namespace `ckad12-svcn` doesn't exist. Create the namespace before creating the Policy.
###### solution
> kubectl create networkpolicy default-deny-ingress --namespace=ckad12-svcn --ptype=ingress --dry-run=client -o yaml

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-svcn
  namespace: ckad12-svcn
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
###### Q9
Deploy a pod with name `webapp-svcn` using the `kodekloud/webapp-color` image with the label `tier=msg`.  
Now, Create a service `webapp-service-svcn` to expose the pod `webapp-svcn` application within the cluster on port `6379`.
###### solution
> kubectl run webapp-svcn --image=kodekloud/webapp-color -l tier=msg
>kubectl expose pod webapp-svcn --port=6379 --name webapp-service-svcn
  
###### Q10
Create a Service called `ckad12-service` that routes traffic to an external IP address.  
Please note that service should listen on `port 53` and be of type `ExternalName`. Use the external IP address 8.8.8.8
###### solution
```yml
apiVersion: v1
kind: Service
metadata:
  name: ckad12-service
spec:
  type: ExternalName
  externalName: 8.8.8.8
  ports:
    - name: http
      port: 53
      targetPort: 53
```
###### Q11
Create a pod named `ckad17-qos-aecs-3` in namespace `ckad17-nqoss-aecs` with image `nginx` and container name `ckad17-qos-ctr-3-aecs`.
Define other fields such that the Pod is configured to use the Quality of Service (QoS) class of `Burstable`.
Also retrieve the `name` and `QoS` class of each Pod in the namespace `ckad17-nqoss-aecs` in the below format and save the output to a file named `qos_status_aecs` in the `/root` directory.
###### solution
A Pod is given a QoS class of `Burstable` if:
- At least one Container in the Pod has a memory or CPU request or limit
```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ckad17-qos-aecs-3
  name: ckad17-qos-aecs-3
  namespace: ckad17-nqoss-aecs
spec:
  containers:
  - image: nginx
    name: ckad17-qos-ctr-3-aecs
    resources:
      requests:
        memory: "100Mi"
```
to get the QoS 
>kubectl describe pod ckad17-qos-aecs-1
###### Q12
Create a custom resource `my-anime` of kind `Anime` with the below specifications:  
  
Name of Anime: `Death Note`  
Episode Count: `37`
###### solution
> kubectl get crd animes.animes.k8s.io -o yaml

api version for CR is the group of the CRD + the version of the CRD
```yml
apiVersion: animes.k8s.io/v1alpha1
kind: Anime
metadata: 
  name: my-anime
spec:
  animeName: "Death Note"
  episodeCount: 37
```
###### Q13
Create a ConfigMap named `ckad04-config-multi-env-files-aecs` in the `default` namespace from the environment(env) files provided at `/root/ckad04-multi-cm` directory.
###### solution
> kubectl create configmap ckad04-config-multi-env-files-aecs --from-env-file=/root/ckad04-multi-cm/file1.properties --from-env-file=/root/ckad04-multi-cm/file2.properties
###### Q14
1. Create a new secret named `ckad01-db-scrt-aecs` with the data given below.      
    Secret Name: `ckad01-db-scrt-aecs`  
    Secret 1: `DB_Host=sql01`  
    Secret 2: `DB_User=root`  
    Secret 3: `DB_Password=password123`  
2. Configure `ckad01-mysql-server` to load environment variables from the newly created secret, where the **keys** from the secret should become the **environment variable name** in the Pod.
###### solution
> kubectl create secret generic ckad01-db-scrt-aecs --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```yml
env:
    - name: DB_Host
      valueFrom:
        secretKeyRef:
          name: ckad01-db-scrt-aecs
          key: DB_Host
    - name: DB_User
      valueFrom:
        secretKeyRef:
          name: ckad01-db-scrt-aecs
          key: DB_User
    - name: DB_Password
      valueFrom:
        secretKeyRef:
          name: ckad01-db-scrt-aecs
          key: DB_Password
```
###### Q15
Create a **ResourceQuota** called `ckad16-rqc` in the namespace `ckad16-rqc-ns` and enforce a limit of `one` **ResourceQuota** for the namespace.
###### solution
```yml
apiVersion: v1 
kind: ResourceQuota 
metadata: 
  name: ckad16-rqc 
  namespace: ckad16-rqc-ns 
spec: 
  hard: 
    resourcequotas: "1"
```

###### explanation
```yml
hard:resourcequotas: "1"
```

This specifically limits the number of ResourceQuota objects that can exist in the namespace to 1.

1. This creates an interesting self-limiting situation:
    - This ResourceQuota itself counts as one ResourceQuota
    - Since it limits ResourceQuotas to 1, no other ResourceQuota objects can be created in this namespace
    - If someone tries to create another ResourceQuota in this namespace, Kubernetes will reject it

This is a meta-control mechanism - it's using a ResourceQuota to limit ResourceQuotas themselves. This can be useful when you want to prevent users from creating additional resource quotas that might override or conflict with your intended resource management policies.
###### Q16
Using the pod template on **student-node** at `/root/ckad08-dotfile-aecs.yaml` , create a pod `ckad18-secret-pod` in the namespace `ckad18-secret` with the specifications as defined below:  
Define a volume section named `secret-volume` that is backed by a Kubernetes Secret named `ckad18-secret-aecs`.  
Mount the `secret-volume` volume to the container's `/etc/secret-volume` directory in `read-only` mode, so that the container can access the secrets stored in the `ckad18-secret-aecs` secret.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  name: ckad18-secret-pod
  namespace: ckad18-secret
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: ckad18-secret-aecs
  containers:
  - name: ckad08-top-scrt-ctr-aecs
    image: registry.k8s.io/busybox
    command:
    - ls
    - "-al"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```
###### Q17
Update the newly created pod `simple-webapp-aom` with a `readinessProbe` using the given specifications.
Configure an `HTTP` readiness probe to the existing pod simple-webapp with `path` value set to `/ready` and `port` number to access container is `8080`.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: simple-webapp
  name: simple-webapp-aom
  namespace: default
spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "80"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: simple-webapp
    ports:
    - containerPort: 8080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
```
###### Q18
Create a new pod with image `redis` and name `ckad-probe` and configure the pod with `livenessProbe` with `command` `ls` and set `initialDelaySeconds` to 5 .
###### solution
> kubectl run ckad-probe --image=redis  --dry-run=client -o yaml > ckad-probe.yaml
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: redis
  name: ckad-probe
spec:
  containers:
    - image: redis
      imagePullPolicy: IfNotPresent
      name: redis
      resources: {}
      livenessProbe:
        exec:
          command:
            - ls
        initialDelaySeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

###### Q20
export the built container image in OC-format and store it at -/human stork/macque 3.0 tar
###### solution
`docker save <image>:<tag> > ~/humane-stork/macaque-3.0.tar`