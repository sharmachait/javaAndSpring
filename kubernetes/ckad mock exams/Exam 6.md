###### Q2
In the `ckad-job` namespace, create a cron job called `my-alarm` that prints current datetime. It must be scheduled to run every Sunday at midnight.  
In case the container in pod failed for any reason, it should be restarted automatically.  
Use `busybox:1.28` image to create job.
###### Solution
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "0 0 * * 0"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date
          restartPolicy: OnFailure

```
###### Q3
Create a Persistent Volume called `log-volume`. It should make use of a `storage class` name `manual`. It should use `RWX` as the access mode and have a size of `1Gi`. The volume should use the hostPath `/opt/volume/nginx`  

Next, create a PVC called `log-claim` requesting a minimum of `200Mi` of storage. This PVC should bind to `log-volume`.  

Mount this in a pod called `logger` at the location `/var/www/nginx`. This pod should use the image `nginx:alpine`.
###### Solution
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/opt/volume/nginx"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: logger
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: log-claim
  containers:
    - name: task-pv-container
      image: nginx:alpine
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/var/www/nginx"
          name: task-pv-storage
```

###### Q4
We have deployed a new pod called `secure-pod` and a service called `secure-service`. Incoming or Outgoing connections to this pod are not working.  
Troubleshoot why this is happening.  
Make sure that incoming connection from the pod `webapp-color` are successful.
Important: Don't delete any current objects deployed.
###### Solution
```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```

to ping one pod from another
exec into the pod with
>kubectl exec -it webapp-pod -- sh
>nc -v -z -w 2 name-of-service 80

where 2 is wait seconds, and 80 is port number
###### Q5
Create a pod called `time-check` in the `dvl1987` namespace. This pod should run a container called `time-check` that uses the `busybox` image.
1. Create a config map called `time-config` with the data `TIME_FREQ=10` in the same namespace.
2. The `time-check` container should run the command: `while true; do date; sleep $TIME_FREQ;done` and write the result to the location `/opt/time/time-check.log`.
3. The path `/opt/time` on the pod should mount a volume that lasts the lifetime of this pod.
###### Solution
> kubectl create configmap time-config --from-literal=TIME_FREQ=10

create in the namespace

```yml
apiVersion: v1
kind: Pod
metadata:
  namespace: dvl1987
  name: time-check
spec:
  containers:
    - name: time-check
      image: busybox
      command: ['sh', '-c', 'while true; do date; sleep &TIME_FRED; done > /opt/time/time-check.log']
      envFrom:
      - configMapRef:
          name: time-config
      volumeMounts:
        - name: cache-volume
          mountPath: /opt/time
  volumes:  
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```
###### Q6
Create a new deployment called `nginx-deploy`, with one single container called `nginx`, image `nginx:1.16` and `4` replicas.  
The deployment should use `RollingUpdate` strategy with `maxSurge=1`, and `maxUnavailable=2`.  
  
Next upgrade the deployment to version `1.17`.  
  
Finally, once all pods are updated, undo the update and go back to the previous version.
###### solution
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
      maxSurge: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
```
>k edit deploy nginx-deploy
>kubectl rollout history deployment/nginx-deploy
>kubectl rollout undo deployment/nginx-deploy --to-revision=1
###### Q7
Create a `redis` deployment with the following parameters:  
Name of the deployment should be `redis` using the `redis:alpine` image. It should have exactly `1` replica.  
The container should request for `.2` CPU. It should use the label `app=redis`.  
It should mount exactly 2 volumes.  
a. An Empty directory volume called `data` at path `/redis-master-data`.  
b. A configmap volume called `redis-config` at path `/redis-master`.  
c. The container should expose the port `6379`.  
The configmap has already been created.
###### solution
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      volumes:
      - name: config-vol
        configMap:
          name: redis-config
      - name: data
        emptyDir:
          sizeLimit: 500Mi
      containers:
      - image: redis:alpine
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: config-vol
        name: redis
        resources:
          requests:
            cpu: "0.2"
        ports:
        - containerPort: 6379
```
###### Q8
Create a cronjob called `dice` that runs every `one minute`. Use the Pod template located at `/root/throw-a-dice`. The image `throw-dice` randomly returns a value between 1 and 6. The result of 6 is considered `success` and all others are `failure`.  
The job should be `non-parallel` and complete the task `once`. Use a `backoffLimit` of `25`.  
If the task is not completed within `20 seconds` the job should fail and pods should be terminated.
You don't have to wait for the job completion. As long as the cronjob has been created as per the requirements.
###### solution
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dice
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 1
      backoffLimit: 25 # This is so the job does not quit before it succeeds.
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          - name: dice
            image: kodekloud/throw-dice
          restartPolicy: Never
