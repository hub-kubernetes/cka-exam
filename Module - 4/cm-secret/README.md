# Configmaps and Secrets 

This demo uses configmaps and secrets to deploy MariaDB cluster 

Create a directory to store all files - ` mkdir mariadbdemo`

---

## Create a secret  

Secrets are stored as base64 encoded objects. For the demo our first secret will be the MariaDB password - 

```
echo -n "mariadbpass" | base64 
bWFyaWFkYnBhc3M=
```

Create a secret resource to store the password - 

```
cat << EOF | tee -a mariadbpass.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-root-password
type: Opaque
data:
  password: bWFyaWFkYnBhc3M=
EOF
```

Alternatively you can also use - 

```
kubectl create secret generic  mariadb-root-password --from-literal=password=mariadbpass
```

Create the secret 

```
kubectl create -f mariadbpass.yaml
```

Lets create one more secret which entails the MariaDB non-root user and the password 

```
kubectl create secret generic mariadb-user-creds \
      --from-literal=MYSQL_USER=mariadbuser \
      --from-literal=MYSQL_PASSWORD=mariadbpass
```
---

## Create a ConfigMap

Create a file that holds all environment variables required for the instance to execute 

```
cat <<EOF | tee -a max_allowed_packet.cnf
[mysqld]
max_allowed_packet = 64M
EOF
```

Load this file to kubernetes as a configmap 

```
kubectl create configmap mariadb-config --from-file=max_allowed_packet.cnf
```

---

## MariaDB Deployment 

In this deployment - 

* Secrets will be loaded as environment variable 
* Configmaps will be loaded as file mounts 

Note that You can also load ConfigMaps as environment variables and you can also load secrets as volumes

```
cat <<EOF | tee -a mariadbdeploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mariadb
  name: mariadb-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - image: docker.io/mariadb:10.4
        name: mariadb
        env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mariadb-root-password
                key: password
        envFrom:
        - secretRef:
            name: mariadb-user-creds
        ports:
        - containerPort: 3306
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mariadb-volume-1
        - mountPath: /etc/mysql/conf.d
          name: mariadb-config-volume
      volumes:
      - emptyDir: {}
        name: mariadb-volume-1
      - configMap:
          name: mariadb-config
          items:
            - key: max_allowed_packet.cnf
              path: max_allowed_packet.cnf
        name: mariadb-config-volume

EOF
```

Verify the Deployment


















