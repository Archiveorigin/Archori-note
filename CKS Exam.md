# Kubernetes Certificate Security Specialist



## Kube-bench

Context

```bash
针对kubeadm创建的cluster运行CIS基准测试工具时，发现了多个必须解决的问题。
```

Task

```bash
通过配置修复所有问题并重新启动受影响的组件以确保新设置生效。
修复针对kubelet发现的所有以下违规行为:

Ensure that the anonymous-auth argument is set to false
Ensure that the -authorization-mode argument is not set to AlwaysAllow

尽可能使用Webhook 身份验证/授权
修复针对etcd发现的以下违规行为:

Ensure that the --client-cert-auth argument is set to true
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “kubelet配置文件”
# 准确网址:
https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubelet-config-file/
```

```bash
sudo -i # 提权
vim /var/lib/kubelet/config.yaml
```

原始内容

```yaml
# /var/lib/kubelet/config.yaml

apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false   # 更改为 false
  webhook:
    cacheTTL: 0s
    enabled: false   # 因为下方 authorization 变更 这里更改为 true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: AlwaysAllow  # 更改为 Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
containerRuntimeEndpoint: ""
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMaximumGCAge: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
    text:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
resolvConf: /run/systemd/resolve/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

# 这个操作对应
# Ensure that the anonymous-auth argument is set to false
# Ensure that the -authorization-mode argument is not set to AlwaysAllow
```

```bash
vim /etc/kubernetes/manifests/etcd.yaml
```

```yaml
...
...
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.40.10:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://192.168.40.10:2380
    - --initial-cluster=xianchaomaster1=https://192.168.40.10:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.40.10:2379
    - --client-cert-auth=true  # 新增
...
...
# 这个操作对应 Ensure that the --client-cert-auth argument is set to true
```

```bash
systemctl daemon-reload
systemctl restart kubelet
```

等待后检查集群状态

```bash
root@xianchaomaster1:~  kubectl get nodes
NAME              STATUS   ROLES           AGE    VERSION
xianchaomaster1   Ready    control-plane   407d   v1.31.0
xianchaonode1     Ready    <none>          407d   v1.31.0
```



## Secret

Context

```bash
您需要使用存储在 TLS Secret 中的 SSL 文件来保护 Web 服务器的安全访问。
```

Task

```bash
在 smart-rose 命名空间中为名为 smart-rose 的现有 Deployment 创建名为 smart-rose 的 TLS Secret。

使用以下 SSL 文件:
证书：/home/candidate/ksmv00201-master/ca-cert/smart.web.k8s.crt
密钥：/home/candidate/ksmv00201-master/ca-cert/smart.web.k8s.key

Deployment 已配置为使用 TLS Secret。
请勿修改现有的 Deployment。
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “secret”
# 准确网址:
https://kubernetes.io/docs/concepts/configuration/secret/
```

```bash
# 检查 smart-rose 命名空间下 smart-rose 的 deployment
kubectl get deployment smart-rose -n smart-rose

# 在 smart-rose 命名空间下创建 TLS Secret
kubectl -n smart-rose create secret tls smart-rose --cert=/home/candidate/ksmv00201-master/ca-cert/smart.web.k8s.crt --key=/home/candidate/ksmv00201-master/ca-cert/smart.web.k8s.key

# 等待一下，检查 deployment 是否恢复正常
kubectl get deployment smart-rose -n smart-rose
```



## Dockerfile & Deployment Optimization

Task

```bash
1、分析和编辑给定的 Dockerfile：/home/candidate/KSSC00301/Dockerfile（基于 ubuntu:16.04镜像），并修改文件中可能涉及到的安全/最佳实践问题的一个异常。

2、分析和编辑给定的清单文件：/home/candidate/KSSC00301/deployment.yaml，并修改文件中有安全/最佳实践问题的一个异常。

提示：
1、请勿添加或删除配置项，只需修改现有配置的属性值让以上两个配置都不在有安全/最佳实践问题。
2、如果需要非特权用户来执行任何项目，则请使用用户 ID 65535 的用户 nobody。
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “securityContext”
# 准确网址:
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
```

```bash
vim /home/candidate/KSSC00301/Dockerfile
```

```bash
# /home/candidate/KSSC00301/Dockerfile

FROM ubuntu:16.04
USER root         # 修改为 nobody

```

```bash
vim /home/candidate/KSSC00301/deployment.yaml
```

```yaml
# /home/candidate/KSSC00301/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: test
  labels:
    app: dev
    name: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dev      # 确认和 .template.metadata.labels 中标签一致
  template:
    metadata:
      labels:
        app: dev
    spec:
      containers:
      - image: busybox:1.28
        name: nginx
        imagePullPolicy: IfNotPresent
        command: ["sh","-c","sleep 3600"]
        securityContext:
          readOnlyRootFilesystem: false # 改为 true 只读
          runAsUser: 2000               # 改为 nobody id 65535
          capabilities:
            add:
              - NET_ADMIN
              - NET_BIND_SERVICE  # 允许绑定到低于 1024 的端口
            drop:
              - all
          privileged: true              # 变更为 false
```

```bash
# 先删除，再应用
kubectl delete -f /home/candidate/KSSC00301/deployment.yaml
kubectl apply -f /home/candidate/KSSC00301/deployment.yaml
```



## securityContext

Context

```bash
必须更新一个现有的Pod，以确保其容器的不变性
```

Task

```bash
修改运行在namespace app名称空间里名为lamp-deployment的现有 Deployment，以便其容器：

·使用用户ID 30000运行
·使用只读的根文件系统
·禁止特权提升

您可以在/home/candidate/kssc00401/lamp-deployment.yaml中找到Deployment的清单文件。
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “securityContext”
# 准确网址:
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
```

