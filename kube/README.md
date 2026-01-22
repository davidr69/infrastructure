# KUBE #

In order to recreate the Kubernetes instance, we use the MicroK8s package
provided by Canonical. It is a snap package that is installed with a single
command.

In additional, applications need to be deployed.  Since these steps are
relatively simple, they have been encapsulated in a single Ansible playbook.

This server only runs MicroK8s and host several apps. Since I run a private
registry on a different server, *containerd* on the Kube server must be
configured to allow non-SSL connections to the private registry. In addition
to installing MicroK8s, modifying the containerd config **MUST** be the next
step since none of the apps will deploy.

A folder must be added per registry in `/var/snap/microk8s/current/args/certs.d`:

```shell
$ cd /var/snap/microk8s/current/args/certs.d
$ mkdir registry:5000
$ cp /dev/stdin hosts.toml
server = "http://registry:5000"

[host."http://registry:5000"]
  capabilities = ["pull", "resolve"]
```

Apps which need to leverage HA Proxy headers will require the nginix
configmap to be modified to include the following data:

```yaml
data:
  compute-full-forwarded-for: "true"
  proxy-real-ip-cidr: 192.168.0.0/16,10.0.0.0/8
  use-forwarded-headers: "true"
```

```shell
kubectl create serviceaccount api-admin -n default
kubectl create clusterrolebinding api-admin-binding --clusterrole=cluster-admin --serviceaccount=default:api-admin
kubectl create token api-admin
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-admin-token-secret
  namespace: default
  annotations:
    kubernetes.io/service-account.name: api-admin
type: kubernetes.io/service-account-token
```


$ kubectl apply -f api-admin-secret.yaml





$ kubectl get configmap coredns -n kube-system -o yaml

```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
          lameduck 5s
        }
        ready
        log . {
          class error
        }
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```