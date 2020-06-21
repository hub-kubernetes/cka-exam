# Ingress on GCP

## Deploying Applications 

```
kubectl create -f app1.yaml -f app2.yaml -f app3.yaml -f service.yaml
```

Verify that individual nodePorts are reachable 

```
$ curl 104.197.208.22:32500
You have reached app1
$ curl 104.197.208.22:31651
You have reached app2
$ curl 104.197.208.22:32617
You have reached app3

```

## Configure Ingress 

GCP uses its internal LoadBalancer as the primary ingress controller. For OnPrem solutions you can use external ingress solutions that are available on top of nginx, haproxy, traefik 

* Create a Fanout Ingress rule to map the 3 services 

```
cat << EOF | tee -a fanout.yaml

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: fanout-ingress
spec:
  rules:
  - http:
      paths:
      - path: /app1
        backend:
          serviceName: app1
          servicePort: 80
      - path: /app2
        backend:
          serviceName: app2
          servicePort: 80
      - path: /app3
        backend:
          serviceName: app3
          servicePort: 80
EOF
```

Get the ingress IP address

```
kubectl get ingress
fanout-ingress   *       34.120.243.0   80      6m42s

```