```bash
vim /home/candidate/kssc00401/lamp-deployment.yaml
```

```yaml
# /home/candidate/kssc00401/lamp-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: lamp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lamp
  template:
    metadata:
      labels:
        app: lamp
    spec:
      containers:
      - name: nginx-1
        image: busybox:1.28
        imagePullPolicy: IfNotPresent
        command: ["sh","-c","sleep 3600"]
        securityContext:                    # ---
          runAsUser: 30000
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false   # ---
      - name: nginx-2
        image: docker.io/library/nginx:1.26.2-alpine3.20-perl
        imagePullPolicy: IfNotPresent
        command: ["sh","-c","sleep 3600"]
        securityContext:                    # ---
          runAsUser: 30000
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false   # ---
```

```bash
# 先删除，后创建
kubectl delete -f /home/candidate/kssc00401/lamp-deployment.yaml
kubectl apply -f /home/candidate/kssc00401/lamp-deployment.yaml
```

```bash
# 注意的是

考试要看spec.template.spec.securityContext 这个字段是否存在，如果存在，直接改这个字段即可，但是这个字段只能包含runAsUser，它是作用于整个 Pod 的设置。

readOnlyRootFilesystem 和 allowPrivilegeEscalation 属性已移到每个容器的 securityContext 中，要是增加这两个属性，还需要在每个containers下添加securityContext字段，添加readOnlyRootFilesystem 和 allowPrivilegeEscalation这两个字段。

# 所以也可以这么写
```

```yaml
# /home/candidate/kssc00401/lamp-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: lamp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lamp
  template:
    metadata:
      labels:
        app: lamp
    spec:
      securityContext:                      # ---
        runAsUser: 30000                    # ---
      containers:
      - name: nginx-1
        image: busybox:1.28
        imagePullPolicy: IfNotPresent
        command: ["sh","-c","sleep 3600"]
        securityContext:                    # ---
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false   # ---
      - name: nginx-2
        image: docker.io/library/nginx:1.26.2-alpine3.20-perl
        imagePullPolicy: IfNotPresent
        command: ["sh","-c","sleep 3600"]
        securityContext:                    # ---
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false   # ---
```



## Kubernetes RBAC

Context

```bash
出于测试目的，由 kubeadm 创建的 cluster 的 Kubernetes API 服务器临时配置为允许未经身份验证和未经授权的访问。
```

Task

```bash
重新配置cluster的kubernetes API服务，以确保只允许经过身份验证和授权的REST请求。具体要求如下:

1、 禁止匿名身份验证
2、 使用授权模式Node，RBAC
3、 使用准入控制器NodeRestriction

注意：所有kubectl配置环境/文件也被配置成使用未经身份验证和未经授权的访问。你不必更改它，但请注意，一旦完成集群的安全加固，kubectl的配置将无法工作，您可以使用集群位于/etc/kubernetes/admin.conf 的原始 kubectl 配置文件来访问受保护的集群。

最后，请删除ClusterRoleBinding system:anonymous 来进行清理。
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “kube-apiserver”
# 准确网址:
https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
```

```bash
sudo -i
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
# /etc/kubernetes/manifasts/kube-apiserver

...
...
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.40.10
    - --allow-privileged=true
    - --authorization-mode=AlwaysAllow   # 授权模式 修改为 Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction # 准入控制器 为 NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --anonymous-auth=true    # 匿名身份验证 改为 false
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
...
...
```

```bash
# 重启集群
systemctl daemon-reload
systemctl restart kubelet

# 检查集群状态
kubectl get nodes

# 注意，在正式考试环境，执行上述操作之后，就无法用kubectl操作k8s集群了，因为禁止了匿名访问，那如何操作k8s集群呢？kubectl 操作k8s集群需要指定--kubeconfig=/etc/kubernetes/admin.conf文件操作k8s。
```

删除 ClusterRoleBinding

```bash
# 首先看看自己k8s集群有没有 system:anonymous 这个 clusterrolebinding
kubectl get clusterrolebinding --kubeconfig=/etc/kubernetes/admin.conf | grep -E '^system:anonymous(\s|$)'

# 如果有，直接删除
kubectl delete clusterrolebinding system:anonymous --kubeconfig=/etc/kubernetes/admin.conf

# 如果没有，则这样获取
kubectl get clusterrolebinding --all-namespaces -o yaml --kubeconfig=/etc/kubernetes/admin.conf | grep -A 20 -B 20 "name: system:anonymous"
# 这样能快速找到system:anonymous绑定的是哪个clusterrolebinding

# 再执行如下命令删除clusterrolebinding
kubectl delete clusterrolebinding <clusterrolebinding-name> --kubeconfig=/etc/kubernetes/admin.conf
```



## Admission Control

Context

```bash
cluster上设置了容器镜像扫描器，但尚未完全集成到cluster的配置中。您必须将容器镜像扫描完全集成到kubeadm配置的集群里。
```

Task

```bash
在/etc/kubernetes/controlconf目录中，有不完整的配置，以及具有HTTPS 端点的https://wakanda:8080/image_policy的功能性容器镜像扫描器：

首先，重新配置 API 服务器以启用所有准入控制插件，以支持提供的 AdmissionConfiguration设置。
接着，重新配置 ImagePolicyWebhook，确保在后端失效时拒绝不符合政策的镜像。
最后，为了验证配置的有效性，部署在/home/candidate/kssc00601/vulnerable-resource.yaml文件中定义的测试资源，该资源使用一个应被拒绝的镜像。

您可以根据需要删除并重新创建该资源进行测试
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “admission controllers”
# 准确网址:
https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
```

```bash
# 提权
sudo -i
cd /etc/kubernetes/controlconf
vim image-policy-config.yaml
```

