# Kubernetes Certified Administrator



## PersistenVolumeClaim

TASK:

```bash
mariadb namespace 中的 MariaDB Deployment 被误删除。请恢复该 Deployment 并确保数据持久性。请按照以下步骤：
如下规格在 mariadb namespace 中创建名为 mariadb 的 PersistentVolumeClaim(PVC)：

访问模式为 ReadWriteOnce

存储为 250Mi

集群中现有一个 PersistentVolume。
您必须使用现有的 PersistentVolume (PV)。
编辑位于 ~/mariadb-deployment.yaml 的 MariaDB Deployment 文件，以使用上一步中创建的 PVC。
将更新的 Deployment 文件应用到集群。
确保 MariaDB Deployment 正在运行且稳定。
```

RESLOVE: 

```bash
# 查看 mariadb namespace 下的 PersistentVolume
root[~]: kubectl get pv -n mariadb
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
mariadb-pv   250Mi      RWO            Retain           Available           local-path     <unset>                          151d
```

创建 pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb
  namespace: mariadb
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: local-path     # 存储类
  resources:
    requests:
      storage: 250Mi
```

```yaml
root[home]: kubectl apply -f pvc.yaml 
persistentvolumeclaim/mariadb created
root[home]: kubectl get pvc -n mariadb
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mariadb   Pending                                      local-path     <unset>                 13s
```

编辑 deployment

```yaml
# ~/mariadb-deployment.yaml 使 Deployment 创建的 Container 挂载 PVC
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  namespace: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:10.5
        imagePullPolicy: IfNotPresent  
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
        volumeMounts:
        - name: mariadb-data          # 使用 mariadb-data 存储卷
          mountPath: /var/lib/mysql   # 挂载到
      volumes:
      - name: mariadb-data            # 将使用的 pvc 命名为 mariadb-data
        persistentVolumeClaim:
          claimName: "mariadb"        # 使用名为 mariadb 的 PersistentVolumeClaim
```

更新控制器

```bash
candidate@base:~$ kubectl apply -f ~/mariadb-deployment.yaml
deployment.apps/mariadb created
```

检查状态

```bash
candidate@base:~$ kubectl get deployment -n mariadb
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
mariadb   1/1     1            1           15s

candidate@base:~$ kubectl -n mariadb get pods
NAME                       READY   STATUS    RESTARTS   AGE
mariadb-5c7c489db6-zd5mj   1/1     Running   0          34s
```

Complete ...



## Service

TASK:

```bash
重新配置 spline namespace 中现有的 front-end Deployment，以公开现有容器 nginx 的端口 80/tcp
创建一个名为 front-end-svc 的新 Service ，以公开容器端口 80/tcp
配置新的 Service ，以通过 NodePort 公开各个 Pod
```

RESLOVE:

查看信息

```bash
candidate@base:~$ kubectl get ns | grep spline
spline               Active   151d
candidate@base:~$ kubectl get deploy -n spline
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
front-end   1/1     1            1           151d
```

编辑控制器

```bash
kubectl edit deployment front-end -n spline
```

```yaml
    spec:
      containers:
      - image: nginx:1.25
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        ports:                 # 
        - containerPort: 80    #
          protocol: tcp        #
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

创建 service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: front-end-svc
  namespace: spline
spec:
  type: NodePort
  selector:
    app: front-end          # Deployment 的 Pod 标签
  ports:
    - name: http
      port: 80              # Service 的端口（ClusterIP:80）
      targetPort: 80        # Pod 容器监听的端口
      protocol: TCP
```

检查

```bash
candidate@base:home$ kubectl apply -f svc.yaml 
service/front-end-svc created
candidate@base:home$ kubectl get svc -n spline -o wide
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
front-end-svc   NodePort   10.102.111.122   <none>        80:30118/TCP   16s   app=front-end
candidate@base:home$ curl 10.102.111.122:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Complete ...



## Ingress

TASK:

```bash
创建新的 Ingress 资源，具体如下：

名称： echo-hello
Namespace： sound

使用 Service 端口 8080 在 http://example.org/echo-hello 上公开 echoserver-service 的 Service。
可以使用以下命令检查 echoserver-service Service 的可用性，该命令应返回 Hello World：

[candidate@cka000003~]$ curl http://example.org/echo-hello
```

