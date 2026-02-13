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

## web server pod

Obtain image:

```shell
$ podman pull docker.io/nginx:1.29.5
$ podman tag docker.io/library/nginx:1.29.5 registry:5000/nginx:1.29.5
$ podman push registry:5000/nginx:1.29.5
```

Web server config:

```text
events {}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    tcp_nopush      on;
    keepalive_timeout  65;

    server {
        listen 80;
        server_name _;

        root /var/www/lavacro;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location /~david/ {
            alias /home/david/public_html/;
            autoindex off;
        }

        access_log /dev/stdout;
        error_log  /dev/stderr warn;

        add_header X-Content-Type-Options nosniff always;
        add_header X-Frame-Options SAMEORIGIN always;
    }
}
```

### PV's

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lavacro-www-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadOnlyMany
  nfs:
    server: nube.lavacro.net
    path: /var/lavacro/www
  persistentVolumeReclaimPolicy: Retain
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: david-public-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadOnlyMany
  nfs:
    server: nube.lavacro.net
    path: /home/david/public_html
  persistentVolumeReclaimPolicy: Retain
```

### PVC's

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: www-content
  namespace: lavacro
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  volumeName: lavacro-www-pv
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: david-public-html
  namespace: lavacro
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  volumeName: david-public-pv
```

### image/deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-lavacro
  namespace: lavacro-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-lavacro
  template:
    metadata:
      labels:
        app: nginx-lavacro
    spec:
      containers:
      - name: web-server
        image: registry:5000/nginx:1.29.5
        volumeMounts:
        - name: www
          mountPath: /var/www/lavacro
          readOnly: true
        - name: public-html
          mountPath: /home/david/public_html
          readOnly: true
      volumes:
      - name: www
        persistentVolumeClaim:
          claimName: www-content
      - name: public-html
        persistentVolumeClaim:
          claimName: david-public-html
```
