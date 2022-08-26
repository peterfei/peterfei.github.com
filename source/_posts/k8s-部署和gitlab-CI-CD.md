title: k8s 部署和gitlab CI/CD
date: 2021-03-16 16:01:03

tags:
---



### 准备工作

#### 1. 安装必备软件

```
yum install -y wget vim net-tools epel-release

```

#### 2. 关闭swap

```
swapoff -a

# 永久禁用，打开/etc/fstab注释掉swap那一行。
sed -i 's/.*swap.*/#&/' /etc/fstab

```



#### 3. 关闭selinux

```
# 临时禁用selinux
setenforce 0
# 永久关闭 修改/etc/sysconfig/selinux文件设置
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config


```



### 4. 关闭防火墙

```
systemctl disable firewalld
systemctl stop firewalld

```



## 安装Docker和配置代理

### 1. 配置yum源

```
## 配置默认源
## 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

## 下载阿里源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

## 刷新
yum makecache fast

## 配置k8s源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF

## 重建yum缓存
yum clean all
yum makecache fast
yum -y update


```



### 2. 安装docker

```
yum -y install yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install  docker-ce-20.10.5 
systemctl enable docker
systemctl start docker

```



### 3. 确保kubelet使用的cgroup driver 与 Docker的一致

```
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "registry-mirrors": ["https://ifa4ye2m.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload sudo systemctl restart docker

```

### 4. 从国内仓库拉取镜像（核心步骤-如果没有代理）

```
#!/bin/bash
images=(
kube-apiserver:v1.18.16
kube-controller-manager:v1.18.16
kube-scheduler:v1.18.16
kube-proxy:v1.18.16
pause:3.2
etcd:3.4.3-0
coredns:1.6.7
kubernetes-dashboard-amd64:v1.10.1
)
for imageName in ${images[@]};do
        docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
        docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
done
```





## 开始安装k8s



### 1. 安装kubeadm，kubelet等



```
 yum install kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0 kubernetes-cni-1.18.0

```



### 2. 集群初始化

```
## master节点执行：
sudo kubeadm init \
 --apiserver-advertise-address 10.1.69.101 \
 --kubernetes-version=v1.15.0 \
 --pod-network-cidr=10.244.0.0/16



如果外网：

kubeadm init --apiserver-advertise-address xx.xx.255.84 --pod-network-cidr=10.244.0.0/16 --kubernetes-version=v1.18.0 
```





得到回复：

```

(...省略)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

## 保存好该命令，丢了不好找回。节点加入时需要
kubeadm join 10.1.69.101:6443 --token ou5pvo.qseafc4s8licblzy \
    --discovery-token-ca-cert-hash sha256:de9c10f11c50c074f212698b9d514fc12a9c1c4ffe70961aff89ac5e585f0663

```





### 3. 拷贝配置，给kubectl使用

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```



### 4. 安装flannel网络

```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

## 查看flannal是否安装成功

sudo kubectl -n kube-system get po -l app=flannel -o wide

```



## 5. node节点加入集群



其他节点执行：

```
kubeadm join 10.1.69.101:6443 --token ou5pvo.qseafc4s8licblzy \
    --discovery-token-ca-cert-hash sha256:de9c10f11c50c074f212698b9d514fc12a9c1c4ffe70961aff89ac5e585f0663

```



## 清理安装

 

- 如果安装过程中出任何问题，可以重置后重新安装

```
sudo kubeadm reset

```



### 安装 helm 和 gitlab-runner



```
 tar -zxvf helm-v2.12.1-linux-amd64.tar.gz
 cd linux-amd64/
 
 cp helm /usr/local/bin
