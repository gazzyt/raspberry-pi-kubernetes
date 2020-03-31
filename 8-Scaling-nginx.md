Now we have two nodes in our cluster we can scale our Nginx pod to run on both nodes.
Update the `deployment.yaml` we created in step 6 and change `replicas: 1` to `replicas: 2`. Then apply the change with:

```
pi@k3sserver:~ $ sudo kubectl apply -f deployment.yaml
deployment.apps/web-server configured
```

After a minute or two check what happened:
```
pi@k3sserver:~ $ sudo k3s kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
svclb-http-server-7bsvx        1/1     Running   6          22d   10.42.0.101   k3sserver   <none>           <none>
hello-world-6df9f4cc87-hmh5d   1/1     Running   6          31d   10.42.0.94    k3sserver   <none>           <none>
svclb-hello-world-87gsq        1/1     Running   6          31d   10.42.0.92    k3sserver   <none>           <none>
web-server-657cd8448f-xkxzq    2/2     Running   12         16d   10.42.0.100   k3sserver   <none>           <none>
svclb-http-server-pfxcg        1/1     Running   3          15d   10.42.1.15    k3snode1    <none>           <none>
svclb-hello-world-tt9w5        1/1     Running   3          15d   10.42.1.14    k3snode1    <none>           <none>
web-server-657cd8448f-6bbpk    2/2     Running   0          83s   10.42.0.103   k3sserver   <none>           <none>
```

There are now two pods running nginx (`web-server-657cd8448f-xkxzq` & `web-server-657cd8448f-6bbpk`) but they are both running on `k3sserver` which is not what we wanted.

The pod scheduler does not guarantee to schedule pods on different nodes but it does try to spread them out if it can.

In this case we have both pods using `persistentVolumeClaim local-path-pvc` which (because it is from the `local-path` storage class) is located on `k3sserver`. Because of this, the scheduler was forced to schedule both replicas on that node.

To fix this we need to use a `statefulSet` instead of a `deployment` as this allows us to use `volumeClaimTemplates` to create new `persistentVolumeClaims` for each replica:

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-server
  namespace: default
  labels:
    app: web-server
spec:
  selector:
    matchLabels:
      app: web-server
  replicas: 2
  serviceName: http-server
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
  volumeClaimTemplates:
  - metadata:
      name: web-root
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: local-path
      resources:
        requests:
          storage: 1Gi
```

Clean up the previous deployment (but leave the service):

```
pi@k3sserver:~ $ sudo kubectl delete deployment web-server
deployment.apps "web-server" deleted

pi@k3sserver:~ $ sudo kubectl delete pvc local-path-pvc
persistentvolumeclaim "local-path-pvc" deleted
```

Apply the changes:

```
pi@k3sserver:~ $ sudo kubectl apply -f statefulset.yaml
statefulset.apps/web-server created
```

After a minute or two check what happened:

```
pi@k3sserver:~ $ sudo kubectl get pod -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
svclb-http-server-7bsvx        1/1     Running   6          22d   10.42.0.101   k3sserver   <none>           <none>
hello-world-6df9f4cc87-hmh5d   1/1     Running   6          32d   10.42.0.94    k3sserver   <none>           <none>
svclb-hello-world-87gsq        1/1     Running   6          32d   10.42.0.92    k3sserver   <none>           <none>
svclb-http-server-pfxcg        1/1     Running   3          15d   10.42.1.15    k3snode1    <none>           <none>
svclb-hello-world-tt9w5        1/1     Running   3          15d   10.42.1.14    k3snode1    <none>           <none>
web-server-0                   2/2     Running   0          78s   10.42.0.106   k3sserver   <none>           <none>
web-server-1                   2/2     Running   0          70s   10.42.1.18    k3snode1    <none>           <none>
```

The pods running nginx (`web-server-0` & `web-server-1`) are now running on different nodes.

We now have two `persistentVolumes`:
```
pi@k3sserver:~ $ sudo kubectl get pv -o wide
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                           STORAGECLASS   REASON   AGE   VOLUMEMODE
pvc-2345b0eb-dcd7-4bf8-a26f-2ee2aaaf5bf7   1Gi        RWO            Delete           Bound    default/web-root-web-server-0   local-path              15m   Filesystem
pvc-85104e4c-405a-4b65-8f39-ff01ae51846c   1Gi        RWO            Delete           Bound    default/web-root-web-server-1   local-path              14m   Filesystem
```

Describing these shows they are located on different nodes:
```
pi@k3sserver:~ $ sudo kubectl describe pv pvc-2345b0eb-dcd7-4bf8-a26f-2ee2aaaf5bf7
Name:              pvc-2345b0eb-dcd7-4bf8-a26f-2ee2aaaf5bf7
...
Node Affinity:
  Required Terms:
    Term 0:        kubernetes.io/hostname in [k3sserver]
...

pi@k3sserver:~ $ sudo kubectl describe pv pvc-85104e4c-405a-4b65-8f39-ff01ae51846c
Name:              pvc-85104e4c-405a-4b65-8f39-ff01ae51846c
...
Node Affinity:
  Required Terms:
    Term 0:        kubernetes.io/hostname in [k3snode1]
...
```