RESLOVE:

查询信息

```bash
candidate@base:home$ kubectl get ingressclasses.networking.k8s.io
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       151d
```

编写 ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-hello
  namespace: sound
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: example.org
    http:
      paths:
      - path: /echo-hello
        pathType: Prefix
        backend:
          service:
            name: echoserver-server
            port:
              number: 8080
```

验证

```bash
candidate@base:home$ kubectl apply -f ingress.yaml 
ingress.networking.k8s.io/echo-hello created
candidate@base:home$ kubectl get ingress -n sound
NAME         CLASS   HOSTS         ADDRESS   PORTS   AGE
echo-hello   nginx   example.org             80      13s
candidate@base:home$ kubectl get ingress -n sound
NAME         CLASS   HOSTS         ADDRESS         PORTS   AGE
echo-hello   nginx   example.org   192.168.40.11   80      45s

candidate@base:home$ curl http://example.org/echo-hello
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
```



## NetworkPolicy

TASK:

```bash
从提供的 YAML 文件中选择并应用适当的 NetworkPolicy。确保所选的 NetworkPolicy 不会过于宽松，同时允许运行在 frontend 和 backend namespaces 中的 frontend 和 backend Deployment 之间的通信。

首先，分析 frontend 和 backend Deployment，了解其通信需求，以便确定需要应用的NetworkPolicy。
接着，检查位于 ~/netpol 文件夹中的 NetworkPolicy YAML 示例。

请注意：不要删除或修改提供的示例，只需应用其中的一个，否则可能会影响得分。
最后，应用一个 NetworkPolicy，以启用 frontend 和 backend Deployment 之间的通信，但不要使其过于宽松。
注意：请勿删除或修改现有的默认拒绝所有入站流量或出口流量 NetworkPolicy。否则可能导致零分
```



查询信息

```bash
# namespace labels
candidate@base:home$ kubectl get ns frontend backend --show-labels
NAME       STATUS   AGE    LABELS
frontend   Active   151d   kubernetes.io/metadata.name=frontend
backend    Active   151d   kubernetes.io/metadata.name=backend
# frontend pods labels
candidate@base:home$ kubectl get pod -n frontend --show-labels
NAME                            READY   STATUS    RESTARTS       AGE    LABELS
frontend-app-5d99f5b946-hhcp7   1/1     Running   7 (3d3h ago)   151d   app=frontend,pod-template-hash=5d99f5b946
# backend pods labels
candidate@base:home$ kubectl get pod -n backend --show-labels
NAME                          READY   STATUS    RESTARTS       AGE    LABELS
backend-app-978469478-dpmmb   1/1     Running   7 (3d2h ago)   151d   app=backend,pod-template-hash=978469478
# 检查默认拒绝策略
candidate@base:home$ kubectl -n backend get networkpolicies
NAME               POD-SELECTOR   AGE
default-deny-all   <none>         151d
```

分析文件夹中网络策略

```yaml
# netpol1.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-1
  namespace: backend
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: frontend

# 允许 frontend namespace 下所有 Pod 访问 backend namespace 下所有 Pod
```

```yaml
# netpol2.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-2
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: frontend
      podSelector:
        matchLabels:
          app: frontend
          
# 允许 frontend namespace 下有 app=frontend 标签的 Pod 访问 backend namespace 下有 app=backend 标签的 Pod ，这个策略更精细。
```

```yaml
# netpol3.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-3
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: test
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: frontend
      podSelector:
        matchLabels:
          app: frontend
    - ipBlock:
        cidr: 10.244.0.0/24
        
# 策略涉及 app=test 标签的 Pod ，与题目不符。
```

```bash
# 最终选择第二个策略
# 配置第二个策略使其生效
candidate@base:~/netpol$ kubectl apply -f netpol2.yaml 
networkpolicy.networking.k8s.io/netpol-2 created
# 检查最终生效情况
candidate@base:~/netpol$ kubectl -n backend get networkpolicies
NAME               POD-SELECTOR   AGE
default-deny-all   <none>         187d
netpol-2           app=backend    32s
```



## Configmap

TASK:

```bash
名为 nginx-static-test 的 NGINX Deployment 正在 nginx-static-test namespace 中运行。它通过名为 nginx-static-config 的 ConfigMap 进行配置。