```
`cat tiller.yaml`
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: helm
    name: tiller
  name: tiller-deploy
  namespace: kube-system
spec:
  replicas: 1
  strategy: {}
  selector:
    matchLabels:
      name: tiller
      app: helm
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: helm
        name: tiller
    spec:
      automountServiceAccountToken: true
      containers:
      - env:
        - name: TILLER_NAMESPACE
          value: kube-system
        - name: TILLER_HISTORY_MAX
          value: "0"
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3
        imagePullPolicy: IfNotPresent
        livenessProbe:
          httpGet:
            path: /liveness
            port: 44135
          initialDelaySeconds: 1
          timeoutSeconds: 1
        name: tiller
        ports:
        - containerPort: 44134
          name: tiller
        - containerPort: 44135
          name: http
        readinessProbe:
          httpGet:
            path: /readiness
            port: 44135
          initialDelaySeconds: 1
          timeoutSeconds: 1
        resources: {}
      serviceAccountName: tiller
status: {}

---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: helm
    name: tiller
  name: tiller-deploy
  namespace: kube-system
spec:
  ports:
  - name: tiller
    port: 44134
    targetPort: tiller
  selector:
    app: helm
    name: tiller
  type: ClusterIP
status:
  loadBalancer: {}
```

 


``` 
kubectl apply -f tiller.yaml
helm init --upgrade --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

#helm init -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3 --stable-repo-url http://mirror.azure.cn/kubernetes/charts/ --service-account tiller --override spec.selector.matchLabels.'name'='tiller',spec.selector.matchLabels.'app'='helm' --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' | kubectl delete -f -
 
 kubectl create serviceaccount --namespace kube-system tiller
 kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
 kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'


 kubectl get po  -A|grep tiller
 kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

 helm repo add gitlab https://charts.gitlab.io
 helm repo update
 helm fetch gitlab/gitlab-runner --version "v0.10.0-rc1"
 
 kubectl create namespace gitlab-runners
 mkdir -p gitlab-runner
 cd gitlab-runner/
 kubectl create -f rbac-runner-config.yaml 

```

cat rbac-runner-config.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitlab
  namespace: gitlab-runners
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: gitlab-runners
  name: gitlab
