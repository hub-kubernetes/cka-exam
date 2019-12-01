# RBAC-User-Management
RBAC_USER_MANAGEMENT

##  Retrieve ca.key and ca.crt from:
        
        kubeadm clusters - certificates are found inside /etc/kubernetes/pki on master
        
        manually deployed clusters - These are the kubernetes apiserver certificates that might be deployed at custom location
    
    mkdir usercluster
    
    cp ca.key ca.crt usercluster/
    
    adduser user
    
    cp -R usercluster ~user/
    
    chown user:user ~user/usercluster
    
    
##  Generate a new CA for the user 

    Login to the newly created user account 
    
    cd usercluster 
    
    openssl genrsa -out user.key 2048
    
    openssl req -new -key user.key -out user.csr -subj "/CN=user/O=dev"
    
    openssl x509 -req -in user.csr  -CA ca.crt -CAkey ca.key -CAcreateserial -out user.crt -days 500
    
##  Create RBAC policies for user account access

    role.yaml
    
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      namespace: office
      name: deployment-manager
    rules:
    - apiGroups: ["", "extensions", "apps"]
      resources: ["deployments", "replicasets", "pods"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
      
      
    rolebinding.yaml
    
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: deployment-manager-binding
      namespace: office
    subjects:
    - kind: User
      name: user
      apiGroup: ""
    roleRef:
      kind: Role
      name: deployment-manager
      apiGroup: ""

    As root user that has access to kubectl execute - 
    
    kubectl create ns office
    
    kubectl create -f role.yaml -f rolebinding.yaml
    
##  Configure access for user account using kubectl 

    kubectl config set-cluster usercluster --server=https://api.kubernetesfederatedcluster.com
    
    kubectl config set-cluster usercluster --certificate-authority=ca.crt
    
    kubectl config set-credentials user --client-key=user.key --client-certificate=user.crt
    
    kubectl config set-context userspace --cluster=usercluster --namespace=office --user=user
    
    kubectl config use-context userspace

##  Smoke Test access policy for user 

    user@ip-172-31-33-248:~$ kubectl run nginx --image=nginx
    kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
    deployment.apps/nginx created
    
    user@ip-172-31-33-248:~$ kubectl get pods -o wide
    NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE
    nginx-65899c769f-jhv89   1/1     Running   0          20s   100.96.2.4   ip-172-20-37-228.ec2.internal
    
    user@ip-172-31-33-248:~$ kubectl get deploy -o wide
    NAME    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
    nginx   1         1         1            1           52s   nginx        nginx    run=nginx
    
    user@ip-172-31-33-248:~$ kubectl get pods --all-namespaces
    Error from server (Forbidden): pods is forbidden: User "user" cannot list pods at the cluster scope
    
##  Distribute .kube across multiple users 

    adduser employee 
    
    sudo -iu employee 
    
    cp ~user/.kube . 
    
    employee@ip-172-31-33-248:~$ kubectl get pods
    NAME                     READY   STATUS    RESTARTS   AGE
    nginx-65899c769f-jhv89   1/1     Running   0          5m
    
    employee@ip-172-31-33-248:~$ kubectl get pods --all-namespaces
    Error from server (Forbidden): pods is forbidden: User "user" cannot list pods at the cluster scope






    
    
    
    

    

    
