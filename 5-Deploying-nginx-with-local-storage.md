We will now launch a container that uses some local storage outside of the container. Kubernetes has many ways to specify and manage storage but we will use a K3S included plugin named Local Path Provisioner which provides disk based storage on the node itself and is dynamically provisioned.
This time we will deploy the configurations via YAML files.

File `pvc.yaml` creates the persistent volume claim allowing pods to request 1GB of local persistent storage. The accessMode of ReadWriteOnce means only one node can mount the volume in read/write mode - this is the only mode supported by the Local Path Provisioner.

#### pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-path-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 1Gi
```

File `deployment.yaml` creates a pod running NGINX. NGINX listens on port 80 and we map the persistent volume so that NGINX serves files from it.

#### deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
  namespace: default
  labels:
    app: web-server
spec:
  selector:
    matchLabels:
      app: web-server
  replicas: 1
  template:
    metadata:
      labels:
        app: web-server
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: web-root
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
      volumes:
      - name: web-root
        persistentVolumeClaim:
          claimName: local-path-pvc
```

File `service.yaml` defines a service that exposes our web server outside of the cluster. It is exposed on port 8081 on localhost of the node. Because we don't specify a `NodePort`, K3S will choose an available port to expose the web server on its real ip address.

#### service.yaml
```
kind: Service
apiVersion: v1
metadata:
  name: http-server
  namespace: default
  labels:
    app: http-server
spec:
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 80
  selector:
    app: web-server
  type: LoadBalancer
  sessionAffinity: None
  externalTrafficPolicy: Cluster
```

Create the objects from the YAML:
```
$ sudo kubectl create -f pvc.yaml
persistentvolumeclaim/local-path-pvc created
$ sudo kubectl create -f deployment.yaml
pod/web-server created
$ sudo kubectl create -f service.yaml
service/http-server created
```

Check which port the web server is exposed on:
```
$ sudo kubectl get service http-server
NAME          TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
http-server   LoadBalancer   10.43.182.166   192.168.1.171   8081:31811/TCP   75m
```

And fetch the homepage:
```
$ curl http://k3sserver:31811/
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.16.1</center>
</body>
</html>
```

We accessed the web server successfully but got a 403 error because the web root directory is empty and directory listing is disabled. To fix this we need to find where the persistent storage is located:
```
$ sudo kubectl get persistentvolumes
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
pvc-4b1f3e29-c3a5-4d7d-88ae-cff5cfe90e08   1Gi        RWO            Delete           Bound    default/local-path-pvc   local-path              81m
```

So this one is at
`/var/lib/rancher/k3s/storage/pvc-4b1f3e29-c3a5-4d7d-88ae-cff5cfe90e08`.

Create a simple html file named index.html and copy it into the persistent volume folder. The following file content will do:

#### index.html
```
<html>
<head>
<title>NGINX Test Page</title>
</head>
<body>This page was served by NGINX running in K3S</body>
</html>
```

Now fetch the homepage again:
```
$ curl http://k3sserver:31811/
<html>
<head>
<title>NGINX Test Page</title>
</head>
<body>This page was served by NGINX running in K3S</body>
</html>
```