```yaml
# /etc/kubernetes/controlconf/image-policy-config.yaml

apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: ImagePolicyWebhook
    configuration:
      imagePolicy:                         # ---
        kubeConfigFile: /etc/kubernetes/controlconf/kube-config.yaml
        allowTTL: 50
        denyTTL: 50
        retryBackoff: 500
        defaultAllow: false                # ---
```

```bash
vim kube-config.yaml # 具体这个文件叫什么实际考试环境不确定，需要找
```

```yaml
# /etc/kubernetes/controlconf/kube-config.yaml

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJUFNZQ2JGZFFNUkl3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRFeE1qWXdOekF5TURCYUZ3MHpOREV4TWpRd056QTNNREJhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURnNEVCVmNKOG5LcDk1Z1JEN2hBQVIwMjhkUTNNMVVJYWF2cWVTV0FXclZZbDZhVzhiWG9uU28vSkUKckdTSHcycWJiWWFyVFN5TkVuVnlCb3ljNitpNjFYRlRtVHVXMDhBODR3WDBTdGZhMnY3RFNnN2ExTlhNclJGTQpFaU1ydGJUS1FXTEduOHIzWWd4MnlabUorMWNlSEZCblU3dWYzNXpPSXRMWUR5N3BUOU0rOHlZREJkOHpDRWgvCnovNk0zQitVVGlCcWhUOGQyM214TEp4Uks3c3Z3S2VZbVd5WlZSKys4VzRyLzU1M0twYnlTYXJ3VFBZdG9TdUQKdVFabmRJKzdZUkxTU21lK3JLODZINkJNVVo3dG13bU95SXBKWUdoMmgxS0JXUFl6dXc3Ky8xVXRVNTlXcjJPQwpDeFdwYmFxNUxPcmZIelcrWlZBdUcyUnhWS0ZiQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRTFNHM3piOVFDQmU1VS85UFhJYS9uVkhVL3RUQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRRFFtWDlFeGExSgpobTNIQ2Q5Uk1QMHhDdm52dDlZMjkyT0NrcHBMcFhZOWtqa1J3YS8rZjJEckowWk0rUmJCMmtqK0RwR3k5SXdlCkRXVzIzVVUzWkRCOFF2M3g3Q1E3YnhBTUtHcS9oWGhiand1K0Y2SCtmWDhNZzBWWlhUZGIvTVB5cHp0MFdsdnQKSngxTTBPWElVS084NG9UQ2h3SzV2ci9VSmxXUGdJMVZkZm9JbUNjMEJDQjRPMGxqcm0yUjNjNGI2VkJpcUxVTgoxQkFTMDNLd1orZ3R6VU4zcWxaS3pwckpGTmpVRHpOZndqL04wWHZzdUU1MW8vRGpjNVZPQ01JMmp4RytpejRuCjNrc2xGL21WRlpPcFdOMTlaM25SeS96TVVjWWI2aEE0UFd4c3ZNVGV1UEF5d0dESDNseUVWb3hzVzExcS9Zc2IKMThoUi9yZW54Y2Y1Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://wakanda:8080/image_policy  # HTTPS 端点按照题目设置
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURLVENDQWhHZ0F3SUJBZ0lJS0U1ODg3VVhobVV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRFeE1qWXdOekF5TURCYUZ3MHlOVEV4TWpZd056QTNNREJhTUR3eApIekFkQmdOVkJBb1RGbXQxWW1WaFpHMDZZMngxYzNSbGNpMWhaRzFwYm5NeEdUQVhCZ05WQkFNVEVHdDFZbVZ5CmJtVjBaWE10WVdSdGFXNHdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDOXhQWEYKN0FiOWg0MkRYWTRKcDJ5d3BIb21VODNOY3JLYW5NbXJVeXhOZFhrY25xWll4bkFwZHFrQVNrZzYrNXVvdm5WNgo2YWdWOVo4dU9UMzFCdC9xQWVvMWJ2RXRiNmhRbWVKRFBKTVpXNkdjbWFVOE5xY0kzU1N6RmxsQllMdXpDQ1RKCjlLcThYUmdzSzlLaTl3R0xLeXpyVFNBU2lPN0IrRHBQOTZIRkJlZXRpQTFGRXQ2Y1VkbGZwOGNFTk1ONklzTFcKNXBTcC9OcWU1MW9yMWE5ZS9nUkR2SkM4V291VzUrdk1ReTFpWmZVbFAvcHZvbHpSdS9HbEgvcHRNUiswR3N0OQpmRVNJNXpvQlRCN0c4b2VETTlyQmNVSjl2N1pZdGNMTS84WDB4V0NzSnBQRWREREZaa3dTbFpHSitCdVJxdlQvCmVvZkFscXVweUE1K00zekZBZ01CQUFHalZqQlVNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUsKQmdnckJnRUZCUWNEQWpBTUJnTlZIUk1CQWY4RUFqQUFNQjhHQTFVZEl3UVlNQmFBRkF0SWJmTnYxQUlGN2xULwowOWNocitkVWRUKzFNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUM5WWdBZ0tpb0U3ZlBvTVBmY0dZN0Y5L0pGClRMMExnZXo3K3E4NlRvMFBSQi9rT25OUzM4YnZnVHFCRGlVdmZYOG1ZcjV2UnFTbkFxdE1MQklNSFMybG9DZnEKMXZONTQ0YU5lU05xeGhhTnVkUUozbWMwaWFZUk1vZG9RSVM5Z0JneFFDdmZDOTE0aG5MaER4cU43UklZWUVJZAozTldST2JNNVZzazlWekJQZFNNUFhjUXFxOVRZMmpiSm9zU2N4ZFVhZlBtNXdvUHBsRWNpejBqTVJEWldUczhMCjc2WGRkN1lKdGVUeWR3Y211dUZscFRHTmpiT1BEZHFpSytMa2pXU2J0eWxNYVRReHBTSDg2RHFDS1dweGlBaGUKMFI4T3N6TCtQQzhzVFh2ZnJGeXcxcVNwRjhnMCt1eTRGZlZTKzRVa0UyZjZ4K1VFVzYreGxXNWJKTURQCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBdmNUMXhld0cvWWVOZzEyT0NhZHNzS1I2SmxQTnpYS3ltcHpKcTFNc1RYVjVISjZtCldNWndLWGFwQUVwSU92dWJxTDUxZXVtb0ZmV2ZMams5OVFiZjZnSHFOVzd4TFcrb1VKbmlRenlUR1Z1aG5KbWwKUERhbkNOMGtzeFpaUVdDN3N3Z2t5ZlNxdkYwWUxDdlNvdmNCaXlzczYwMGdFb2p1d2ZnNlQvZWh4UVhucllnTgpSUkxlbkZIWlg2ZkhCRFREZWlMQzF1YVVxZnphbnVkYUs5V3ZYdjRFUTd5UXZGcUxsdWZyekVNdFltWDFKVC82CmI2SmMwYnZ4cFIvNmJURWZ0QnJMZlh4RWlPYzZBVXdleHZLSGd6UGF3WEZDZmIrMldMWEN6UC9GOU1WZ3JDYVQKeEhRd3hXWk1FcFdSaWZnYmthcjAvM3FId0phcnFjZ09mak44eFFJREFRQUJBb0lCQUJJTVhuZWhlQlM2eEttKwp4eGlCOU9OajhUNGQ4RS9lM2IrNHM3RjRxcGovV0RKeG9FNkhLUG00a0dBM3NHRHp0eDA0YUFIMW9RZmRvWE1LCi9LcUdLZHVlclFEQitXd2gxM2M2KzNyN2t0M3hpaEJUeUpST2VscHNkZVlXZFF5enY0WktldjArS05MYlk4WW4Kc05QUS9ET1pPcDl5YVY5NTZJTklNWHVUaUs0dE5IM1Nyc3d4NTJldFlOM3hQTnQrOGtqdWlFK1FMeGtqYjNRWgpXSjlqcHFkZEE1eDlqMVRNWGIwek81QVJYejVQMUZsTTJIRFA3T1pLWXB6ZlhzMXJqNVVYb2hRU1E0em90YU15Cng4STBvMmNZTllWeDAxMHpYTmhQNmFldW1vaDNOUVk2Z2M1dXpVSWREa3JyTGN3RWllNHZNcU5YNGJBbk1Fc1UKMXVJbHMyRUNnWUVBeVBkVjVDZFJERWpReVc5WXNka0hrc1BzZ3l6QVdMZWEzVXhyNnZyL1lkTUgvWU5nN0gyNgpYQytZbzZMQUd3WFFHY2dBK1VMUmtYSDNVUUk5Vkh4TWQyMENjMVpNeTBTazU3K3d6elExTGFqZ1VxTnh5Y3llCllEVEpRaDZTQ3FrVFV4UEV2ay9sQ3dkNURidlZ6S1E4MHNuVTlqTkI0Wm14MFYwTkkzNG1jV2tDZ1lFQThieXQKbU1FU0RSTnc4Slo3RkNjZVZOR2RQeUIwNW9FZ1NxTm0wK2pUd1RRKzJQb09oQ2NsSnorREtrcHB3ajVsQ1Z2MwpCRHA2T2lLNVM3QjdGQ1VmR2k4UExhZnVZL3dBMUo4bW9MdEMyYU5PWDFqaGFqRkpBb2dia250b0xaaHRtNFZ0CklJbHdqOHJrZW9LTm9kcnFlbE1RVXczL01Edk45UnZFYUMvMUtQMENnWUVBclFEUGZpT2lqL0szV2xGeWgxZ1EKUHZaUFN2VmhlSDVHNFMrQ3o3elgwUHo4cWU5SnB3enRPNkwxd2hpL1RBUUxDOGF6bitFM3kvL1NLbmpGRjFBUgorOVZxQUtSUVk4UnFPZDg1ZElhN0tOMXlqM0dJNlhJdS9SODBDcW1LaTRiVnpmVDhyK0RUaWxVYWp3b2VtWmJoCmpZeVd1b09SdVlhNEgwWDlvNHBieWRFQ2dZRUF6UzcwUG1NcWVqVFZPVERST1ZMVzJQR3V3ZVUvdEdOSDBIS1AKbGpEYWcvUmZuL1hubWw1TGw5dTk3b2lJNmhuaDBxYmZyUlFocVBUT1NLTjhaS1g1bDFUNFVpMW5HRERQVjZuYQp0TFVkMGZOZVUybnlzeHN3T0ZqazVsbWZISXgwQkh5bEd1ZnR4ZTlXTFhKZzIxQWdsRUdxNm9SSDVWM3R2QzJjCmNUNjduZFVDZ1lFQWgva2NJdU9qckZkZ01kK1BHdy9PRisraVpaVlFWNFZFc3hQMkRtRlh6em5CdDkwMFhzQmUKbTF2a0x2ZWw4SlRCMlNrMGZxTlVPbU9Ya2lwVmRyTk03TTQ1RU00Q3ljQ3ZOV0Y5bDlQQVdrZ1hYbHlIdVBkcApZcVR4OVVrSUJWcHVhOHJqQm5jWVZOaSsxY0lFQVF2cUZBWllRekU1YmR0VU5xOUhrckE0Q1MwPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

```bash
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml

