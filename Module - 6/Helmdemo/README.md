# helm

# HELM - The Package Manager for Kubernetes 

##  Helm installation 
    
    a.  Install Helm package 
    
    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash
    
    root@ip-172-31-33-248:~# which helm
    /usr/local/bin/helm
    
    root@ip-172-31-33-248:~# which tiller
    /usr/local/bin/tiller
    
    b.  Configure Tiller
    
    Helm installs the tiller service on your cluster to manage charts.
    
    Since RBAC is our official mode of securing access, we will create RBAC policies for tiller to manage deployments
    
    kubectl -n kube-system create serviceaccount tiller

    kubectl create clusterrolebinding tiller \
     --clusterrole=cluster-admin \
     --serviceaccount=kube-system:tiller
     
    helm init --service-account tiller
    
    root@ip-172-31-33-248:~# kubectl get pods -n kube-system | grep -i tiller
    tiller-deploy-689d79895f-75swz                         1/1     Running   0          15s
    
##  Helm Charts 

    Structure of Helm Chart -
     
    mychart/
       Chart.yaml
       values.yaml
       charts/
       templates/
    
    1.  The templates/ directory is for template files. 
    
        When Tiller evaluates a chart, it will send all of the files in the templates/ directory through the template rendering engine. 
    
        Tiller then collects the results of those templates and sends them on to Kubernetes. 
       
    2.  The values.yaml file is also important to templates. 
    
        This file contains the default values for a chart.
        
        These values may be overridden by users during helm install or helm upgrade
        
    3.  The Chart.yaml file contains a description of the chart.
    
    4.  The charts/ directory may contain other charts or subcharts. 
    

##  Creating our First Chart 

    We will deploy an nginx application - 
    
      1.  nginx - deployed as a deployment 
      
      2.  nginx.conf and virtualhost.conf deployed as configmap 
      
      3.  Add configmap as volume to nginx.conf 
    
    
    a.  helm create nginxdemo

    b.  Files inside nginxdemo/templates
        
        1.  NOTES.txt - The “help text” for your chart. This will be displayed to your users when they run helm install.
        
        2.  _helpers.tpl: A place to put template helpers that you can re-use throughout the chart
    
        3.  rm -fr *
        
        4.  cp nginx* nginxdemo/templates/
        
        5.  echo "You are installing NGINX with CONFIGMAP as a volume" >  nginxdemo/templates/NOTES.txt
        
        RUN - 
        
        helm install ./nginxdemo
        
        6.  Note the random name - NAME:   killer-liger
        
        7.  helm get manifest killer-liger
        
        8.  helm delete killer-liger


##  Adding variables to HELM 

    We will recreate our nginx application to add some new variables 
    
    1.  helm create nginxdemo2
    
    2.  cd nginxdemo2/templates
    
    3.  rm -fr * 
    
    4.  cp nginx* nginxdemo2/templates/
    
    5.  cd nginxdemo2/templates/
    
    6.  vi nginx-configmap.yaml nginx-deployment.yaml
    
        a. nginx-configmap.yaml
        
            Modify name: nginx-conf to name: {{ .Release.Name }}-configmap
            
        b.  nginx-deployment.yaml
        
             Modify name: nginx-conf to {{ .Release.Name }}-configmap
             
    7.  What is .Release.Name ---
    
            Helm tracks all releases. Release is an built in object (like a class). It renders several parameters like 
            
            Release.Name
            
            Release.Namespace
            
            Release.Revision .. and so on. 
            
            name is the unique Random name that helm assigns to each release of the package
    
    8.  Run helm in dry run mode - 
        
        helm install --debug --dry-run ./nginxdemo2/
        
        Observations -
        
              name: washed-wolverine-configmap
        
        Change the files inside template directory and replace {{ .Release.Name }}-configmap with {{ .Release.Namespace }}-configmap
        
        Execute - helm install --debug --dry-run ./nginxdemo2/
        
        Observations - 
        
              name: default-configmap
    
    9.  helm install ./nginxdemo2
    
    10. Verify Installation
    
    11. helm list -- to get list of current releases
    
    12. helm delete RELEASE_NAME
    

