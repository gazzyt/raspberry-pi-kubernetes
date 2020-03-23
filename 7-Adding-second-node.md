We will now add a second node to our cluster. I will use a Raspberry PI 2 with no SSD this time.

In order for a new node to join a cluster it needs a security token. To get the token, run the following on the existing server:

```
pi@k3sserver:~ $sudo cat /var/lib/rancher/k3s/server/node-token
```

Then from the new node we run the same install script as we did for the server but add a couple of parameters that cause this node to join an existing cluster:
* K3S_TOKEN - this is the token we got from the server
* K3S_URL - the url of the K3S server. Port 6443 is the default.

```
pi@k3snode1:~ $ curl -sfL https://get.k3s.io | K3S_TOKEN=xxx K3S_URL=https://k3sserver:6443 sh -
[INFO]  Finding latest release
[INFO]  Using v1.17.3+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.3+k3s1/sha256sum-arm.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.3+k3s1/k3s-armhf
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
chcon: can't apply partial context to unlabeled file '/usr/local/bin/k3s'
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service â /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent

```

Check it worked:
```
pi@k3sserver:/var/lib/rancher $ sudo k3s kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
k3sserver   Ready    master   20d   v1.17.2+k3s1
k3snode1    Ready    <none>   41s   v1.17.3+k3s1

```
It worked!

Now let's see what pods are running:

```
pi@k3sserver:~ $ sudo k3s kubectl get pods --all-namespaces -o wide
NAMESPACE              NAME                                         READY   STATUS      RESTARTS   AGE     IP           NODE        NOMINATED NODE   READINESS GATES
kube-system            helm-install-traefik-7lkl7                   0/1     Completed   2          27d     10.42.0.2    k3sserver   <none>           <none>
default                svclb-http-server-pfxcg                      1/1     Running     1          7d21h   10.42.1.7    k3snode1    <none>           <none>
default                svclb-hello-world-tt9w5                      1/1     Running     1          7d21h   10.42.1.8    k3snode1    <none>           <none>
kube-system            svclb-traefik-ppvqq                          2/2     Running     2          7d21h   10.42.1.6    k3snode1    <none>           <none>
default                svclb-hello-world-87gsq                      1/1     Running     4          24d     10.42.0.73   k3sserver   <none>           <none>
default                hello-world-6df9f4cc87-hmh5d                 1/1     Running     4          24d     10.42.0.79   k3sserver   <none>           <none>
kube-system            svclb-traefik-5lmw9                          2/2     Running     8          27d     10.42.0.72   k3sserver   <none>           <none>
default                svclb-http-server-7bsvx                      1/1     Running     4          14d     10.42.0.80   k3sserver   <none>           <none>
kubernetes-dashboard   dashboard-metrics-scraper-7b8b58dc8b-mbcx9   1/1     Running     4          23d     10.42.0.78   k3sserver   <none>           <none>
default                web-server-657cd8448f-xkxzq                  2/2     Running     8          8d      10.42.0.76   k3sserver   <none>           <none>
kube-system            local-path-provisioner-58fb86bdfd-x65lm      1/1     Running     82         27d     10.42.0.74   k3sserver   <none>           <none>
kube-system            metrics-server-6d684c7b5-q2sxq               1/1     Running     80         27d     10.42.0.70   k3sserver   <none>           <none>
kube-system            coredns-d798c9dd-xvhk6                       1/1     Running     4          27d     10.42.0.71   k3sserver   <none>           <none>
kubernetes-dashboard   kubernetes-dashboard-866f987876-qc2pz        1/1     Running     72         23d     10.42.0.75   k3sserver   <none>           <none>
kube-system            traefik-6787cddb4b-z5vlg                     1/1     Running     4          27d     10.42.0.77   k3sserver   <none>           <none>
```