```
###### Q9
Create a pod called `my-busybox` in the `dev2406` namespace using the `busybox` image. The container should be called `secret` and should sleep for `3600` seconds.  
The container should mount a `read-only` secret volume called `secret-volume` at the path `/etc/secret-volume`. The secret being mounted has already been created for you and is called `dotfile-secret`.  
Make sure that the pod is scheduled on `controlplane` and no other node in the cluster.
###### solution
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-busybox
  name: my-busybox
  namespace: dev2406
spec:
  nodeSelector:
    kubernetes.io/hostname: controlplane
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  containers:
  - image: busybox
    name: secret
    command: ["sleep"]
    args: ["3600"]
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```
###### Q10
Create a single ingress resource called `ingress-vh-routing`. The resource should route HTTP traffic to multiple hostnames as specified below:
1. The service `video-service` should be accessible on `http://watch.ecom-store.com:30093/video`
2. The service `apparels-service` should be accessible on `http://apparels.ecom-store.com:30093/wear`
To ensure that the path is correctly rewritten for the backend service, add the following `annotation` to the resource:
```
nginx.ingress.kubernetes.io/rewrite-target: /
```
Here `30093` is the port used by the `Ingress Controller`
###### solution
```yml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: ingress-vh-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: watch.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/video"
        backend:
          service:
            name: video-service
            port:
              number: 8080
  - host: apparels.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/wear"
        backend:
          service:
            name: apparels-service
            port:
              number: 8080
```
###### Q11
A replicaset `rs-d33393` is created. However the pods are not coming up. Identify and fix the issue.
Once fixed, ensure the ReplicaSet has 4 `Ready` replicas.
###### Solution
The image used for the replicaset should be `busybox` instead of `busyboxXXXXXXX`. Use `kubectl edit rs rs-d33393` to fix the image. Then delete all PODs to provision new ones with the new image.
###### Q12
Please use the namespace `nginx-depl-svcn` for the following scenario.  
Create a deployment with name `nginx-ckad10-svcn` using `nginx` image with `2` replicas. Also expose the deployment via ClusterIP service .i.e. `nginx-ckad10-service-svcn` on port 80. Use the label `app=nginx-ckad` for both resources.  
Now, create a NetworkPolicy .i.e. `netpol-ckad-allow-svcn` so that only pods with label `criteria: allow` can access the deployment and apply it.
###### Solution
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-ckad
  name: nginx-ckad10-svcn
  namespace: nginx-depl-svcn
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
      - image: nginx
        name: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-ckad
  name: nginx-ckad10-service-svcn
  namespace: nginx-depl-svcn
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-ckad
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-ckad-allow-svcn
  namespace: nginx-depl-svcn
spec:
  podSelector:
    matchLabels:
      app: nginx
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

###### Q13
Create a service account named `ckad23-sa-aecs` in the namespace `ckad23-nssa-aecs`.  
Grant the service account `get` and `list` permissions to access **all resources** within the namespace using a Role named `wide-access-aecs`.  
Also bind the **Role** to the service account using a **RoleBinding** named `wide-access-rb-aecs`, restricting the access to the **ckad23-nssa-aecs** namespace only.
###### Solution
imperative way

```sh
kubectl config use-context cluster3
kubectl create ns ckad23-nssa-aecs
kubectl create serviceaccount ckad23-sa-aecs -n ckad23-nssa-aecs
kubectl create role wide-access-aecs --namespace=ckad23-nssa-aecs --verb=get,list --resource=* 
kubectl create rolebinding wide-access-rb-aecs \
   --role=wide-access-aecs \
   --serviceaccount=ckad23-nssa-aecs:ckad23-sa-aecs \
   --namespace=ckad23-nssa-aecs
```

```yml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ckad23-sa-aecs
  namespace: ckad23-nssa-aecs
---
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: wide-access-aecs
  namespace: ckad23-nssa-aecs
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list"]
---
# RoleBinding 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: wide-access-rb-aecs
  namespace: ckad23-nssa-aecs
subjects:
- kind: ServiceAccount
  name: ckad23-sa-aecs
  namespace: ckad23-nssa-aecs
roleRef:
  kind: Role
  name: wide-access-aecs
  apiGroup: rbac.authorization.k8s.io
```
###### Q18