# Kubernetes Certificates 

Kubernetes supports authentication for accounts using the below - 

* x509 certificates
* Username/Password
* OIDC
* Tokens
* LDAP/SAML with external plugins

In this demo we will see how to create a certificate for a new external user who is a part of the `network-admin` group. 


* Create private key 

Lets first create a directory `certs` to store all the certificates 

```
mkdir certs
```

Execute the below command to create the private key 

```
openssl genrsa -out certs/user.key
chmod 400 certs/user.key
```

* Create a CSR 

The below command creates a CSR (certificate signing request). Please note the below details - 

1. CN = name of the physical user account 
2. O = name of the internal kubernetes group

```
openssl req -new -key certs/user.key -out certs/user.csr -subj "/CN=networkadmin/O=network-admin"
```

* Create a Kubernetes CSR resource YAML 

```
cat > certs/networkadmin-user.yaml <<EOF
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: new-user-request
spec:
  request: $(cat certs/user.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF

```

Create the CSR resource

```
kubectl create -f certs/networkadmin-user.yaml
certificatesigningrequest.certificates.k8s.io/new-user-request created

```

* Verify the created CSR 

```
kubectl get csr
NAME               AGE   SIGNERNAME                                    REQUESTOR                 CONDITION
csr-qcvdd          34m   kubernetes.io/kube-apiserver-client-kubelet   system:node:master        Approved,Issued
csr-v6txd          34m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:0dpdie   Approved,Issued
csr-vjh88          34m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:0dpdie   Approved,Issued
new-user-request   21s   kubernetes.io/legacy-unknown                  kubernetes-admin          Pending
```

* Approve the cer

```
kubectl certificate approve new-user-request


kubectl get csr
NAME               AGE   SIGNERNAME                                    REQUESTOR                 CONDITION
csr-qcvdd          36m   kubernetes.io/kube-apiserver-client-kubelet   system:node:master        Approved,Issued
csr-v6txd          35m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:0dpdie   Approved,Issued
csr-vjh88          35m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:0dpdie   Approved,Issued
new-user-request   83s   kubernetes.io/legacy-unknown                  kubernetes-admin          Approved,Issued
```


* Get the certificate file - 

```
kubectl get csr new-user-request -o jsonpath='{.status.certificate}' \
  | base64 --decode > certs/networkadmin.crt

```

Verify the details using - 

```
openssl x509 -noout -text -in networkadmin.crt 
```

* Create credentials for networkadmin user  

Create a new entry for your .kube/config file 

```
kubectl config set-credentials networkadmin\
  --client-certificate=certs/networkadmin.crt \
  --client-key=certs/user.key \
  --embed-certs
```

Add a new context for the networkadmin user 

```
kubectl config set-context network-admin --cluster=kubernetes --user=networkadmin

kubectl config use-context network-admin

```

Execute some network commands to see if you are able to access networking 

```
kubectl get networkpolicy
Error from server (Forbidden): networkpolicies.networking.k8s.io is forbidden: User "networkadmin" cannot list resource "networkpolicies" in API group "networking.k8s.io" in the namespace "default"
```

Error????? WHY ???

Rollback to admin context - 

```
kubectl config use-context kubernetes-admin@kubernetes
```

## Authorization Modules in Kubernetes 

Refer - https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules for different authorization modules in kubernetes

For our demo - we will use clusterrole in order to give a cluster level network policies access to the user 

```

cat > network-admin-role.yaml <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: network-admin
rules:
- apiGroups:
  - networking.k8s.io
  resources:
  - networkpolicies
  verbs:
  - '*'
- apiGroups:
  - extensions
  resources:
  - networkpolicies
  verbs:
  - '*'
EOF

```

Create the new clusterrole 

```
kubectl create -f network-admin-role.yaml

```


You can now create a clusterrolebinding to the group - network-admin

```
kubectl create clusterrolebinding network-admin --clusterrole=network-admin --group=network-admin

```


Verify the access again - 

Switch the context - 

```
kubectl config use-context network-admin
```

Execute the GET command in any namespace 

```
kubectl get networkpolicies
No resources found in default namespace.


kubectl get pods 
Error from server (Forbidden): pods is forbidden: User "networkadmin" cannot list resource "pods" in API group "" in the namespace "default"
```



























