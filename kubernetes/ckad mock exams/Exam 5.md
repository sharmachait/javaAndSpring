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