##  Working with values.yaml

    1.  helm create nginxdemo3
    
    2.  rm -fr nginxdemo3/templates/*
    
    3.  cp nginxdemo2/templates/* nginxdemo3/templates/

    4.  cd nginxdemo3
    
    5.  vi values.yaml and remove all data from  the file. 
    
    6.  Add the below values to value.yaml
    
        nginximage: nginx:latest
        replicas: 3
        ports:
          containerport: 80
    
    7.  Make the below changes inside templates/nginx-deployment.yaml
    
            replicas: 1 to replicas: {{ .Values.replicas }}
            
            image: nginx to {{ .Values.nginximage }}
            
            - containerPort: 80 to - containerPort: {{ .Values.ports.containerport }}
            
    8.  helm install --debug --dry-run ./nginxdemo3
    
    9.  Verify the values of the manifest 

    10. helm install nginxdemo3

    11. helm list 
    
    12. helm delete RELEASE_NAME



##  Using helm Template Functions and Pipe operator 

    1.  Template functions are used to transform the supplied data from Values.yaml and make it more usable 
    
    2.  cp -R nginxdemo3  nginxdemo4
    
    3.  Add the below entry inside nginxdemo4/values.yaml
        
        nginxname: nginx

    3.  cd nginxdemo4/templates
    
    4.  Modify the below in nginx-deployment.yaml - 
    
            - name: nginx   to   - name: {{ quote .Values.nginxname }}
            
    5.  helm install --debug --dry-run nginxdemo4
        
        Observations: 
        
            - name: "nginx"   ----  quotes added 
    
    6.  Modify the below in nginx-deployment.yaml -
    
            - name: {{ quote .Values.nginxname }}    to   - name: {{  .Values.nginxname | upper | quote }}
            
    7.  helm install --debug --dry-run nginxdemo4
    
        Observations:
            
            - name: "NGINX"
            
    8.  Modify the below in nginx-deployment.yaml -
    
            - name: {{  .Values.nginxname | upper | quote }}  to   - name: {{  .Values.nginxname | upper | repeat 5 | quote }}
            
    9.  helm install --debug --dry-run nginxdemo4
    
            Observations:
            
            - name: "NGINXNGINXNGINXNGINXNGINX"
            
    9.  Modify the below in nginx-deployment.yaml -
    
            image: {{ .Values.nginximage }}    to    image: {{ .Values.nginximageNOTEXISTING | default "nginx:1.15.8" | quote }}
            
    10. helm install --debug --dry-run nginxdemo4
    
            Observations:
            
            image: "nginx:1.15.8"
            

##  Control Structures and Loops 

    1.  cp -R nginxdemo4 nginxdemo5
    
    2.  Edit values.yaml file as below - 
    
        nginximage: nginx:latest 
        
                    TO
        
        nginximagename: nginx
        nginximagetag: latest
        
    3.  Modify - nginxdemo5/templates/nginx-deployment.yaml as below 
    
            image: {{ .Values.nginximageNOTEXISTING | default "nginx:1.15.8" | quote }}
            
                                    TO
                                    
            image: {{ .Values.nginximagename }}:{{ .Values.nginximagetag }} 
            
    4.  helm install --debug --dry-run nginxdemo5
    
        Observations:
            
                    image: nginx:latest

    
    5.  Modify values.yaml as below 
    
            nginximagetag: latest  to  nginximagetag: 1.15.4
    
    5.  Modify - nginxdemo5/templates/nginx-deployment.yaml as below 
    
            image: {{ .Values.nginximagename }}:{{ .Values.nginximagetag }} 
            
                                    TO
                                    
            {{ if eq .Values.nginximagename "1.15.8" }}
            image: {{ .Values.nginximagename }}:{{ .Values.nginximagetag }}
            {{ else }}
            image: nginx:latest
            {{ end }}
    
    6.  helm install --debug --dry-run nginxdemo5
    
        Observations: 
            
            image: nginx:latest
            
    7.  Add below list to values.yaml
    
            deploymentlabels:
              app: frontend
              tier: webtier
              application: nginx
              
    8.  Modify - nginxdemo5/templates/nginx-deployment.yaml as below
            
            metadata:
              name: nginx
              
              
                                 TO
                                 
            metadata:
              name: nginx
              labels:
                {{- range $key, $val := .Values.deploymentlabels }}
                {{ $key }}: {{ $val | quote }}
                {{- end }}

    9.  helm install --debug --dry-run nginxdemo5
    
        Observations:
        
            metadata:
              name: nginx
              labels:
                app: "frontend"
                application: "nginx"
                tier: "webtier"

              
##  Helm Packaging 

Helm Packaging allows you to package your charts as a single tar file and upload it to your version control system. THis chart can then be downloaded like any other linux application using apt-get or yum from an external repository. 

```

On your version control - create a repository with the name helm-package - we will use github here. 

Switch to your  home directory 

cd ~/ 

git clone https://github.com/hub-kubernetes/helm-package.git

cd helm-package

Package your helm chart 

helm package ~/adobe-training/Helmdemo/nginxdemo/ 
 
 ls 
 
nginxdemo-0.1.0.tgz


helm repo index .

git add .


git commit -m " adding helm repo " 

git push

Open your github repository 

Click on settings 

Click on Github Pages 

Add source to master 


Your github repo now has a single blank file index.html

A link would be available like - https://hub-kubernetes.github.io/helm-package/ -- make a note of this 


add this repo to helm - 

helm repo add nginxdemo 'https://hub-kubernetes.github.io/helm-package/'

helm repo add nginxdemo 'https://hub-kubernetes.github.io/helm-package/'
"nginxdemo" has been added to your repositories

helm repo update

You can now search for your helm chart - 

helm search nginxdemo
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                
local/nginxdemo         0.1.0           1.0             A Helm chart for Kubernetes
nginxdemo/nginxdemo     0.1.0           1.0             A Helm chart for Kubernetes

```

##  Using stable repository 

Helm stable repository has a bunch of applications for official images that you can download and deploy as a single package - 

The link is as below : 

https://github.com/helm/charts/tree/master/stable

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















         
    
        
    
    

    
        
    

   
     




