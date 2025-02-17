###### Q5
A web application running on cluster2 called `robox-west-apd` on the `fusion-apd-x1df5` namespace. The Ops team has created a new service account with a set of permissions for this web application. Update the newly created SA for this deployment.  
Also, change the strategy type to `Recreate`, so it will delete all the pods immediately and update the newly created SA to all the pods.
###### sol
set `.spec.strategy.type==Recreate`
```sh
kubectl set serviceaccount deploy robox-west-apd galaxy-apd-xb12 -n fusion-apd-x1df5
```
###### Q6
On the `cluster1`, the team has installed multiple helm charts on a different namespace. By mistake, those deployed resources include one of the vulnerable images called `kodekloud/click-counter:latest`. Find out the release name and uninstall it.
###### sol
```sh
kubectl get ns
helm -n security-alpha-01 list
helm -n security-alpha-01 uninstall security-alpha-apd
```
###### Q10
We have deployed an application in the `green-space` namespace. we also deployed the ingress controller and the ingress resource.  
However, currently, the ingress controller is not working as expected. Inspect the ingress definitions and troubleshoot the issue so that the services are accessible as per the ingress resource definition.
Also, update the path for the `app-wear-service` to `/app-wear` and `app-video-service` to `/app-video`
Note: You are allowed to edit or delete the resources related to ingress but do not change the pods.
###### solution
- Check the status of the ingress, pods, and application related services.

```
cluster3-controlplane ~ ➜  k get pods -n ingress-nginx 
NAME                                        READY   STATUS      RESTARTS      AGE
ingress-nginx-admission-create-l6fgw        0/1     Completed   0             11m
ingress-nginx-admission-patch-sfgc4         0/1     Completed   0             11m
ingress-nginx-controller-5f8964959d-278rc   0/1     Error       2 (26s ago)   29s
```

You would see an Error or CrashLoopBackOff in the ingress-nginx-controller. Inspect the logs of the controller pod.

```
cluster3-controlplane ~ ✖ k logs -n ingress-nginx ingress-nginx-controller-5f8964959d-278rc 
-------------------------------------------------------------------------------
--------
F0316 08:03:28.111614      57 main.go:83] No service with name default-backend-service found in namespace default:

-------
```

You see an error msg saying "No service with name default-backend-service found in namespace default".

- We don't have the service with that name in the default namespace, so we need to edit the ingress controller deployment to use the service that we have .i.e. `default-backend-service` in the `green-space` namespace.
  
- To create the controller deployment with correct backend service, first save the deployment in a file, delete the controller deployment, edit the file and create the deployment.

- Save the deployment in file

```
k get -n ingress-nginx deployments.apps ingress-nginx-controller -o yaml >> ing-control.yaml
```

- Delete the deployment.  
    `k delete -n ingress-nginx deploy ingress-nginx-controller`
- Edit the file to match the correct service.

```
     spec:
        containers:
        - args:
          - /nginx-ingress-controller
          - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
          - --election-id=ingress-controller-leader
          - --watch-ingress-without-class=true
          - --default-backend-service=green-space/default-backend-service   #Changed to correct namespace
          - --controller-class=k8s.io/ingress-nginx
          - --ingress-class=nginx
          - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
          - --validating-webhook=:8443
          - --validating-webhook-certificate=/usr/local/certificates/cert
          - --validating-webhook-key=/usr/local/certificates/key
```

- Apply the manifest, it should be up and running.
###### Q11
For this scenario, we have already deployed an application in the `global-space`. Inspect them and create an ingress resource with name `ingress-resource-xnz` to make the application available at `/eat` on the Ingress service. Use ingress class of `nginx`.  
Also, make use of following annotation fields: -
```
nginx.ingress.kubernetes.io/rewrite-target: /
nginx.ingress.kubernetes.io/ssl-redirect: "false"
```
`Ingress` resource comes under the `namespace` scoped, so don't forget to create the ingress in the `global-space` namespace.  
Make sure the paths select the correct backend services.
###### Solution
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-resource-xnz
  namespace: global-space
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - backend:
          service:
            name: food-service
            port:
              number: 8080
        path: /eat
        pathType: Prefix
