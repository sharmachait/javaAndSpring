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
      command: ['sh', '-c', 'while true; do date; sleep &TIME_FRED; done']
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