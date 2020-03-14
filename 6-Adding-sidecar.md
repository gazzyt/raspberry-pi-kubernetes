Our nginx deployment suffered from the problem that it wasn't convenient to deploy and update the the web content. We will solve this via a sidecar container. A sidecar is a term for a supplemental container deployment in a pod alongside the main container. In our case the main container will be nginx and our sidecar will synchronise the web content from a github repository.

First we need a github repository containing some web content. I created one at `https://github.com/gazzyt/sidecar-content`.

Modify the `deployment.yaml` file we used previously thus:

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
          subPath: sidecar-content
        ports:
        - containerPort: 80
      - name: git-sync
        image: lostlakkris/git-sync:latest
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: web-root
          mountPath: /data
          subPath: sidecar-content
        env:
        - name: REPO
          value: https://github.com/gazzyt/sidecar-content.git
      volumes:
      - name: web-root
        persistentVolumeClaim:
          claimName: local-path-pvc
```

There are two changes here:

* We will clone the repository into a subfolder `sidecar-content` so we need to update the volume mapping for the nginx container by adding a `subPath` element.
* Add a new sidecar container.
  * Image `lostlakkris/git-sync` exists on Docker Hub and has an ARM version so we can use this rather than create our own. The container documentation is at https://github.com/LostLakkris/Dockerfiles/tree/master/s6.git-sync
  * We need to map the persistent volume into this container as well so it can write the html content files. `/data` is the path the container expects to see so map this to `sidecar-content`
  * Set an environment variable named `REPO` to contain the github repo url.

Update the deployment:
```
sudo kubectl apply -f deployment.yaml
```

Now fetch the homepage:
```
$ curl http://k3sserver:31811/
<html>
<head>
<title>NGINX Test Page</title>
</head>
<body>This page was served by NGINX running in K3S and synced from github</body>
</html>
```

By default, `git-sync` synchronises every 2 minutes so html changes we push to our repo will be made live within that time.