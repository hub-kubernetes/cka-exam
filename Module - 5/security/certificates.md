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
Can't load /root/.rnd into RNG
```

















