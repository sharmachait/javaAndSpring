###### Q4
Create a `persistent volume` called `cloudstack-pv` with the below properties:  
**-** Its capacity should be `128Mi`.  
**-** The volume type should be `hostpath` and path should be `/opt/cloudstack-pv`.  
Next, create a `persistent volume claim` called `cloudstack-pvc` as per below properties:  
**-** Request `50Mi` of storage from `cloudstack-pv` PV.
###### Solution
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cloudstack-pv
spec:
  storageClassName: manual
  capacity:
    storage: 128Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/opt/cloudstack-pv"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloudstack-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```
###### Q5
In the `ckad-job` namespace, schedule a job called `learning-every-hour` that prints this message in the shell every hour at 0 minutes: `I will pass CKAD certification`.  
In case the container in pod failed for any reason, it should be restarted automatically.  
Use `alpine` image for the cronjob!
###### Solution
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  namespace: ckad-job
  name: learning-every-hour
spec:
  schedule: "0 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: learning-every-hour
            image: alpine
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - echo I will pass CKAD certification
          restartPolicy: OnFailure
```
###### Q7
One co-worker deployed an nginx helm chart on the `cluster3` server called `lvm-crystal-apd`. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.  
After updating the helm chart, upgrade the helm chart version to `18.1.15` and increase the replica count to `2`.  
**NOTE: -** We have to perform this task on the `cluster3-controlplane` node.  
You can SSH into the `cluster3` using `ssh cluster3-controlplane` command.
###### Solution
```sh
ssh cluster3-controlplane
helm ls -A
helm repo ls
helm repo update lvm-crystal-apd -n crystal-apd-ns
helm pull lvm-crystal-apd/nginx --untar

vi /lvm-crystal-apd/Chart.yml # update the version 18.1.15
vi /lvm-crystal-apd/values.yml # update the replica count 2

helm upgrade lvm-crystal-apd ./nginx -n crystal-apd-ns
```
###### Q8
One application, `webpage-server-01`, is deployed on the Kubernetes cluster by the Helm tool. Now, the team wants to deploy a new version of the application by replacing the existing one. A new version of the helm chart is given in the `/root/new-version` directory on the `student-node`. Validate the chart before installing it on the Kubernetes cluster.   
Use the `helm` command to validate and install the chart. After successfully installing the newer version, uninstall the older version.
###### Solution
```sh
helm list -A
helm lint ./new-version
helm install webpage-server-02 ./new-version
helm uninstall webpage-server-01
```
###### Q9
In the `ckad-multi-containers` namespace, create a pod named `healthy-server`, which consists of 2 containers. One main container and one init-container both are running `busybox:1.28` image. Init container should print this message `Initialize application environment!` and then sleep for `10` seconds. Main container should print this message `The app is running!` and then sleep for `3600` seconds.
###### solution
do one of the following two
passing command to sh with -c 
```yml
  - command:
    - sh
    - -c
    - echo Initialize application environment! && sleep 10
```
or

```yml
  - command:
    - sh
    - -c
    - echo 'Initialize application environment!' && sleep 10
```
dont do 
```yml
  - command:
    - sh
    - -c
    - echo "Initialize application environment!" && sleep 10
```
###### Q10
The deployment called `foundary-apd` inside the `blue-apd` namespace on `cluster3` has undergone several, routine, rolling updates and rollbacks. 
Inspect the `revision 3` of this deployment and store the image name that was used in this revision in the `/root/records/foundry-revision-book.txt` file on the `student-node`.  
###### solution
```sh
kubectl rollout history -n blue-apd deploy foundary-apd --revision=3
```
###### Q11
One co-worker deployed an nginx helm chart on the `cluster1` server called `bitnami`. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.  
After updating the helm chart, upgrade the helm chart version to `18.3.0` and increase the replica count to `2`.  
**NOTE: -** We have to perform this task on the `cluster1-controlplane` node.  
You can SSH into the `cluster1` using `ssh cluster1-controlplane` command.
###### solution
```sh
helm repo update bitnami
helm pull bitnami/nginx --untar
cd nginx
vi Chart.yaml
vi values.yaml [/replicaCount]
helm uninstall bitnami -n ckad10-finance-ns
helm install bitnami -n ckad10-finance-ns ./nginx
```
###### Q12
We have deployed some pods in the namespaces `ckad-alpha` and `ckad-beta`.  
You need to create a NetworkPolicy named `ns-netpol-ckad` that will restrict all Pods in Namespace `ckad-alpha` to only have outgoing traffic to Pods in Namespace `ckad-beta` . Ingress traffic should not be affected.  
However, the NetworkPolicy you create should allow egress traffic on port 53 TCP and UDP form any where. (or not and)
###### solution
```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ns-netpol-ckad
  namespace: ckad-alpha
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ckad-beta
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```
and not (as this will logically and both)
```yml
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ckad-beta
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```
###### Q14