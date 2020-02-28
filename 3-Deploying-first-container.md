## Deploying Our First Container
Kubernetes has a nice [Hello Minikube tutorial](https://kubernetes.io/docs/tutorials/hello-minikube/) which would be a great place to start.
They provide a pre-built container image in their `create deployment` command but this didn't work on the PI. Presumably they don't have an ARM build so we'll have to create our own.

Helpfully they provide the source code on the page which I copied to my [GitHub here](https://github.com/gazzyt/hello-world) having updated the Dockerfile to update the version of NodeJS to the latest official available on Docker Hub.

To build an ARM container image for this and push it to Docker Hub I used another Raspberry PI which had Docker installed and did the following:
```
git clone https://github.com/gazzyt/hello-world

cd hello-world/

docker build -t gazzyt/hello-world:1.0 .

docker push gazzyt/hello-world:1.0
```

Returning to the K3S server, let's deploy our container:

```
pi@k3sserver:~ $ sudo kubectl create deployment hello-world --image=gazzyt/hello-world:1.0
deployment.apps/hello-world created
pi@k3sserver:~ $ sudo kubectl get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   1/1     1            1           8s
pi@k3sserver:~ $ sudo kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
hello-world-6df9f4cc87-hmh5d   1/1     Running   0          16s
```
It will take longer the first time as the image is downloaded and unpacked.

Expose the web service to the network:
```
pi@k3sserver:~ $ sudo kubectl expose deployment hello-world --type=LoadBalancer --port=8080
service/hello-world exposed
pi@k3sserver:~ $ sudo kubectl get services
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
kubernetes    ClusterIP      10.43.0.1      <none>          443/TCP          3d22h
hello-world   LoadBalancer   10.43.234.32   192.168.1.171   8080:31509/TCP   6s
```

A Kubernetes service has been created exposing our service on port 31059.

Let's access it:
```
gary@laptop:~$ curl http://k3sserver:31509
Hello World!gary@laptop:~$
```

Success!