...
...
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.40.10
    - --allow-privileged=true
    - --authorization-mode=AlwaysAllow
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook # 没有则添加 另外添加上题目要求的 ImagePolicyWebhook
    - --admission-control-config-file=/etc/kubernetes/controlconf/image-policy-config.yaml # 如果没有则添加 值为刚开始修改的 image-policy-config.yaml
    - --enable-bootstrap-token-auth=true
    - --anonymous-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
...
...
```

```bash
# 考试的时候需要检查 volume 和 volumeMounts 存不存在，若不存在需要添加
```

```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml

...
...
    volumeMounts:                                  
    - mountPath: /etc/kubernetes/controlconf    # ---
      name: config                              # ---
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:                            # ---
      path: /etc/kubernetes/controlconf  # 这里是题目要求的目录
      type: DirectoryOrCreate
    name: config                         # ---
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}
...
...
```

```bash
# 重启集群
systemctl daemon-reload
systemctl restart kubelet
# 此时测试
kubectl apply -f /home/candidate/kssc00601/vulnerable-resource.yaml
# 此时应该被拒绝，但模拟环境失效
```



## istio

Context

```bash
您必须使用 Istio 保护使用未加密的第4层(L4)传输的基于微服务的应用程序。
```

Task

```bash
执行以下任务以使用 Istio 保护现有应用程序的第4层(L4)传输通信。
为了保护第4层(L4)通信，已安装lstio。

