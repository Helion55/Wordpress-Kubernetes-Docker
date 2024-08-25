# Wordpress-Mysql Setup on Kubernetes
  
![Diagram](Wordpress-Kubernetes-diagram.jpg)

## Project Overview
A Wordpress application storing data in MySQl database running on a kubernetes cluster using the Kubernetes objects Deployment, service, Persistent Volume and Namespace. The application also running on a Docker Compose Stack.
## Tech Stack
- Wordpress
- MySql
- Docker
- Kubernetes

## Wordpress 
Wordpress Image is used to run the docker compose stack first. Having the environment variables
- WORDPRESS_DB_HOST
- WORDPRESS_DB_USER
- WORDPRESS_DB_PASSWORD
- WORDPRESS_DB_NAME
On Docker using a docker volume and on Kubernetes a persistent volume and persistent volume claim wordpress is storing its data, running on port 8080 .

## MySQL 
The MySQL database image is pulled and configuring these environment variables
- MYSQL_DATABASE
- MYSQL_USER
- MYSQL_PASSWORD
- MYSQL_RANDOM_ROOT_PASSWORD
On Docker using a docker volume and on Kubernetes a persistent volume and persistent volume claim wordpress is storing its data, running on port 3306 .

## Docker
Using this Docker Compose file to deploy the stack
```yaml
version: "3.1"
services:

 app:
  image: wordpress:6.4.2-php8.1-apache
  container_name: con_name
  restart: always
  ports:
   - 8080:80
  environment:
   WORDPRESS_DB_HOST: db_host
   WORDPRESS_DB_USER: user
   WORDPRESS_DB_PASSWORD: password
   WORDPRESS_DB_NAME: db_name
   
  volumes:
   - wordpress:/var/www/html
   
 db:
  image: mysql:8.0
  container_name: name
  restart: always
  environment:
   MYSQL_DATABASE: db_name
   MYSQL_USER: user
   MYSQL_PASSWORD: password
   MYSQL_RANDOM_ROOT_PASSWORD: '1'
  volumes:
   - db:/var/lib/mysql
   
volumes:
 wordpress:
 db:
```
## Kubernetes
### Deployment file for
- Wordpress
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: webdeployment
spec:
 selector:
  matchLabels:
   app: website
 template:
  metadata:
   labels:
    app: website
  spec:
   containers:
   - name: web
     image: wordpress:6.2.1-apache
     ports:
      - containerPort: 8080
     env:
      - name: WORDPRESS_DB_HOST
        value: mysql
      - name: WORDPRESS_DB_USER
        value: admin
      - name: WORDPRESS_DB_PASSWORD
        value: password
      - name: WORDPRESS_DB_NAME
        value: mysql
     volumeMounts:
     - name: wpclaim
       mountPath: /var/www/html   
        
   volumes: 
   - name: wpclaim    
     persistentVolumeClaim:
      claimName: wpclaim
```
- MySQL
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: perclaim
          mountPath: /var/lib/mysql
      volumes:
      - name: perclaim
        persistentVolumeClaim:
          claimName: perclaim
```
### Service files for 
- Wordpress
Loadbalancer Type service is used as this deployment is to run on cloud
```yaml
apiVersion: v1
kind: Service                    
metadata:
 name: webservice 
spec:
 type: LoadBalancer
 selector:
  app: website
 ports:
 - port: 8080
   targetPort: 80
```

- MySQL
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  clusterIP: None
  ports:
  - port: 3306
```
### Persistent Volume and Persistent Volume Claim files for
- Wordpress
Persistent Volume
```yaml

apiVersion: v1
kind: PersistentVolume
metadata:
 name: wpvolume
spec:
 storageClassName: volume
 capacity:
  storage: 500Mi
 accessModes:
  - ReadWriteOnce
 hostPath:
  path: "wp/"
```

Persistent Volume Claim
```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wpclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: volume
```

- MySQL
Persistent Volume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: pervol
spec:
 storageClassName: store
 capacity:
  storage: 500Mi
 accessModes:
  - ReadWriteOnce
 hostPath:
  path: "files/"
```
Persistent Volume Claim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: perclaim
spec:
 storageClassName: store
 accessModes:
  - ReadWriteOnce
 resources:
  requests:
   storage: 100Mi
```
### Namespace file
```yaml
apiVersion: v1
kind: Namespace
metadata:
 name: music
```

These are the manifest files to deploy Wordpress and MySQL...