rules:
- apiGroups: [""] #"" indicates the core API group
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["*"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gitlab
  namespace: gitlab-runners
subjects:
- kind: ServiceAccount
  name: gitlab # Name is case sensitive
  apiGroup: ""
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: gitlab # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io

```



>  `cat  values-spm-operation-frontend.yaml`



```
## GitLab Runner Image
##
## By default it's using gitlab/gitlab-runner:alpine-v{VERSION}
## where {VERSION} is taken from Chart.yaml from appVersion field
##
## ref: https://hub.docker.com/r/gitlab/gitlab-runner/tags/
##
# image: gitlab/gitlab-runner:alpine-v11.6.0

## Specify a imagePullPolicy
## 'Always' if imageTag is 'latest', else set to 'IfNotPresent'
## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
##
imagePullPolicy: IfNotPresent
gitlabUrl: http://xxx.xxx.cn/
runnerRegistrationToken: "-Rxxz6Cqdk33Q96mRdxk"

## 和之前配置的 rbac name 对应

## 指定关联 runner 的标签
tags: " operations"

privileged: true
serviceAccountName: gitlab
## The GitLab Server URL (with protocol) that want to register the runner against
## ref: https://docs.gitlab.com/runner/commands/README.html#gitlab-runner-register
##
# gitlabUrl: http://gitlab.your-domain.com/

## The Registration Token for adding new Runners to the GitLab Server. This must
## be retrieved from your GitLab Instance.
## ref: https://docs.gitlab.com/ce/ci/runners/README.html
##
# runnerRegistrationToken: ""

## The Runner Token for adding new Runners to the GitLab Server. This must
## be retrieved from your GitLab Instance. It is token of already registered runner.
## ref: (we don't yet have docs for that, but we want to use existing token)
##
# runnerToken: ""
#
## Unregister all runners before termination
##
## Updating the runner's chart version or configuration will cause the runner container
## to be terminated and created again. This may cause your Gitlab instance to reference
## non-existant runners. Un-registering the runner before termination mitigates this issue.
## ref: https://docs.gitlab.com/runner/commands/README.html#gitlab-runner-unregister
##
unregisterRunners: true

## When stopping ther runner, give it time to wait for it's jobs to terminate.
##
## Updating the runner's chart version or configuration will cause the runner container
## to be terminated with a graceful stop request. terminationGracePeriodSeconds
## instructs Kubernetes to wait long enough for the runner pod to terminate gracefully.
## ref: https://docs.gitlab.com/runner/commands/#signals
terminationGracePeriodSeconds: 3600

## Set the certsSecretName in order to pass custom certficates for GitLab Runner to use
## Provide resource name for a Kubernetes Secret Object in the same namespace,
## this is used to populate the /home/gitlab-runner/.gitlab-runner/certs/ directory
## ref: https://docs.gitlab.com/runner/configuration/tls-self-signed.html#supported-options-for-self-signed-certificates
##
# certsSecretName:

## Configure the maximum number of concurrent jobs
## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-global-section
##
concurrent: 10

## Defines in seconds how often to check GitLab for a new builds
## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-global-section
##
checkInterval: 30
limitLine: 100000
## Configure GitLab Runner's logging level. Available values are: debug, info, warn, error, fatal, panic
## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-global-section
##
# logLevel:

## Configure GitLab Runner's logging format. Available values are: runner, text, json
## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-global-section
##
# logFormat:

## For RBAC support:
rbac:
  create: false
  serviceAccountName: gitlab
  ## Define specific rbac permissions.
  # resources: ["pods", "pods/exec", "secrets"]
  # verbs: ["get", "list", "watch", "create", "patch", "delete"]

  ## Run the gitlab-bastion container with the ability to deploy/manage containers of jobs
  ## cluster-wide or only within namespace
  clusterWideAccess: false

  ## Use the following Kubernetes Service Account name if RBAC is disabled in this Helm chart (see rbac.create)
  ##
  # serviceAccountName: default

## Configure integrated Prometheus metrics exporter
## ref: https://docs.gitlab.com/runner/monitoring/#configuration-of-the-metrics-http-server
metrics:
  enabled: true

## Configuration for the Pods that that the runner launches for each new job
##
runners:
  ## Default container image to use for builds when none is specified
  ##
  image: ubuntu:16.04

  ## Specify one or more imagePullSecrets
  ##
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # imagePullSecrets: []

  ## Specify the image pull policy: never, if-not-present, always. The cluster default will be used if not set.
  ##
  # imagePullPolicy: ""

  ## Defines number of concurrent requests for new job from GitLab
  ## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runners-section
  ##
  # requestConcurrency: 1

  ## Specify whether the runner should be locked to a specific project: true, false. Defaults to true.
  ##
  # locked: true

  ## Specify the tags associated with the runner. Comma-separated list of tags.
  ##
  ## ref: https://docs.gitlab.com/ce/ci/runners/#using-tags
  ##
  # tags: ""

  ## Specify if jobs without tags should be run.
  ## If not specified, Runner will default to true if no tags were specified. In other case it will
  ## default to false.
  ##
  ## ref: https://docs.gitlab.com/ce/ci/runners/#allowing-runners-with-tags-to-pick-jobs-without-tags
  ##
  # runUntagged: true

  ## Run all containers with the privileged flag enabled
  ## This will allow the docker:dind image to run if you need to run Docker
  ## commands. Please read the docs before turning this on:
  ## ref: https://docs.gitlab.com/runner/executors/kubernetes.html#using-docker-dind
  ##
  privileged: false

  ## The name of the secret containing runner-token and runner-registration-token
  # secret: gitlab-runner

  ## Namespace to run Kubernetes jobs in (defaults to the same namespace of this release)
  ##
  # namespace:

  ## Distributed runners caching
  ## ref: https://gitlab.com/gitlab-org/gitlab-runner/blob/master/docs/configuration/autoscale.md#distributed-runners-caching
  ##
  ## If you want to use s3 based distributing caching:
  ## First of all you need to uncomment General settings and S3 settings sections.
  ##
  ## Create a secret 's3access' containing 'accesskey' & 'secretkey'
  ## ref: https://aws.amazon.com/blogs/security/wheres-my-secret-access-key/
  ##
  ## $ kubectl create secret generic s3access \
  ##   --from-literal=accesskey="YourAccessKey" \
  ##   --from-literal=secretkey="YourSecretKey"
  ## ref: https://kubernetes.io/docs/concepts/configuration/secret/
  ##
  ## If you want to use gcs based distributing caching:
  ## First of all you need to uncomment General settings and GCS settings sections.
  ##
  ## Access using credentials file:
  ## Create a secret 'google-application-credentials' containing your application credentials file.
  ## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runnerscachegcs-section
  ## You could configure
  ## $ kubectl create secret generic google-application-credentials \
  ##   --from-file=gcs-applicaton-credentials-file=./path-to-your-google-application-credentials-file.json
  ## ref: https://kubernetes.io/docs/concepts/configuration/secret/
  ##
  ## Access using access-id and private-key:
  ## Create a secret 'gcsaccess' containing 'gcs-access-id' & 'gcs-private-key'.
  ## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runners-cache-gcs-section
  ## You could configure
  ## $ kubectl create secret generic gcsaccess \
  ##   --from-literal=gcs-access-id="YourAccessID" \
  ##   --from-literal=gcs-private-key="YourPrivateKey"
  ## ref: https://kubernetes.io/docs/concepts/configuration/secret/
  cache: {}
    ## General settings
    # cacheType: s3
    # cachePath: "gitlab_runner"
    # cacheShared: true

    ## S3 settings
    # s3ServerAddress: s3.amazonaws.com
    # s3BucketName:
    # s3BucketLocation:
    # s3CacheInsecure: false
    # secretName: s3access

    ## GCS settings
    # gcsBucketName:
    ## Use this line for access using access-id and private-key
    # secretName: gcsaccess
    ## Use this line for access using google-application-credentials file
    # secretName: google-application-credentials

  ## Build Container specific configuration
  ##
  builds: {}
    # cpuLimit: 200m
    # memoryLimit: 256Mi
    # cpuRequests: 100m
    # memoryRequests: 128Mi

  ## Service Container specific configuration
  ##
  services: {}
    # cpuLimit: 200m
    # memoryLimit: 256Mi
    # cpuRequests: 100m
    # memoryRequests: 128Mi

  ## Helper Container specific configuration
  ##
  helpers: {}
    # cpuLimit: 200m
    # memoryLimit: 256Mi
    # cpuRequests: 100m
    # memoryRequests: 128Mi
    # image: gitlab/gitlab-runner-helper:x86_64-latest

  ## Service Account to be used for runners
  ##
  # serviceAccountName:

  ## If Gitlab is not reachable through $CI_SERVER_URL
  ##
  # cloneUrl:

  ## Specify node labels for CI job pods assignment
  ## ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
  ##
  # nodeSelector: {}

  ## Specify pod labels for CI job pods
  ##
  # podLabels: {}

  ## Specify annotations for job pods, useful for annotations such as iam.amazonaws.com/role
  # podAnnotations: {}

  ## Configure environment variables that will be injected to the pods that are created while
  ## the build is running. These variables are passed as parameters, i.e. `--env "NAME=VALUE"`,
  ## to `gitlab-runner register` command.
  ##
  ## Note that `envVars` (see below) are only present in the runner pod, not the pods that are
  ## created for each build.
  ##
  ## ref: https://docs.gitlab.com/runner/commands/#gitlab-runner-register
  ##
  # env:
  #   NAME: VALUE


## Configure resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources: {}
  # limits:
  #   memory: 256Mi
  #   cpu: 200m
  # requests:
  #   memory: 128Mi
  #   cpu: 100m

## Affinity for pod assignment
## Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
##
affinity: {}

## Node labels for pod assignment
## Ref: https://kubernetes.io/docs/user-guide/node-selection/
##
nodeSelector: {
  nodeType: k8s
}
  # Example: The gitlab runner manager should not run on spot instances so you can assign
  # them to the regular worker nodes only.
  # node-role.kubernetes.io/worker: "true"

## List of node taints to tolerate (requires Kubernetes >= 1.6)
## Ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
##
tolerations: []
  # Example: Regular worker nodes may have a taint, thus you need to tolerate the taint
  # when you assign the gitlab runner manager with nodeSelector or affinity to the nodes.
  # - key: "node-role.kubernetes.io/worker"
  #   operator: "Exists"

## Configure environment variables that will be present when the registration command runs
## This provides further control over the registration process and the config.toml file
## ref: `gitlab-runner register --help`
## ref: https://docs.gitlab.com/runner/configuration/advanced-configuration.html
##
# envVars:
#   - name: RUNNER_EXECUTOR
#     value: kubernetes

## list of hosts and IPs that will be injected into the pod's hosts file
hostAliases: []
  # Example:
  # - ip: "127.0.0.1"
  #   hostnames:
  #   - "foo.local"
  #   - "bar.local"
  # - ip: "10.1.2.3"
  #   hostnames:
  #   - "foo.remote"
  #   - "bar.remote"

## Annotations to be added to manager pod
##
podAnnotations: {}
  # Example:
  # iam.amazonaws.com/role: <my_role_arn>

## HPA support for custom metrics:
## This section enables runners to autoscale based on defined custom metrics.
## In order to use this functionality, Need to enable a custom metrics API server by
## implementing "custom.metrics.k8s.io" using supported third party adapter
## Example: https://github.com/directxman12/k8s-prometheus-adapter
##
#hpa: {}
  # minReplicas: 1
  # maxReplicas: 10
  # metrics:
  # - type: Pods
  #   pods:
  #     metricName: gitlab_runner_jobs
  #     targetAverageValue: 400m

```



###  Helm for gitlab configmap

 cat templates/configmap.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "gitlab-runner.fullname" . }}
  labels:
    app: {{ include "gitlab-runner.fullname" . }}
    chart: {{ include "gitlab-runner.chart" . }}
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"
data:
  entrypoint: |
    #!/bin/bash
    set -e
    mkdir -p /home/gitlab-runner/.gitlab-runner/
    cp /scripts/config.toml /home/gitlab-runner/.gitlab-runner/

    # Register the runner
    if [[ -f /secrets/accesskey && -f /secrets/secretkey ]]; then
      export CACHE_S3_ACCESS_KEY=$(cat /secrets/accesskey)
      export CACHE_S3_SECRET_KEY=$(cat /secrets/secretkey)
    fi

    if [[ -f /secrets/gcs-applicaton-credentials-file ]]; then
      export GOOGLE_APPLICATION_CREDENTIALS="/secrets/gcs-applicaton-credentials-file"
    else
      if [[ -f /secrets/gcs-access-id && -f /secrets/gcs-private-key ]]; then
        export CACHE_GCS_ACCESS_ID=$(cat /secrets/gcs-access-id)
        # echo -e used to make private key multiline (in google json auth key private key is oneline with \n)
        export CACHE_GCS_PRIVATE_KEY=$(echo -e $(cat /secrets/gcs-private-key))
      fi
    fi

    if [[ -f /secrets/runner-registration-token ]]; then
      export REGISTRATION_TOKEN=$(cat /secrets/runner-registration-token)
    fi

    if [[ -f /secrets/runner-token ]]; then
      export CI_SERVER_TOKEN=$(cat /secrets/runner-token)
    fi

    if ! sh /scripts/register-the-runner; then
      exit 1
    fi

    cat >>/home/gitlab-runner/.gitlab-runner/config.toml <<EOF
      output_limit = 100000
      [[runners.kubernetes.volumes.pvc]]
        name = "gitlab-runner-maven-repo-claim"
        mount_path = "/home/cache/maven"
      [[runners.kubernetes.volumes.host_path]]
        name = "docker"
        mount_path = "/var/run/docker.sock"
    EOF
    # Start the runner
    exec /entrypoint run --user=gitlab-runner \
      --working-directory=/home/gitlab-runner

  config.toml: |
    concurrent = {{ .Values.concurrent }}
    check_interval = {{ .Values.checkInterval }}
    log_level = {{ default "info" .Values.logLevel | quote }}
    output_limit = {{ default 10000 .Values.limitLine}}
    {{- if .Values.logFormat }}
    log_format = {{ .Values.logFormat | quote }}
    {{- end }}
    {{- if .Values.metrics.enabled }}
    listen_address = '[::]:9252'
    {{- end }}
  configure: |
    set -e
    cp /init-secrets/* /secrets
  register-the-runner: |
    #!/bin/bash
    MAX_REGISTER_ATTEMPTS=30

    for i in $(seq 1 "${MAX_REGISTER_ATTEMPTS}"); do
      echo "Registration attempt ${i} of ${MAX_REGISTER_ATTEMPTS}"
      /entrypoint register \
        {{- range .Values.runners.imagePullSecrets }}
        --kubernetes-image-pull-secrets {{ . | quote }} \
        {{- end }}
        {{- range $key, $val := .Values.runners.nodeSelector }}
        --kubernetes-node-selector {{ $key | quote }}:{{ $val | quote }} \
        {{- end }}
        {{- range $key, $value := .Values.runners.podLabels }}
        --kubernetes-pod-labels {{ $key | quote }}:{{ $value | quote }} \
        {{- end }}
        {{- range $key, $val := .Values.runners.podAnnotations }}
        --kubernetes-pod-annotations {{ $key | quote }}:{{ $val | quote }} \
        {{- end }}
        {{- range $key, $value := .Values.runners.env }}
        --env {{ $key | quote -}} = {{- $value | quote }} \
        {{- end }}
        {{- if and (hasKey .Values.runners "runUntagged") .Values.runners.runUntagged }}
        --run-untagged=true \
        {{- end }}
        --non-interactive

      retval=$?

      if [ ${retval} = 0 ]; then
        break
      elif [ ${i} = ${MAX_REGISTER_ATTEMPTS} ]; then
        exit 1
      fi

      sleep 5
    done

    exit 0

  check-live: |
    #!/bin/bash
    if /usr/bin/pgrep -f .*register-the-runner; then
      exit 0
    elif /usr/bin/pgrep gitlab.*runner; then
      exit 0
    else
      exit 1
    fi
```



### gitlab-runner

```
 helm install --name xxx gitlab/gitlab-runner -f values-spm-labour.yaml  --namespace gitlab-runners --version "v0.10.0-rc1"
 helm upgrade xxx ./gitlab-runner/
```

###  创建runner maven 缓存 

> Java 需要

cat maven-nfs.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-runner-maven-repo
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 5Gi
  mountOptions:
  - nolock
  nfs:
    path: /data/nfs/volumes
    server: YOURIPADDRESS
  persistentVolumeReclaimPolicy: Recycle
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-runner-maven-repo-claim
  namespace: gitlab-runners
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  volumeName: gitlab-runner-maven-repo
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 5Gi
 
```

####  生成maven缓存PVC
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-runner-maven-repo-claim
  namespace: gitlab-runners
spec:
  accessModes:
  - ReadWriteMany
    resources:
    requests:
      storage: 5Gi
    volumeName: gitlab-runner-maven-repo
status:
    accessModes:
  - ReadWriteMany
    capacity:
    storage: 5Gi
```

 #### 代码及镜像上传
```
##如果删除
helm del --purge spm-labour

##生成gitlab namespace
kubectl create ns gitlab
### 代码镜像上传密钥
kubectl create secret docker-registry  huawei-auth --docker-server=XXX.myhuaweicloud.com --docker-username=XXX --docker-password=XXXX -ngitlab
###kubectl create secret docker-registry  aliyun-auth --docker-server=registry.cn-zhangjiakou.aliyuncs.com --docker-username=xxx --docker-password=xxx -ngitlab 
```
##### 其它及遇到的坑

> GitLab集成k8s 签名错误

```
kubectl get secrets
kubectl get secret default-token-52xsf  -o jsonpath="{['data']['ca\.crt']}" | base64 --decode

```

> cat gitlab-admin-service-account.yaml

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitlab
  namespace: gitlab

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gitlab
  namespace: gitlab
subjects:
  - kind: ServiceAccount
    name: gitlab
    namespace: gitlab
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```

> 获取 Service Token

```
 kubectl -n gitlab  describe secret $(kubectl -n gitlab get secret | grep gitlab| awk '{print $1}')
```

