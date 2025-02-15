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

