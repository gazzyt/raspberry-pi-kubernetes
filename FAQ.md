## 1 CrashLoopBackOff after rebootingK3S server

```
pi@k3sserver:~ $ sudo k3s kubectl get pods --all-namespaces -o wide
NAMESPACE              NAME                                         READY   STATUS             RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
kube-system            helm-install-traefik-7lkl7                   0/1     Completed          2          34d   10.42.0.2     k3sserver   <none>           <none>
kube-system            traefik-6787cddb4b-z5vlg                     1/1     Running            6          34d   10.42.0.99    k3sserver   <none>           <none>
default                svclb-http-server-7bsvx                      1/1     Running            6          21d   10.42.0.101   k3sserver   <none>           <none>
default                hello-world-6df9f4cc87-hmh5d                 1/1     Running            6          31d   10.42.0.94    k3sserver   <none>           <none>
kubernetes-dashboard   dashboard-metrics-scraper-7b8b58dc8b-mbcx9   1/1     Running            6          30d   10.42.0.98    k3sserver   <none>           <none>
kube-system            svclb-traefik-ppvqq                          2/2     Running            6          14d   10.42.1.16    k3snode1    <none>           <none>
kube-system            svclb-traefik-5lmw9                          2/2     Running            12         34d   10.42.0.96    k3sserver   <none>           <none>
default                svclb-hello-world-87gsq                      1/1     Running            6          31d   10.42.0.92    k3sserver   <none>           <none>
kube-system            coredns-d798c9dd-xvhk6                       0/1     Running            6          34d   10.42.0.97    k3sserver   <none>           <none>
default                web-server-657cd8448f-xkxzq                  2/2     Running            12         15d   10.42.0.100   k3sserver   <none>           <none>
default                svclb-http-server-pfxcg                      1/1     Running            3          14d   10.42.1.15    k3snode1    <none>           <none>
default                svclb-hello-world-tt9w5                      1/1     Running            3          14d   10.42.1.14    k3snode1    <none>           <none>
kube-system            local-path-provisioner-58fb86bdfd-x65lm      0/1     CrashLoopBackOff   88         34d   10.42.0.93    k3sserver   <none>           <none>
kubernetes-dashboard   kubernetes-dashboard-866f987876-qc2pz        0/1     CrashLoopBackOff   78         30d   10.42.0.102   k3sserver   <none>           <none>
kube-system            metrics-server-6d684c7b5-q2sxq               0/1     CrashLoopBackOff   86         34d   10.42.0.95    k3sserver   <none>           <none>
```

Always affects local path provisioner and metrics server and sometimes kubernetes dashboard.

To fix, restarts the k3s daemon:
```
pi@k3sserver:~ $ sudo systemctl stop k3s
pi@k3sserver:~ $ sudo systemctl start k3s
```