您可以使用浏览器访问 Istio 的文档。

首先，确保 mtls namespace 中的所有 Pod 都已注入 istio-proxy sidecar 。
接下来，为 mtls namespace 中的所有工作负载配置严格模式下的相互身份验证。
```

Resolve

```bash
# 注意，这个题无法模拟
# 参考文档
https://istio.io/v1.24/docs/setup/additional-setup/sidecar-injection/
https://istio.io/v1.24/docs/reference/config/security/peer_authentication/
```

```bash
# 给名称空间下的 Pod 打上 istio-injection=enable 标签
kubectl label namespace mtls istio-injection=enabled
# 检查标签是否存在
kubectl get ns mtls --show-labels

# 把deployment管理的pod删除，重建，这样新的pod，就会自动注入envoy代理了。
kubectl rollout restart deployment -n mtls
```

打开第二个链接

```yaml
# ~/istio.yaml
# 配置严格模式下的相互身份验证

apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: foo   # 改为 mtls
spec:
  mtls:
    mode: STRICT
```

```bash
kubectl apply -f istio.yaml
```



## 从 Docker 组删除用户

Task

```bash
执行以下任务，来保护kssc00801集群

从 docker 组中删除developer用户

注意: 不要从其他组中删除用户

重新配置并重启 Docker 守护进程，确保位于/var/run/docker.sock 的套接字文件由 root 组拥有。
重新配置并重启 Docker 守护进程，确保它不监听任何 TCP 协议的端口。

注意: 完成任务后，确保 Kubernetes 集群保持健康运行状态。
```

Resolve

```bash
# 提权
sudo -i

# 从 docker 组中删除 developer 用户
gpasswd -d developer docker

# 修改 docker.socket 配置文件
vim /usr/lib/systemd/system/docker.socket
```

```bash
# /usr/lib/systemd/system/docker.socket

[Unit]
Description=Docker Socket for the API

[Socket]
# If /var/run is not implemented as a symlink to /run, you may need to
# specify ListenStream=/var/run/docker.sock instead.
ListenStream=/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker  # 更改为 root

[Install]
WantedBy=sockets.target
```

```bash
# 重启 docker 服务
systemctl daemon-reload && systemctl restart docker

# 修改 docker.service
vim /usr/lib/systemd/system/docker.service
```

```bash
# /usr/lib/systemd/system/docker.service

...
...
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock # 删除 -H tcp://0.0.0.0:2375 
...
...
```

```bash
# 重启 docker 服务
systemctl daemon-reload && systemctl restart docker
```



## Upgrading Kubeadm clusters

Context

```bash
当前使用 kubeadm 配置的集群最近完成了一次升级。由于兼容性原因，有一个节点被保留在较旧的版本上。
```

Task

```bash
升级集群中的节点 node01，使其与 control plane 节点的版本保持一致。
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “kubeadm upgrade”
# 准确网址:
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
```

```bash
# 提权
sudo -i
kubectl get nodes
```

```bash
# 过滤 1.311 这个版本号的详细信息
apt-cache show kubeadm | grep 1.31.1

apt-mark unhold kubelet
apt-get install kubelet=1.31.1-1.1 -y
systemctl daemon-reload && systemctl restart kubelet

# 检查集群状态
kubectl get nodes
```



## Ingress HTTPS

Context

```bash
您需要使用 HTTPS 路由来对外公开 Web 应用。
```

Task

```bash
在 prod01 namespace 创建一个名为 web 的 Ingress 资源，并根据以下要求配置：

1）将主机 web.k8singress.local 和所有路径的流量路由到现有的 web Service。
2）使用已存在的 web-cert Secret 进行 TLS 终止。
3）将所有 HTTP 请求重定向至 HTTPS。

提示：可使用以下命令测试 Ingress 配置：
curl -Lk https://web.k8singress.local
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “ingress”
# 准确网址:
https://kubernetes.io/docs/concepts/services-networking/ingress/
```

```bash
# 查看 Service 详细信息
[Arch ~]: kubectl describe service web -n prod01
Name:                     web
Namespace:                prod01
Labels:                   <none>
Annotations:              <none>
Selector:                 app=web
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.108.99.62
IPs:                      10.108.99.62
Port:                     <unset>  80/TCP    # 找到这个
TargetPort:               80/TCP
Endpoints:                10.244.121.32:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