```
###### Q12
Create an nginx pod called `nginx-resolver-ckad03-svcn` using image `nginx`, and expose it internally at port `80` with a service called `nginx-resolver-service-ckad03-svcn`.
###### Solution
```sh
kubectl run nginx-resolver-ckad03-svcn --image=nginx --port=80
kubectl expose pod nginx-resolver-ckad03-svcn --port=80 --name=nginx-resolver-service-ckad03-svcn
```
###### Q13
Create a Deployment named `ckad13-deployment` with "two replicas" of `nginx` image and expose it using a service named `ckad13-service`.  
Please be noted that service needs to be accessed from both inside and outside the cluster (use port `31080`).
###### Solution
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ckad13-deployment
spec:
  replicas: 2
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
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: ckad13-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 31080
```
###### Q14
We have a sample CRD at `/root/ckad10-crd-aecs.yaml` which should have the following validations:  
- `destinationName`, `country`, and `city` must be **string** types.
- `pricePerNight` must be an **integer** between `50` and `5000`.
- `durationInDays` must be an **integer** between `1` and `30`
Update the file incorporating the above validations in a `namespaced` scope.  
`Note`: Remember to create the CRD after the required changes.
###### Solution
```yml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: holidaydestinations.destinations.k8s.io
  annotations:
    "api-approved.kubernetes.io": "unapproved, experimental-only"
  labels:
    app: holiday
spec:
  group: destinations.k8s.io
  names:
    kind: HolidayDestination
    singular: holidaydestination
    plural: holidaydestinations
    shortNames:
      - hd
  scope: Namespaced
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                destinationName:
                  type: string
                country:
                  type: string
                city:
                  type: string
                pricePerNight:
                  type: integer
                  minimum: 50
                  maximum: 5000
                durationInDays:
                  type: integer
                  minimum: 1
                  maximum: 30
            status:
              type: object
              properties:
                availableRooms:
                  type: integer
                  minimum: 0
                  maximum: 1000
      # subresources for the custom resource
      subresources:
        # enables the status subresource
        status: {}
```
###### Q15
Create a `ClusterRole` named `healthz-access` that allows `GET` and `POST` **requests** to the **non-resource** endpoint `/healthz` and all `subpaths`.

Bound this **ClusterRole** to a user `healthz-user` using a **ClusterRoleBinding** named `healthz-access-binding`.
###### Solution
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: healthz-access
rules:
rules:
- nonResourceURLs: ["/healthz", "/healthz/*"] # '*' in a nonResourceURL is a suffix glob match
  verbs: ["get", "post"]
```
```sh
kubectl create clusterrole healthz-access --non-resource-url=/healthz,/healthz/* --verb=get,post
kubectl create clusterrolebinding healthz-access-binding --clusterrole=healthz-access --user=healthz-user
```
###### Q16
Create a ConfigMap named `ckad03-config-aecs` in the `default` namespace with below specifications:  
###### Solution
```sh
kubectl create configmap ckad03-config-aecs --from-literal=Exam=utlimate-mock-ckad --from-literal=Provider=kodekloud
```
###### Q17
Define a Kubernetes custom resource definition (CRD) for a new resource **kind** called `Foo` (plural form - `foos`) in the `samplecontroller.k8s.io` group.  
This CRD should have a version of `v1alpha1` with a schema that includes two properties as given below:  
> `deploymentName` (a string type) and `replicas` (an integer type with minimum value of 1 and maximum value of 5).
  
It should also include a `status` **subresource** which enables retrieving and updating the status of **Foo** object, including the `availableReplicas` property, which is an `integer` type.  
The **Foo** resource should be `namespace` **scoped**.  
`Note`: We have provided a template `/root/foo-crd-aecs.yaml` for your ease. There are few issues with it so please make sure to incorporate the above requirements before deploying on cluster.
###### Solution
```yml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: foos.samplecontroller.k8s.io
  annotations:
    "api-approved.kubernetes.io": "unapproved, experimental-only"
spec:
  group: samplecontroller.k8s.io
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
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
      subresources:
        status: {}
  scope: Namespaced
  names:
    plural: foos
    singular: foo
    kind: Foo
    shortNames:
    - foo
```
all the resources having groups endings like k8s.io are protected by k8s so to be able to create a CRD in that group we need to provide annotaiotn
```yml
"api-approved.kubernetes.io": "unapproved, experimental-only"
```
###### Q18
Create a new pod with image `nginx` and name `ckad-probe-aom` and configure the pod with `livenessProbe` with `command` `ls` and set `initialDelaySeconds` to 5 .
###### Solution
```yml
apiVersion: v1
kind: Pod
metadata:
  name: ckad-probe-aom
spec:
  containers:
  - image: nginx
    name: ckad-probe-aom
    livenessProbe:
      exec:
        command:
        - ls
      initialDelaySeconds: 5
```
###### Q19
A pod called `kodekloud-logs-aom` is running in the default namespace. It has two containers; get the logs of the `sidecar` container and copy them to `/root/ckad-exam.aom` on student-node.
###### Solution
```sh
kubectl logs kodekloud-logs-aom -c sidecar > /root/ckad-exam.aom
```
###### Q20
Three pods `hulk,thor and ironman` were created on `cluster1`. Of the three pods, identify the following and copy them to below file,
- Pod with high memory usage
- **Memory** limit configured to the identified pod.  
    copy them as `Podname,Memorylimit` to `/root/pod-metrics` file on student-node.
###### Solution
```sh
kubectl top pod
kubectl get pod hulk -o yaml | grep memory
```
###### Q21
Update the newly created pod `analytics-app` with a `readinessProbe` using the given specifications.  
Configure an `HTTP` readiness probe to the existing pod simple-webapp with `path` value set to `/ready` and `port` number to access container is `8080`.
###### Solution
```yml
    readinessProbe:
      httpGet:
        port: 8080
        path: /ready
      initialDelaySeconds: 15
      periodSeconds: 10
```