更新 nginx-static-config 这个 ConfigMap 以仅允许 TLSv1.3 连接。
注意：您可以根据需要重新创建、重新启动或扩展资源。
您可以使用以下命令测试更改:

candidate@cka000005$ curl -k --tls-max 1.2 https://web.k8snginx.local
```

RESLOVE:

```bash
# 导出现有的 nginx-static-config ConfigMap
candidate@base:~$ kubectl -n nginx-static-test get configmap nginx-static-config -o yaml > nginx-static-config.yaml
```

```yaml
# nginx-static-config.yaml
apiVersion: v1
data:
  nginx.conf: |
    worker_processes  1;

    events {
        worker_connections  1024;
    }

    http {
        # SSL 配置加在这里，http 块内，server 块外
        ssl_protocols TLSv1.3; # 将1.2改为1.3即可
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256';
        ssl_prefer_server_ciphers on;

        server {
            listen 80;
            server_name web.k8snginx.local;

            location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;
            }
        }
    }
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
```

```bash
# 删除旧有 ConfigMap 创建新的 ConfigMap
candidate@base:~$ kubectl -n nginx-static-test delete configmap nginx-static-config
configmap "nginx-static-config" deleted
candidate@base:~$ kubectl -n nginx-static-test apply -f nginx-static-config.yaml
configmap/nginx-static-config created
# 重启 nginx-static-test Deployment
candidate@base:~$ kubectl -n nginx-static-test rollout restart deployment nginx-static-test
deployment.apps/nginx-static-test restarted
# 检查新 Pod 状态
candidate@base:~$ kubectl -n nginx-static-test get pods
NAME                                 READY   STATUS    RESTARTS   AGE
nginx-static-test-7965994f9f-4f2dz   1/1     Running   0          53s
# 发现是 Running 状态
```



## StorageClass

TASK:

```bash
首先，为名为 rancher.io/local-path 的现有制备器，创建一个名为 test-local-path 的新 StorageClass

将卷绑定模式设置为 WaitForFirstConsumer

接下来，将 test-local-path StorageClass 配置为默认的 StorageClass
```

RESLOVE:

```yaml
# sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: test-local-path
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
```

```bash
candidate@base:~$ kubectl apply -f sc.yaml 
storageclass.storage.k8s.io/test-local-path created
candidate@base:~$ kubectl get storageclass
NAME                        PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path                  rancher.io/local-path   Delete          WaitForFirstConsumer   false                  188d
test-local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  31s
```



## HPA

TASK:

```bash
在 autoscale namespace 中创建一个名为 nginx-server 的新HorizontalPodAutoscaler(HPA)。此 HPA 必须定位到 autoscale namespace 中名为 nginx-server 的现有 Deployment 。
将 HPA 设置为每个 Pod 的 CPU 使用率旨在 50% 。将其配置为至少有 1 个 Pod，且不超过 4 个 Pod 。此外，将缩小稳定窗口设置为 30 秒。
```

RESLOVE:

```bash
candidate@base:~$ kubectl autoscale deployment nginx-server --cpu-percent=50 --min=1 --max=4 -n autoscale
horizontalpodautoscaler.autoscaling/nginx-server autoscaled
# 修改 yaml 文件，如下
candidate@base:~$ kubectl -n autoscale edit horizontalpodautoscaler nginx-server
horizontalpodautoscaler.autoscaling/nginx-server edited
```

附录：修改 yaml 文件

```yaml
# nginx-server.yaml
spec:
  maxReplicas: 4
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 30
  metrics:
