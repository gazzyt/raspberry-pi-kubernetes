## Installing Kubernetes
Rancher have a lightweight Kubernetes distribution called [K3S](https://k3s.io/) which supports Raspberry PI.
They provide a [Quick-Start Guide](https://rancher.com/docs/k3s/latest/en/quick-start/) which I used to create a default single node cluster.
Installation is simplicity itself:
```
curl -sfL https://get.k3s.io | sh -
```

Let's see if it worked:
```
pi@k3sserver:~ $ kubectl get deployments
WARN[2020-02-27T20:25:26.151326295Z] Unable to read /etc/rancher/k3s/k3s.yaml, please start server with --write-kubeconfig-mode to modify kube config permissions
error: error loading config file "/etc/rancher/k3s/k3s.yaml": open /etc/rancher/k3s/k3s.yaml: permission denied
```

```
pi@k3sserver:~ $ sudo kubectl get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
```

So we need `sudo` for these kubectl commands.