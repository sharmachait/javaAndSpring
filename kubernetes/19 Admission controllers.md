## admission controllers
all the roles are only restrictions to the k8s api

but we may want to be able to restrict k8s itself to use images from an internal registry only
or to restrict pods from being run as root users
or that metadata for each resource always contains labels

such restrictions can not be achieved with roles

admission controllers help us achieve control over such details

#### list of admission controllers

1. AlwaysPullImages - to enforce that local images are not used and a new image is always pulled from the registry
2. DefaultStorageClass - automatically adds the default storage class to PVCs as they are created
3. EventRateLimit - to rate limit the number of requests the api server will handle
4. NamespaceExists - restricts requests to namespaces that dont exist, enabled by default
5. NamespaceAutoProvision - when enabled creates a namespace if doesnt exist for every api request

to see the list of admission controllers enabled by default

> kube-apiserver -h | grep enable-admission-plugins

this may need to be run in the control plane node
can be achieved with
> kubectl exec kube-apiserver-controlplane-pod -n kube-system -- kube-apiserver -h | grep enable-admission-plugins

or if the kube-apiserver is running as a pod

> ps -ef | grep kube-apiserver | grep admission-plugins

we can exec into a pod with
>`kubectl exec -it <pod-name> -- /bin/bash`

if a pod has multiple containers it will ask for container name
> `kubectl exec -it <pod-name> -c <container-name> -- /bin/bash`

to enable or disable a new admission controller it can be added to the pod manifest of the kube-apiserver

we can see the enabled admission controllers with 
> kubectl get pod kube-apiserver-local-control-plane -n kube-system -o yaml

```yml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
  labels:
    component: kube-apiserver
    tier: control-plane
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.28.0  # Replace with your Kubernetes version
    command:
    - kube-apiserver
    - --advertise-address=192.168.0.1  # Replace with your control plane node IP
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
    - --disable-admission-plugins=DefaultStorageClass
```

to change the kube-api-server we can edit the manifest file at `/etc/kubernetes/manifests/kube-apiserver.yaml`
kubernetes automatically reads it and updates the server, we can add other admission controllers as comma separated values

NamespaceExists and NamespaceAutoprovision controllers are deprecated and NamespaceLifecycle controller is recommended, it also makes sure that the dafault and k8s's namespaces can not be deleted

### validating vs mutating admission controllers

Mutating controllers are controllers that can modify the original request that was sent to the kube api server, like the DefaultStorageClass controller

Validating controllers are controllers that validate a request before it is allowed to be processed, NameSapceExists controller, ImagePolicy controller

Generally mutating controllers are invoked first so that the request has reached its final form before being validated

the NamespaceAutoProvision controller should be invoked before NamespaceExists controller, because if it doesnt exist how can it be validated

## MutatingAdmissionWebhook and ValidatingAdmissionWebhook controllers
to apply some custom logic as a custom admission controller
it can be achieved via MutatingAdmissionWebhook and ValidatingAdmissionWebhook

we can make these webhooks be pointed to a server either inside the cluster itself or to some third part server

the controller sends a an admission review object to the webhook which can process the admission request and validate or mutate the request and return to the controller

the admission request object is in json format, its part of apiVersion `admission.k8s.io/v1`
and has all the details about the request being sent to the kube api server

the third party server that processes the admission request returns with and admissionReview object as well with the result of the processing and allowed status of true or false
the code for the webhook processing server can be deployed anywhere, it must have a validate and a mutate route for respective controllers
can be deployed with in the k8s cluster itself, it would need a service aswell to communicate with it
a filter that communicates with the webhook can be configured by making a webhook configuration object on our k8s cluster

in case of third-party deployment of the webhook servere
```yml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  clientConfig:
    url: "https://external-server.example.com"
  rules: 
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
```

in case the server is deployed on the cluster itself
```yml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  clientConfig:
    service:
      namespace: "webhook-namespace"
      name: "webhook-service"
    caBundle: "jahgfhho3979v.......asdf127"
  rules: 
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
```