```



## PriorityClass

TASK:

```bash
为用户工作负载创建一个名为 high-priority 的新 PriorityClass ，其值比用户定义的现有最高优先级类值小一。修改在 priority namespace 中运行的现有 busybox-logger Deployment ，以使用 high-priority 优先级类。
确保 busybox-logger Deployment 在设置了新优先级类后成功部署。
```

RESLOVE:

```bash
# 查看集群中存在的 PriorityClass
candidate@base:home$ kubectl get priorityclass
NAME                      VALUE        GLOBAL-DEFAULT   AGE    PREEMPTIONPOLICY
max-priority              1000000000   false            201d   PreemptLowerPriority
system-cluster-critical   2000000000   false            201d   PreemptLowerPriority
system-node-critical      2000001000   false            201d   PreemptLowerPriority
# 创建新的 PriorityClass
```

```yaml
# priority.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 999999999       # 按照题目要求，值比用户定义最大值小1
globalDefault: false
```

```bash
# 应用新的 PriorityClass
candidate@base:home$ kubectl apply -f priority.yaml
priorityclass.scheduling.k8s.io/high-priority created
# 检查
candidate@base:home$ kubectl get priorityclass
NAME                      VALUE        GLOBAL-DEFAULT   AGE    PREEMPTIONPOLICY
high-priority             999999999    false            43s    PreemptLowerPriority
max-priority              1000000000   false            201d   PreemptLowerPriority
system-cluster-critical   2000000000   false            201d   PreemptLowerPriority
system-node-critical      2000001000   false            201d   PreemptLowerPriority
# 修改 Deployment 中的 PriorityClassName 为新创建的 PriorityClass
```

```bash
kubectl -n priority edit deployment busybox-logger
```

```yaml
...
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: busybox-logger
    spec:
      priorityClassName: high-priority  # 在 spec.template.spec 中指定
      containers:
      - command:
        - /bin/sh
        - -c
        - while true; do echo 'Logging...'; sleep 5; done
        image: busybox:1.28
...
```

```bash
deployment.apps/busybox-logger edited
```



## Resources

CONTEXT:

```bash
您管理一个 WordPress 应用程序。由于资源的请求过高，导致某些 Pod 无法启动。
```

TASK:

```bash
relative-fawn namespace 中的 WordPress 应用程序包含：
具有 3 个副本的 WordPress Deployment

按照如下方式调整所有 Pod 资源请求：

将节点资源平均分配给这 3 个 Pod
为每个 Pod 分配公平的 CPU 和内存份额
添加足够的开销以保持节点稳定

请确保，对容器和初始化容器使用完全相同的请求。您无需更改任何资源限制。

在更新资源请求时，可以暂时将 WordPress Deployment 缩放为 0 个副本可能会有所帮助。
更新后，请确认：
WordPress 保持 3 个副本

所有 Pod 都在运行并准备就绪
```

RESLOVE:

```bash
# 查看信息并将将副本缩放为 0
candidate@base:/home$ kubectl -n relative-fawn get deployment wordpress
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   3/3     3            3           201d
candidate@base:/home$ kubectl -n relative-fawn scale deployment wordpress --replicas=0
deployment.apps/wordpress scaled
candidate@base:/home$ kubectl -n relative-fawn get deployment wordpress
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   0/0     0            0           201d
# 修改 Deployment 资源
candidate@base:/home$ kubectl -n relative-fawn edit deployment wordpress
```

```yaml
...
        - containerPort: 80
          protocol: TCP
        resources:
          requests:           # 设置 requests 字段至合适
            cpu: 100m
            memory: 200Mi
          limits:
            cpu: "1"
            memory: 500Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      initContainers:
      - command:
        - sh
        - -c
        - sleep 6
        image: busybox:1.28
        imagePullPolicy: IfNotPresent
        name: init-mysql
        resources:
          requests:       # 另一个 container 同样
            cpu: 100m
            memory: 200Mi
          limits:
            cpu: "1"
            memory: 500Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File

...
```

```bash
deployment.apps/wordpress edited
```

```bash
# 恢复 Pod 副本数
candidate@base:/home$ kubectl -n relative-fawn scale deployment wordpress --replicas=3
deployment.apps/wordpress scaled
# 检查
candidate@base:/home$ kubectl -n relative-fawn get deployment wordpress
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   3/3     3            3           201d
```



## CRD

TASK:

```bash
验证已经部署到集群的 cert-manager 应用程序。
使用 kubectl 将 cert-manager 名称空间所有自定义的资源（CRD）的列表，保存到 
~/resources.yaml 。