We already have 3 pods running on `k3snode1`. Running `describe` on these e.g.
```
pi@k3sserver:~ $ sudo k3s kubectl describe pod svclb-http-server-pfxcg
Name:         svclb-http-server-pfxcg
Namespace:    default
Priority:     0
Node:         k3snode1/192.168.1.186
Start Time:   Sun, 15 Mar 2020 21:57:38 +0000
Labels:       app=svclb-http-server
              controller-revision-hash=c9b6df98b
              pod-template-generation=1
              svccontroller.k3s.cattle.io/svcname=http-server
Annotations:  <none>
Status:       Running
IP:           10.42.1.7
IPs:
  IP:           10.42.1.7
Controlled By:  DaemonSet/svclb-http-server
Containers:
  lb-port-8081:
    Container ID:   containerd://d6705d3107aca5d6ee96449e9bda8f44cb00e98eea5671165565211b2c0e65ae
    Image:          rancher/klipper-lb:v0.1.2
    Image ID:       docker.io/rancher/klipper-lb@sha256:2fb97818f5d64096d635bc72501a6cb2c8b88d5d16bc031cf71b5b6460925e4a
    Port:           8081/TCP
    Host Port:      8081/TCP
    State:          Running
      Started:      Sat, 21 Mar 2020 18:35:02 +0000
    Last State:     Terminated
      Reason:       Unknown
      Exit Code:    255
      Started:      Sun, 15 Mar 2020 21:58:25 +0000
      Finished:     Sat, 21 Mar 2020 18:33:07 +0000
    Ready:          True
    Restart Count:  1
    Environment:
      SRC_PORT:    8081
      DEST_PROTO:  TCP
      DEST_PORT:   8081
      DEST_IP:     10.43.85.145
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-bz75s (ro)
...
```
We see that they all run the container image `rancher/klipper-lb:v0.1.2`. This contains a [simple script](https://github.com/rancher/klipper-lb/blob/master/entry) which configures packets received on certain ports to be forwarded to another ip address.

From the `describe` output we see that pod `svclb-http-server-pfxcg` running on `k3snode1` will cause packets received on port `8081` to be forwarded to port `8081` at IP address `10.43.85.145`. By listing the services we can see that this destination IP is the address of the load balancer we created for NGINX.

```
pi@k3sserver:~ $ sudo k3s kubectl get services --all-namespaces -o wide
NAMESPACE              NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
default                kubernetes                  ClusterIP      10.43.0.1       <none>          443/TCP                      28d   <none>
kube-system            kube-dns                    ClusterIP      10.43.0.10      <none>          53/UDP,53/TCP,9153/TCP       28d   k8s-app=kube-dns
kube-system            metrics-server              ClusterIP      10.43.7.154     <none>          443/TCP                      28d   k8s-app=metrics-server
kube-system            traefik-prometheus          ClusterIP      10.43.90.86     <none>          9100/TCP                     28d   app=traefik,release=traefik
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP      10.43.39.99     <none>          8000/TCP                     24d   k8s-app=dashboard-metrics-scraper
kubernetes-dashboard   kubernetes-dashboard        NodePort       10.43.173.211   <none>          443:31586/TCP                24d   k8s-app=kubernetes-dashboard
default                hello-world                 LoadBalancer   10.43.234.32    192.168.1.171   8080:31509/TCP               24d   app=hello-world
default                http-server                 LoadBalancer   10.43.85.145    192.168.1.171   8081:32183/TCP               15d   app=web-server
kube-system            traefik                     LoadBalancer   10.43.111.104   192.168.1.186   80:31007/TCP,443:31456/TCP   28d   app=traefik,release=traefik
```

So we have a single node running NGINX, a single load balancer instance for it but packets from any node in the cluster are forwarded to it.

Running `ifconfig` on `k3snode1` we can list all its ip addresses. Then we can use curl to make an http request to port 8081.

```
pi@k3snode1:~ $ curl http://127.0.0.1:8081/
<html>
<head>
<title>NGINX Test Page</title>
</head>
<body>This page was served by NGINX running in K3S and synced from github.
<p>Version 2</p></body>
</html>
```
So this request was sent to the our NGINX pod on `k3sserver`.

If fact almost any of the IP addresses will work the same:
```
curl http://169.254.183.226:8081/
curl http://169.254.97.117:8081/
curl http://169.254.46.145:8081/
curl http://192.168.1.186:8081/
curl http://10.42.1.1:8081/
```
