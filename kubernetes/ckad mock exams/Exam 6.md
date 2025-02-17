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