注意：您必须使用 kubectl 的默认输出格式。请勿设置输出格式。否则将导致分数降低或者不得分。

使用 kubectl ，提取定制资源 Certificate 的 subject 规范字段的文档，并将其保存到 
~/subject.yaml 。
```

RESLOVE:

```bash
# 获得 cert-manager namespace 信息
candidate@base:/home$ kubectl -n cert-manager get pods
NAME                                       READY   STATUS    RESTARTS       AGE
cert-manager-7b5cdf866f-9ndzp              1/1     Running   20 (12d ago)   201d
cert-manager-cainjector-7c9788477c-4dttx   1/1     Running   22 (14d ago)   201d
cert-manager-webhook-764949f558-nj8pb      1/1     Running   8 (14d ago)    201d
# 获得 cert-manager namespace 的 CRD 资源列表
candidate@base:/home$ kubectl get crd | grep cert-manager --color
certificaterequests.cert-manager.io                   2025-03-23T02:24:12Z
certificates.cert-manager.io                          2025-03-23T02:24:12Z
challenges.acme.cert-manager.io                       2025-03-23T02:24:12Z
clusterissuers.cert-manager.io                        2025-03-23T02:24:12Z
issuers.cert-manager.io                               2025-03-23T02:24:12Z
orders.acme.cert-manager.io                           2025-03-23T02:24:12Z
# 将 CRD 资源列表导出至 yaml 文件
candidate@base:/home$ kubectl get crd | grep cert-manager > resources.yaml
# 获取 Certificate 的 subject 规范字段添加到 subject.yaml
candidate@base:/home$ kubectl explain certificate.spec.subject > subject.yaml
```



## Gateway

TASK:

```bash
将现有 Web 应用程序从 Ingress 迁移到 Gateway API。您必须维护 HTTPS 访问权限。
注意：集群中安装了一个名为 nginx 的 GatewayClass 。

首先，创建一个名为 web-local-gateway 的 Gateway ，主机名为 gateway.web.k8s.local ，
并保持现有名为 web 的 Ingress 资源的现有 TLS 和侦听器配置。
接下来，创建一个名为 web-route 的 HTTPRoute ，主机名为 gateway.web.k8s.local ，并保持现有名为 web 的 Ingress 资源的现有路由规则。

您可以使用以下命令测试 Gateway API 配置：
[candidate@cka000011]$ curl -Lk https://gateway.web.k8s.local:31443

最后，删除名为 web 的现有 Ingress 资源。
```

RESLOVE:

```bash
# 查看 ingress
candidate@base:/home$ kubectl get ingress
NAME   CLASS   HOSTS                   ADDRESS         PORTS     AGE
web    nginx   gateway.web.k8s.local   192.168.40.11   80, 443   201d
# 获取 yaml 信息
candidate@base:/home$ kubectl get ingress web -o yaml
```

```yaml
# web.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.k8s.io/v1","kind":"Ingress","metadata":{"annotations":{"nginx.ingress.kubernetes.io/rewrite-target":"/"},"name":"web","namespace":"default"},"spec":{"ingressClassName":"nginx","rules":[{"host":"gateway.web.k8s.local","http":{"paths":[{"backend":{"service":{"name":"web","port":{"number":80}}},"path":"/","pathType":"Prefix"}]}}],"tls":[{"hosts":["gateway.web.k8s.local"],"secretName":"web-cert"}]}}
    nginx.ingress.kubernetes.io/rewrite-target: /
  creationTimestamp: "2025-03-23T02:51:07Z"
  generation: 2
  name: web
  namespace: default
  resourceVersion: "267389"
  uid: 586b4aca-e47d-4002-9ef3-30242ed98e37
spec:
  ingressClassName: nginx
  rules:
  - host: gateway.web.k8s.local
    http:
      paths:
      - backend:
          service:
            name: web
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - gateway.web.k8s.local
    secretName: web-cert
status:
  loadBalancer:
    ingress:
    - ip: 192.168.40.11
```

```yaml
# 创建 gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-local-gateway
spec:
  gatewayClassName: nginx    # 题目给出
  listeners:
   - name: https
     protocol: HTTPS         # 协议，按照题目要求
     port: 443
     hostname: gateway.web.k8s.local  # 主机名，按照题目要求
     tls:
       mode: Terminate
       certificateRefs:
       - name: web-cert      # 填写 ingress 中的 tls secretname
