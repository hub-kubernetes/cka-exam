# helm

# HELM - The Package Manager for Kubernetes 

##  Helm installation 
    
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## Add the stable repository details 

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com
```

##  Helm Charts 

Structure of Helm Chart -
     
```
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  values.yaml         # The default configuration values for this chart
  values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file
  charts/             # A directory containing any charts upon which this chart depends.
  crds/               # Custom Resource Definitions
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
```


##  Using stable repository 

Helm stable repository has a bunch of applications for official images that you can download and deploy as a single package - 

The link is as below : 

https://github.com/helm/charts/tree/master/stable


Adding stable repository to helm 

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

```

Lets install jenkins using helm chart - 

`   helm install --name my-release stable/jenkins`

This puts pods in pending status because peristent volumes is missing - 

Lets delete this pod and modify few settings to disable persistent volumes- 

helm list 

```
helm list 
NAME            REVISION        UPDATED                         STATUS          CHART           APP VERSION     NAMESPACE
my-release      1               Wed May 22 19:10:44 2019        DEPLOYED        jenkins-1.1.21  lts             default  
```

```
helm delete my-release --purge 
release "my-release" deleted
```

WE will now download the jenkins repo, make some edits and then install jenkins 

```
mkdir jenkins 

cd jenkins 

helm fetch stable/jenkins 

ls -ltra

-rw-r--r--  1 root root 27981 May 22 19:14 jenkins-1.1.21.tgz


tar xvf jenkins-1.1.21.tgz

cd jenkins/

vi values.yaml 

```

We will now edit values.yaml as per https://github.com/helm/charts/tree/master/stable/jenkins

`   vi values.yaml` 

Search for Persistent

Change `enabled: true` to `enabled: false`

```
cd ..

helm install ./jenkins
```

It might take couple of mins to load. Check the notes from NOTES.txt to check the details to login to jenkins 


>   1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace default veering-walrus-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        You can watch the status of by running 'kubectl get svc --namespace default -w veering-walrus-jenkins'
  export SERVICE_IP=$(kubectl get svc --namespace default veering-walrus-jenkins --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
  echo http://$SERVICE_IP:8080/login

3. Login with the password from step 1 and the username: admin

Username is : admin 

Get the Password by - 

```   
printf $(kubectl get secret --namespace default veering-walrus-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
YHeHhKXGx0
```

Get the NodePort by - 
```   
kubectl get svc | grep jenkins 
veering-walrus-jenkins         LoadBalancer   10.96.22.82    <pending>     8080:30429/TCP      4m40s 
```

Login to jenkins by going to - http://EXTERNAL_IP_ADDR:30429/


Lets now do one more demo - 

`   cd ~/ `

` mkdir prometheus && cd prometheus` 

` helm fetch stable/prometheus`

```
ls -ltr
total 24
-rw-r--r-- 1 root root 22460 May 22 19:33 prometheus-8.11.4.tgz

tar xvf prometheus-8.11.4.tgz

cd prometheus/

```

We will now edit the values.yaml to disable all persistent volumes - 

` vi values.yaml ` 

Search for persistentVolume and change ` enabled: true` to ` enabled: false`. There will be 2 instances - one for prometheus-server and second for alert manager 

` cd .. ` 

` helm install ./prometheus `

Copy the output on wordpad

` helm fetch stable/grafana ` 

` tar xvf grafana-3.3.8.tgz `

` cd grafana ` 

` kubectl get svc | grep -i prometheus-server ` 

Get the name of the service of prometheus server service 

```
kubectl get svc | grep -i prometheus-server
whopping-uakari-prometheus-server               ClusterIP      10.107.135.75    <none>        80/TCP              14m
```

` vi values.yaml `


Install grafana - 

` helm install ./grafana `

Save the output from grafana on wordpad 

Get the nodeport port for grafana service - 

```
kubectl get svc | grep grafana
jaunty-chinchilla-grafana                       NodePort       10.107.211.81    <none>        80:30405/TCP        9m29s
```

Access the grafana dashboard using http://EXTERNAL_IP_ADDRESS:30405 on your dashboard

Login using user: admin 

Get the password using the NOTES that you got using grafana - 

```
kubectl get secret --namespace default jaunty-chinchilla-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
fRBtwZOKIbwxAuW50K4flyRszHORnagEiOQMIdkw
```

On the Datasource section - Add datasource 

use the datasource as prometheus 

for the URL - http://SERVICE_NAME_OF_PROMETHEUS_SERVER

` http://whopping-uakari-prometheus-server`

Click Save and Test 

Click the + Sign and select Import 

We will now import the official grafana dashboard for promethus - 

Put - ` 159 ` or `6417` or `3662` in the Grafana Dashboard section and it will upload the pre-existing dashboard 

Give a name for the dashboard 

Name : my grafan dashboard

In the Prometheus datasource section select the correct datasource - 

Prometheus : Prometheus 

Click Import and view the dashboard















         
    
        
    
    

    
        
    

   
     