```bash
# 查看 IngressClass
kubectl get ingressclass

# 编写 Ingress
vim ingress.yaml
```

```yaml
# ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: prod01
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true" # 启用 HTTP 到 HTTPS 重定向
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    -  web.k8singress.local
    secretName: web-cert  # 使用现有的 web-cert Secret
  rules:
  - host: web.k8singress.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80    # 对应 Service 的 Port
```

```bash
kubectl apply -f ingress.yaml
```

```bash
# 测试
curl -Lk https://web.k8singress.local
```



## Falco

Context

```bash
检测到 Pod 行为异常，可能对系统安全产生威胁。
```

Task

```bash
属于应用程序 olla 的一个 Pod 出现异常，正从敏感文件 /dev/mem 直接读取系统内存数据。

首先，识别出正在访问 /dev/mem 的异常 Pod。
然后，将该异常 Pod 所属的 Deployment 缩减至零副本。

注意事项：
  1. 除缩小 pod 副本外，不修改 Deployment 的其他配置。
  2. 不对其他 Deployment 进行更改。
  3. 不要删除任何 Deployment。
```

Resolve

```bash
# 参考官网
https://falco.org/docs/rules/basic-elements/
```

```bash
# 配置 Falco 规则
sudo -i
vim /etc/falco/falco_rules.local.yaml
```

新增如下

```yaml
# /etc/falco/falco_rules.local.yaml

- rule: devmem
  desc: devmem
  condition: >
    fd.name == "/dev/mem" and evt.type in (open,read,write,mmap,ioctl)
  output: >
    Process ID: %proc.pid
    Command: %proc.cmdline
    file: %fd.name
    Container ID: %container.id
    Pod Name: %k8s.pod.name
  priority: NOTICE
  tags: [file]
```

```bash
# 运行 falco 扫描，记录 30 秒的扫描结果
falco -M 30 -r /etc/falco/falco_rules.local.yaml >> devmem.log

# 查看 devmem.log 内容如下
Shell (command=some_command file=/dev/mem container_id=1888b0a07518)
# 可以看到容器的 id 是 1888b0a07518

# 使用 container_id 来识别访问 /dev/mem 的容器所对应的 Pod
docker ps | grep 1888b0a07518
# 输出如下
k8s_nginx_olla-pod-name-595d9df89f-cr2sk _default_1
# 容器名称中可能包含 Kubernetes 的结构信息，例如：
k8s_<container-name>_<pod-name>_<namespace>_<random-id>

# 从容器名称 k8s_nginx_web-5678b56f9b-abcde_default_1 中提取：
  • Pod 名称：olla-pod-name-595d9df89f-cr2sk
  • 命名空间：default
# 由 pod 名字可以推出 deployment 名字是 olla-pod-name

# 查看 pod 再哪个名称空间下
exit # 返回
kubectl get pods -A | grep olla-pod-name

# 缩容 Pod 至 0
kubectl scale deployment -n default olla-pod-name --replicas=0
```



## 日志审计

Context

```bash
您必须为 kubeadm 配置的集群启用审计。
```

Task

```bash
首先，请重新配置集群中的 APIserver 服务，以便：
  
  1） /etc/kubernetes/logpolicy/sample-policy.yaml 提供了基本的策略，正在被使用
  2）日志存储在 /var/log/kubernetes/audit-logs.txt 文件中
  3）最多保留 3 个日志，保留时间为 5 天
  
注意：基本策略仅指定不记录的内容。

其次，编辑并扩展基本策略以记录：

  1）RequestResponse 级别的 persistentvolumes 事件
  2）front-apps namespace 中的 configmaps 事件的请求正文
  3）Metadata 级别的所有 namespace 中的 ConfigMap 和 Secret 的更改
  4）Metadata 级别记录所有其他请求
  
注意：确保 APIserver 使用扩展后的策略。
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “auditing”
# 准确网址:
https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/
```

```bash
# 提权
sudo -i
vim /etc/kubernetes/logpolicy/sample-policy.yaml
```

```yaml
# /etc/kubernetes/logpolicy/sample-policy.yaml

apiVersion: audit.k8s.io/v1
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  # No auditing for "controller-leader" configmap
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]
  # 在日志中用 RequestResponse 级别记录 Pod 变化。                    # ---
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["persistentvolumes"]
  # 记录 front-apps namespace下的 configmaps 请求 ( Request )
  - level: Request
    resources:
    - group: ""
      resources: ["configmaps"]
    namespaces: ["front-apps"]
  # Metadata 级别的所有 namespace 中的 ConfigMap 和 Secret 的更改
  - level: Metadata
    resources:
    - group: ""
      resources: ["configmaps","secrets"]
 # Metadata 级别记录所有其他请求 遵循第一个匹配规则胜出 一定要最后写
  - level: Metadata
    resources:
    - group: "*"
      resources: ["*"]
    omitStages:
    - "RequestReceived"                                           # ---
```

```bash
# 配置 apiserver 服务
touch /var/log/kubernetes/audit-logs.txt
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml

...
...
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.40.10
    - --allow-privileged=true
    - --authorization-mode=AlwaysAllow
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --anonymous-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --audit-policy-file=/etc/kubernetes/logpolicy/sample-policy.yaml # 审计策略文件
    - --audit-log-path=/var/log/kubernetes/audit-logs.txt # 审计日志文件
    - --audit-log-maxage=5 # 保留5天
    - --audit-log-maxbackup=3 # 保留3个日志
...
...
# 确认挂载
    - mountPath: /etc/kubernetes/logpolicy/sample-policy.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/kubernetes/audit-logs.txt
      name: audit-log
      readOnly: false
...
...
# 确认挂载
  - hostPath:
      path: /etc/kubernetes/logpolicy/sample-policy.yaml
      type: File
    name: audit
  - hostPath:
      path: /var/log/kubernetes/audit-logs.txt
      type: File
    name: audit-log                   
...
...
```