```

```bash
# 应用
candidate@base:/home$ kubectl apply -f gateway.yaml 
gateway.gateway.networking.k8s.io/web-local-gateway created
# 检查
candidate@base:~$ kubectl get gateway
NAME                CLASS   ADDRESS   PROGRAMMED   AGE
web-local-gateway   nginx             Unknown      23h
# 创建 httproute
```

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parenRefs:
  - name: web-local-gateway    # 上面创建的 gateway 名称
  hostnames:
  - "gateway.web.k8s.local"    # 按照题目要求写
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /               # ingress 中 path 路径
    backendRefs:
    - name: web                # ingress 中 service name
      port: 80                 # ingress 中 serivce port
```

```bash
# 应用 httproute
candidate@base:home$ kubectl apply -f httproute.yaml 
httproute.gateway.networking.k8s.io/web-route created
# 检查
candidate@base:home$ kubectl get httproute
NAME        HOSTNAMES                   AGE
web-route   ["gateway.web.k8s.local"]   45s
```

```bash
# 测试 gateway api 配置
candidate@base:home$ curl -Lk https://gateway.web.k8s.local:31443
# 删除 web ingress
candidate@base:home$ kubectl delete ingress web
ingress.networking.k8s.io "web" deleted
```



## sidecar

CONTEXT:

```bash
为了将传统应用程序集成到 Kubernetes 的日志架构（如 kubectl logs）中，通常的做法是添加一个用于日志流式传输的 sidecar 容器。
```

TASK:

```bash
更新现有的 sync-leverager Deployment，将使用 busybox:stable 镜像，且名为 sidecar 的并置容器，添加到现有的 Pod 。新的并置容器必须运行以下命令：
/bin/sh -c "tail -n+1 -f /var/log/sync-leverager.log"
使用挂载在 /var/log 的 Volume，使日志文件 sync-leverager.log 可供并置容器使用。
除了添加所需的卷挂载之外，请勿修改现有容器的规范。
```

RESLOVE:

```bash
# 导出 sync-leverager yaml 文件
candidate@base:home$ kubectl get deployment sync-leverager -o yaml > sidecar.yaml
# 编辑 sidecar.yaml
```

```yaml
# sidecar.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"sync-leverager"},"name":"sync-leverager","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"sync-leverager"}},"template":{"metadata":{"labels":{"app":"sync-leverager"}},"spec":{"containers":[{"command":["/bin/sh","-c","while true; do echo \"$(date) INFO log line\" \u003e\u003e /var/log/sync-leverager.log; sleep 5; done"],"image":"nginx:1.25","imagePullPolicy":"IfNotPresent","name":"sync-leverager"}]}}}}
  creationTimestamp: "2025-03-23T05:17:30Z"
  generation: 1
  labels:
    app: sync-leverager
  name: sync-leverager
  namespace: default
  resourceVersion: "267172"
  uid: 64bc4ebd-2d07-4094-ab6a-a9c560622d85
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: sync-leverager
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: sync-leverager
    spec:
      volumes:        # 新增
      - name: varlog
        emptyDir: {}  # ---
      containers:    
      - command:
        - /bin/sh
        - -c
        - while true; do echo "$(date) INFO log line" >> /var/log/sync-leverager.log;
          sleep 5; done
        image: nginx:1.25
        imagePullPolicy: IfNotPresent
        name: sync-leverager
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:             # 新增
        - name: varlog
          mountPath: /var/log
      - name: sidecar
        image: busybox:stable
        imagePullPolicy: IfNotPresent
        args: ["/bin/sh","-c", "tail -n+1 -f /var/log/sync-leverager.log"]
        volumeMounts:
        - name: varlog
          mountPath: /var/log     # ---
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2025-03-23T05:17:31Z"
    lastUpdateTime: "2025-03-23T05:17:32Z"
    message: ReplicaSet "sync-leverager-984fc4f56" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2025-09-26T06:14:17Z"
    lastUpdateTime: "2025-09-26T06:14:17Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```

