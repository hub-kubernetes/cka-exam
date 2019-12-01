Step1: First download the dashboard yaml file
```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml
```

Step2: Apply the dashboard yaml file
```
kubectl apply -f recommended.yaml
```

Step3: Wait for few minutes and then check whether dashboard pods are up and running
```
root@kubemaster:~# kubectl get po -n kubernetes-dashboard -o wide
NAME                                        READY   STATUS    RESTARTS   AGE   IP           NODE        NOMINATED NODE   READINESS GATES
dashboard-metrics-scraper-fb986f88d-wh4v2   1/1     Running   0          25m   10.244.2.2   kubenode1   <none>           <none>
kubernetes-dashboard-6bb65fcc49-svhrh       1/1     Running   0          25m   10.244.1.2   kubenode2   <none>           <none>
root@kubemaster:~# 
```
Notice that dashboard pod is created in `kubernetes-dashboard` namespace and also note down the node on which it is running

Step4: Now get the endpoint by running following command
```
root@kubemaster:~# kubectl -n kubernetes-dashboard get endpoints -o wide
NAME                        ENDPOINTS         AGE
dashboard-metrics-scraper   10.244.2.2:8000   20m
kubernetes-dashboard        10.244.1.2:8443   20m
```

Step5: Login to kubenode2, and run `kubectl proxy`

Step6: Open browser window and open link: 10.244.1.2:8443, this will open up kubernetes dashboard

Step7: Now on master node, generate a token to login to dashboard
```
root@kubemaster:~# kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token:
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1nNmpxOCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImVkZDA4Y2EyLWI4NzgtNDY0MC04OGQ2LTAwNzJlZmM5ZDE1OSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.G0t6wn6-NW4aLZ7-Jr-79R8zxPDJkxkqfHjVOeRDbW5h44OlFOMGsqf-bO2olm-81_XFnIlLmoQOgT1be00smx1wSxVGkVRzEe7CzcKrykjE3DC5gDpxvfFxdpLHk1pH8UFhyoHSGOwACOXCdiWtvACgEO6glCKEpev62GCEHDL8u0CjLlOgP1PRa5ZE6f3SKkW1FoRn8CYZWRFS55duqD33prUofGvxr9tl3j_bj41tA4NbVrfwNGGj809XF1tfhKU0ZUZK1Ehdw-TQWnPO0U0zGHqK2pub7QdallKOhDBq7okIe09v291mYPgm9xmWv_Lr9fAhlOFDSstIlk3tKA
```

Step8: Login with token on kubenode2