```bash
# 重启集群
systemctl daemon-reload
systemctl restart kubelet
```

```bash
# 检查集群状态
kubectl get nodes
# 查看日志信息，需要有输出
tail -f /var/log/kubernetes/audit-logs.txt
```



## Networkpolicy

Context

```bash
您需要通过 NetworkPolicy 来管理现有的 Deployments ，实现控制跨越不同 namespace 的流量。
```

Task

```bash
首先，在 prod namespace 中创建一个名为 deny 的 NetworkPolicy，以阻止所有的入口流量。
提示：prod namespace 标记的标签是 env: prod。

其次，在 data 命名空间中创建一个名为 allow-from-prod 的 NetworkPolicy，以允许仅来自 prod namespace 的 Pod 流量进入。使用 prod namespace 的标签来允许流量。
说明：data namespace 标记为 env: data。

注意：不要修改或删除任何现有的 namespace 或 Pod，只需创建题目需要的 NetworkPolicy。
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “networkpolicy”
# 准确网址:
https://kubernetes.io/docs/concepts/services-networking/network-policies/
```

```bash
vim deny.yaml
```

```yaml
# deny.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

```bash
kubectl apply -f deny.yaml
```

```bash
vim allow.yaml
```

```yaml
# allow.yaml

kind: NetworkPolicy
metadata:
  name: allow-from-prod
  namespace: data
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          env: prod
```



## ServiceAccount

Context

```bash
在安全审计中发现，某个 Deployment 存在不符合安全要求的服务账号令牌，这可能会引发安全漏洞。
```

Task

```bash
首先，需要修改 monitor namespace 中现有的 stats-monitor-sa ServiceAccount，禁止自动挂载API 凭据。
接着，修改 monitor namespace 中现有的 stats-monitor Deployment，确保 ServiceAccount 令牌挂载到 /var/run/secrets/kubernetes.io/serviceaccount/token 路径。
通过名为 token 的投射卷注入该令牌。确保令牌以只读模式挂载。。

提示: Deployment 的清单文件位于 /home/candidate/stats-monitor/deployment1.yaml
```

Resolve

```bash
# 打开 Kubernetes 官网，搜索 “serviceaccount”
# 准确网址:
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
```

```bash
kubectl edit sa stats-monitor-sa -n monitor
```

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"stats-monitor-sa","namespace":"monitor"}}
  creationTimestamp: "2024-11-27T06:20:34Z"
  name: stats-monitor-sa
  namespace: monitor
  resourceVersion: "120779"
  uid: bdd95326-78b8-4aee-84b1-07cfe12f96af
automountServiceAccountToken: false   # 新增
```

```bash
# 修改 Deployment 资源
vim /home/candidate/stats-monitor/deployment1.yaml
```

```yaml
# /home/candidate/stats-monitor/deployment1.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: stats-monitor
  namespace: monitor
spec:
  replicas: 1
  selector:
    matchLabels:
      app: stats-monitor
  template:
    metadata:
      labels:
        app: stats-monitor
    spec:
      serviceAccountName: stats-monitor-sa
      containers:
        - name: stats-monitor
          image: busybox:1.28
          imagePullPolicy: IfNotPresent
          command: ["sleep", "360000"]
          volumeMounts:                     # ---
          - mountPath: /var/run/secrets/kubernetes.io/serviceaccount/token
            name: token
            readOnly: true
      volumes:
      - name: token
        pojected:
          sources:
          - serviceAccountToken:
              path: token                   # ---
```

```bash
kubectl apply -f /home/candidate/stats-monitor/deployment1.yaml
```



## 基于 Bom 创建 SPDX 文档

Task

```bash
在 alpine namespace 中的 alpine Deployment 中，运行着三个不同版本的 Alpine 镜像容器。
首先，需要找出哪个 Alpine 镜像中包含了版本为 3.1.4-r5 的 libcrypto3 软件包。
其次，使用预安装的 bom 工具，在 /home/candidate/kssc00150/alpine.spdx 文件中为找出的镜像版本生成 SPDX 文档。
最后， 更新 alpine Deployment，删除使用上述找到镜像版本的容器。

Deployment 的清单文件可以在/home/candidate/kssc00150/alipine-deployment.yaml 中找到。

提示: 不要修改 Deployment 的任何其他不包含 3.1.4-r5 的 libcrypto3 的软件包容器。
```

Resolve

```bash
cat /home/candidate/kssc00150/alipine-deployment.yaml
```

```yaml
# /home/candidate/kssc00150/alipine-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpine
  namespace: alpine
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpine
  template:
    metadata:
      labels:
        app: alpine
    spec:
      containers:
        - name: alpine-container-1
          image: alpine:3.18.9
          imagePullPolicy: IfNotPresent
          command: [ "/bin/sh", "-c", "while true; do sleep 360000000; done" ]
        - name: alpine-container-2
          image: alpine:3.19.1
          imagePullPolicy: IfNotPresent
          command: [ "/bin/sh", "-c", "while true; do sleep 360000000; done" ]
        - name: alpine-container-3
          image: alpine:3.20.3
          imagePullPolicy: IfNotPresent
          command: [ "/bin/sh", "-c", "while true; do sleep 360000000; done" ]
```

```bash
# 通过上面的 yaml 文件可以看到 pod 里有三个容器，分别是 alpine-container-1、alpine-container-2、alpine-container-3，用的镜像分别是 alpine:3.18.9、alpine:3.19.1、alpine:3.20.3
```