```bash
# 应用
candidate@base:home$ kubectl apply -f sidecar.yaml
deployment.apps/sync-leverager configured
# 检查
candidate@base:home$ kubectl get deployment sync-leverager
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
sync-leverager   1/1     1            1           202d
candidate@base:home$ kubectl get pod | grep sync-leverager
sync-leverager-5cc597cf7-zn45m   2/2     Running   0             71s
```



## calico

CONTEXT:

```bash
集群的 CNI 未通过安全审核，已被移除。您必须安装一个可以实施网络策略的新 CNI。
```

TASK:

```bash
安装并设置满足以下要求的容器网络接口（CNI）：

选择并安装以下 CNI 选项之一：
1）Flannel 版 本 0.26.1
2）Calico 版 本 3.27.0

选择的 CNI 必须：
1）让 Pod 相互通信
2）支 持 Network Policy 实 施
3）从清单文件安装（请勿使用 Helm）
```

RESLOVE:

```bash
# 在模拟环境做这个题可能会导致集群出问题，所以先创建快照做完后恢复
```

```bash
# 下载 Calico tigera-operator.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
# 部署 tigera-operator
kubectl create -f tigera-operator.yaml
# 查看 Pod CIDR
kubectl cluster-info dump | grep -i cluster-cidr
# 假设输出 "clusterCIDR": "10.244.0.0/16"
# 下载 Calico 自定义资源配置 custom-resources.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml
# 备注：这个网址考试没给，可以在 tigera-operator.yaml 这个下载地址基础上记个单词
# custom-resources.yaml 即可。
# 编辑 custom-resources.yaml 文件，修改 cidr 字段
vim custom-resources.yaml
```

```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
name: default
spec:
calicoNetwork:
ipPools:
- blockSize: 26
cidr: 10.244.0.0/16      # 按上步骤实际查询的 pod 网段写
encapsulation: VXLANCrossSubnet
natOutgoing: Enabled
nodeSelector: all()
```

```bash
# 创建 Calico 自定义资源
kubectl create -f custom-resources.yaml
# 检查运行状态
kubectl -n calico-system get pod
```





## argocd

TASK:

```bash
通过执行以下任务在集群中安装 Argo CD：

添加名为 argo 的官方 Argo CD Helm 存储库。
注意：Argo CD CRD 已在集群中预安装。
为 argocd namespace 生成 Argo CD Helm 图表版本 5.5.22 的模板，并将其保存到
~/argo-helm.yaml ，将图表配置为不安装 CRDs 。

使用 Helm 安装 Argo CD ，并设置发布名称为 argocd ，使用与模板中相同的配置和版本（5.5.22） ，将其安装在 argocd namespace 中，并配置为不安装 CRDs 。
注意：您不需要配置对 Argo CD 服务器 UI 的访问权限。
```

RESLOVE:

```bash
# 添加官方 Argo CD Helm 存储库
candidate@base:~$ helm repo add argo https://argoproj.github.io/argo-helm
"argo" already exists with the same configuration, skipping
candidate@base:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "argo" chart repository
Update Complete. ⎈Happy Helming!⎈
# 搜索 Argo CD Chart 验证是否添加成功
candidate@base:~$ helm search repo argo | grep argo-cd
argo/argo-cd              	8.6.0        	v3.1.8       	A Helm chart for Argo CD, a declarative, GitOps...
# 生成 Argo CD Helm 模版 版本号必须是题目要求的
candidate@base:~$ helm template argocd argo/argo-cd --version 5.5.22 --namespace argocd --set crds.install=false > ~/argo-helm.yaml
# 使用 Helm 安装 Argo CD
candidate@base:~$ helm install argocd argo/argo-cd --version 5.5.22 --namespace --set crds.install=false
# 验证 Argo CD 安装
candidate@base:~$ kubectl -n argocd get pods
NAME                                                READY   STATUS             RESTARTS     AGE
argocd-application-controller-0                     0/1     Running            0            12s
argocd-applicationset-controller-69f65f94d4-6hqwg   1/1     Running            0            12s
argocd-dex-server-8496b55dcc-xhv49                  1/1     Running            0            12s
argocd-notifications-controller-6f68d54df5-lgndx    1/1     Running            0            12s
argocd-redis-5df9769596-rhp6j                       1/1     Running            0            12s
argocd-repo-server-5d5b6c4466-dh77z                 0/1     Running            0            12s
argocd-server-98cd79c7c-krqjg                       0/1     CrashLoopBackOff   1 (8s ago)   12s
```



