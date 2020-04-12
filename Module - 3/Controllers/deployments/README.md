# Deployment Controller

* Create a deployment 

```
kubectl create -f app1.yaml --record
```

* Verify the deployment is complete

```
kubectl get pods 
NAME                          READY   STATUS    RESTARTS   AGE
nginx-app1-56f7b66c7d-5vxk9   1/1     Running   0          10s
nginx-app1-56f7b66c7d-bz4td   1/1     Running   0          10s
nginx-app1-56f7b66c7d-vqpjg   1/1     Running   0          10s


kubectl get deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app1   3/3     3            3           48s

```

* Verify the status of deployment

```
kubectl rollout status deployment nginx-app1
```

* Expose the deployment as ClusterIP

```
kubectl expose deploy nginx-app1 --port=80 --type=ClusterIP
service/nginx-app1 exposed
```

* Verify successful creation of service 

```
kubectl get svc

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-app1   ClusterIP      10.101.30.208   <none>        80/TCP         24s

curl 10.101.30.208
You have reached app1

```

* Edit the same yaml file and add `minReadySeconds ` parameter and set it to `30` seconds 

```
Before - 

spec:
  replicas: 3
  selector:
  
  After
  
spec:
  replicas: 3
  minReadySeconds: 30
  selector:
```

* Apply the latest configuration 

```
kubectl apply -f app1.yaml 
```

* Verify minReadySeconds is available 

```
kubectl describe deploy nginx-app1 | grep -i minreadyseconds
MinReadySeconds:        30
```

* Use command line to perform a rolling update

```
kubectl set image deployments nginx-app1 app1=harshal0812/nginx-app2 
```
> Note - app1 = container name 

* Verify that rollingUpdate happens in accordance with the minReadySeconds criteria. 

```
kubectl get pods 
curl http://SERVICE_UP
```

* Check the history of all the deployments performed 

```
kubectl rollout history deployment nginx-app1
```

* Rollback to the last deployed version version 

```
kubectl rollout undo deployments nginx-app1
```

* Pause the rollout 

```
kubectl rollout pause deployment nginx-app1
```

* Resume the rollout

```
kubectl rollout resume deployment nginx-app1
```