```bash
# 查看 pod 名字
[Arch ~]: kubectl get pods -n alpine
NAME                      READY   STATUS    RESTARTS        AGE
alpine-574d5fd875-xsplb   2/2     Running   6 (6h51m ago)   28d

# 查看三个 pod 里面容器的软件包版本是否包含 libcrypto3

kubectl exec -it alpine-574d5fd875-xsplb -c alpine-container-1 -n alpine -- apk list | grep libcrypto3
# 显示如下
libcrypto3-3.1.7-r0 x86_64 {openssl} (Apache-2.0) [installed]

kubectl exec -it alpine-574d5fd875-xsplb -c alpine-container-2 -n alpine -- apk list | grep libcrypto3
# 显示如下
libcrypto3-3.1.4-r5 x86_64 {openssl} (Apache-2.0) [installed]

kubectl exec -it alpine-574d5fd875-xsplb -c alpine-container-3 -n alpine -- apk list | grep libcrypto3
# 显示如下
libcrypto3-3.3.2-r0 x86_64 {openssl} (Apache-2.0) [installed]

# 假如通过上面查出来 alpine-container-2 这个容器里有 libcrypto3-3.1.4-r5 这个版本软件包，那需要把 alipine-deployment.yaml 这个文件里，关于 alpine-container-2 这个容器删除。

# 生成 spdx 文件
sudo -i
bom generate --image alpine:3.19.1 --output /home/candidate/kssc00150/alpine.spdx

# 注意：如果在当前节点报错找不到这个镜像，需要 kubectl get pods -n alpine -owide，看 pod 调度到哪个机器，ssh 登录到指定的机器，在执行 bom 命令。

# 模拟环境 bom 是 docker 运行的，可以这样操作：
docker run --restart=always -e PROFILE=your_profile_value docker.io/delacruzmoises30/bom:latest generate -f spdx -o /home/candidate/kssc00150/alpine.spdx alpine:3.19.1
```

```bash
vim /home/candidate/kssc00150/alipine-deployment.yaml
```

```yaml
# /home/candidate/kssc00150/alipine-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpine
  namespace: alpine
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpine
  template:
    metadata:
      labels:
        app: alpine
    spec:
      containers:
        - name: alpine-container-1
          image: alpine:3.18.9
          imagePullPolicy: IfNotPresent
          command: [ "/bin/sh", "-c", "while true; do sleep 360000000; done" ]
        - name: alpine-container-2       # deleted
          image: alpine:3.19.1           # deleted
          imagePullPolicy: IfNotPresent  # deleted
          command: [ "/bin/sh", "-c", "while true; do sleep 360000000; done" ]                                        # deleted
        - name: alpine-container-3
          image: alpine:3.20.3
          imagePullPolicy: IfNotPresent
          command: [ "/bin/sh", "-c", "while true; do sleep 360000000; done" ]
```

```bash
# 重新创建
kubectl apply -f /home/candidate/kssc00150/alipine-deployment.yaml

# 检查
kubectl get pods -n alpine
```



## Pod 安全标准

Context

```bash
为满足要求，所有用户命名空间必须强制执行受限的 Pod 安全标准。
```

Task

```bash
在 confidential namespace 命名空间中，有一个 Deployment 不符合限制性的pod安全标准，导致其Pod 无法成功调度。

请修改该 Deployment 配置，使其符合受限标准，并验证 Pod 能够正常启动和运行。

提示: Deployment 的配置文件位于/home/candidate/kssc00160/nginx-unprivileged.yaml
```

Resolve

```bash
cat /home/candidate/kssc00160/nginx-unprivileged.yaml
```

```yaml
# /home/candidate/kssc00160/nginx-unprivileged.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-unprivileged
  namespace: confidential
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-unprivileged
  template:
    metadata:
      labels:
        app: nginx-unprivileged
    spec:
      containers:
        - name: nginx
          image: busybox:1.28
          imagePullPolicy: IfNotPresent
          command: ["sleep", "36000000"]
```

```bash
# 重新部署 Deployment 获取报错信息
kubectl delete -f /home/candidate/kssc00160/nginx-unprivileged.yaml
kubectl apply -f /home/candidate/kssc00160/nginx-unprivileged.yaml
# 报错信息如下
Error from server (Forbidden): error when creating "nginx-unprivileged.yaml": pods "nginx-unprivileged" is forbidden:
violates PodSecurity "restricted:latest":
allowPrivilegeEscalation != false (container "nginx" must set
securityContext.allowPrivilegeEscalation=false),
securityContext.capabilities.drop=["ALL"] is required,
securityContext.runAsNonRoot=true is required,
securityContext.seccompProfile.type must be set to "RuntimeDefault" or "Localhost"
# 根据报错信息，重新修改 nginx-unprivileged.yaml 文件
vim /home/candidate/kssc00160/nginx-unprivileged.yaml
```

```yaml
# /home/candidate/kssc00160/nginx-unprivileged.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-unprivileged
  namespace: confidential
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-unprivileged
  template:
    metadata:
      labels:
        app: nginx-unprivileged
    spec:
      containers:
        - name: nginx
          image: busybox:1.28
          imagePullPolicy: IfNotPresent
          command: ["sleep", "36000000"]
          securityContext:                                           # ---
            allowPrivilegeEscalation: false   # 禁止权限提升
            capabilities:
              drop: ["ALL"]                   # 删除所有 capabilities
            runAsNonRoot: true                # 以非 root 用户运行
            seccompProfile:
              type: RuntimeDefault            # 使用默认 seccomp 配置  # ----
```

```bash
# 检查集群状态
[Arch ~]: kubectl get deployment -n confidential
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
nginx-unprivileged   1/1     1            1           10m
```