## etcd

CONTEXT:

```bash
kubeadm 配置的集群已迁移到新机器。它需要更改配置才能成功运行。
```

TASK:

```bash
修复在机器迁移过程中损坏的单节点集群。

首先，确定损坏的集群组件，并调查导致其损坏的原因。
注意：已停用的集群使用外部 etcd 服务器。

接下来，修复所有损坏的集群组件的配置。
注意：确保重新启动所有必要的服务和组件，以使更改生效。否则可能导致分数降低。

最后，确保集群运行正常。确保：每个节点 和 所有 Pod 都处于 Ready 状态。
```

RESLOVE:

```bash
# 修复 kube-apiserver 配置
candidate@base:~$ sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
...
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379   # 更改为此
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
...
```

```bash
# 重启 kubelet
candidate@base:~$ systemctl daemon-reload
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Multiple identities can be used for authentication:
 1.  linux,,, (linux)
 2.  candidate
Choose identity to authenticate as (1-2): 2
Password: 
==== AUTHENTICATION COMPLETE ===
candidate@base:~$ systemctl restart kubelet
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to restart 'kubelet.service'.
Multiple identities can be used for authentication:
 1.  linux,,, (linux)
 2.  candidate
Choose identity to authenticate as (1-2): 2
Password: 
==== AUTHENTICATION COMPLETE ===
# 修复 kube-scheduler-master01 配置
candidate@base:~$ sudo vim /etc/kubernetes/manifests/kube-scheduler.yaml
```

```yaml
...
    resources:
      requests:
        cpu: 100m   # 更改为 100m
...
```

```bash
# 等待重启
# 再次检查集群状态
```



## cri-dockerd

CONTEXT:

```bash
您的任务是为 Kubernetes 准备一个 Linux 系统。 Docker 已被安装，但您需要为 kubeadm 配置它。
```

TASK:

```bash
完成以下任务，为 Kubernetes 准备系统：

设 置 cri-dockerd ：
1）安装 Debian 软件包 ~/cri-dockerd_0.3.6.3-0.ubuntu-jammy_amd64.deb Debian 软件包
使用 dpkg 安装。

2）启用并启动 cri-docker 服务
配置以下系统参数:

net.bridge.bridge-nf-call-iptables 设 置 为 1
net.ipv6.conf.all.forwarding 设 置 为 1
net.ipv4.ip_forward 设 置 为 1
net.netfilter.nf_conntrack_max 设置为 131072

确保这些系统参数在系统重启后仍然存在，并应用于正在运行的系统。
```

RESLOVE:

```bash
# 安装 cir-dockerd
candidate@base:~$ sudo dpkg -i ~/cri-dockerd_0.3.6.3-0.ubuntu-jammy_amd64.deb
Selecting previously unselected package cri-dockerd.
(Reading database ... 249916 files and directories currently installed.)
Preparing to unpack .../cri-dockerd_0.3.6.3-0.ubuntu-jammy_amd64.deb ...
Unpacking cri-dockerd (0.3.6~3-0~ubuntu-jammy) ...
Setting up cri-dockerd (0.3.6~3-0~ubuntu-jammy) ...
# 启动并启用 cri-docker
candidate@base:~$ sudo systemctl enable cri-docker
candidate@base:~$ sudo systemctl start cri-docker
# 配置系统参数
candidate@base:~$ sudo modprobe br-netfilter
candidate@base:~$ sudo vim /etc/sysctl.conf
# 在末行追加
```

```bash
net.bridge.bridge-nf-call-iptables = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.ip_forward = 1
net.netfilter.nf_conntrack_max = 131072
```

```bash
# 使配置立刻生效
candidate@base:~$ sudo sysctl -p
net.bridge.bridge-nf-call-iptables = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.ip_forward = 1
net.netfilter.nf_conntrack_max = 131072
```



