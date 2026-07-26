# Minikube Test Framework

> This project explores test automation frameworks that can be deployed on a Kubernetes cluster (Minikube) that can be executed on hardware, software, infrastructure, and security components.

## Overview
Two products will be explored.

1. [Tekton][tekton]
2. [TestKube][testkube]

## Minikube Cluster

$ minikube image ls 
```
registry.k8s.io/pause:3.10.1
registry.k8s.io/kube-scheduler:v1.35.1
registry.k8s.io/kube-proxy:v1.35.1
registry.k8s.io/kube-controller-manager:v1.35.1
registry.k8s.io/kube-apiserver:v1.35.1
registry.k8s.io/etcd:3.6.6-0
registry.k8s.io/coredns/coredns:v1.13.1
gcr.io/k8s-minikube/storage-provisioner:v5
```

## Tekton

### Install Tekton Pipelines

$ kubectl apply --filename https://infra.tekton.dev/tekton-releases/pipeline/latest/release.yaml

$ kubectl get pods -A 
```
NAMESPACE                    NAME                                                READY   STATUS    RESTARTS      AGE
kube-system                  coredns-7d764666f9-jrdtg                            1/1     Running   0             31m
kube-system                  etcd-minikube                                       1/1     Running   0             31m
kube-system                  kube-apiserver-minikube                             1/1     Running   0             31m
kube-system                  kube-controller-manager-minikube                    1/1     Running   0             31m
kube-system                  kube-proxy-vrxc7                                    1/1     Running   0             31m
kube-system                  kube-scheduler-minikube                             1/1     Running   0             31m
kube-system                  storage-provisioner                                 1/1     Running   1 (30m ago)   31m
tekton-pipelines-resolvers   tekton-pipelines-remote-resolvers-6945d8fcc-7lwcl   1/1     Running   0             25s
tekton-pipelines             tekton-events-controller-5c9b9758fb-xgsg8           1/1     Running   0             26s
tekton-pipelines             tekton-pipelines-controller-6b99c97b7-lzd6k         1/1     Running   0             26s
tekton-pipelines             tekton-pipelines-webhook-669d89b9b4-j4wfv           1/1     Running   0             25s
```

$ minikube image ls
```
registry.k8s.io/pause:3.10.1
registry.k8s.io/kube-scheduler:v1.35.1
registry.k8s.io/kube-proxy:v1.35.1
registry.k8s.io/kube-controller-manager:v1.35.1
registry.k8s.io/kube-apiserver:v1.35.1
registry.k8s.io/etcd:3.6.6-0
registry.k8s.io/coredns/coredns:v1.13.1
ghcr.io/tektoncd/pipeline/webhook-d4749e605405422fd87700164e31b2d1:<none>
ghcr.io/tektoncd/pipeline/resolvers-ff86b24f130c42b88983d3c13993056d:<none>
ghcr.io/tektoncd/pipeline/events-a9042f7efb0cbade2a868a1ee5ddd52c:<none>
ghcr.io/tektoncd/pipeline/controller-10a3e32792f33651396d02b6855a6e36:<none>
gcr.io/k8s-minikube/storage-provisioner:v5
```

### Install and Set Up Tekton Triggers

$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml

$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml

$ kubectl get pods -A
```
NAMESPACE                    NAME                                                 READY   STATUS    RESTARTS      AGE
kube-system                  coredns-7d764666f9-jrdtg                             1/1     Running   0             33m
kube-system                  etcd-minikube                                        1/1     Running   0             34m
kube-system                  kube-apiserver-minikube                              1/1     Running   0             34m
kube-system                  kube-controller-manager-minikube                     1/1     Running   0             34m
kube-system                  kube-proxy-vrxc7                                     1/1     Running   0             33m
kube-system                  kube-scheduler-minikube                              1/1     Running   0             34m
kube-system                  storage-provisioner                                  1/1     Running   1 (33m ago)   34m
tekton-pipelines-resolvers   tekton-pipelines-remote-resolvers-6945d8fcc-7lwcl    1/1     Running   0             2m59s
tekton-pipelines             tekton-events-controller-5c9b9758fb-xgsg8            1/1     Running   0             3m
tekton-pipelines             tekton-pipelines-controller-6b99c97b7-lzd6k          1/1     Running   0             3m
tekton-pipelines             tekton-pipelines-webhook-669d89b9b4-j4wfv            1/1     Running   0             2m59s
tekton-pipelines             tekton-triggers-controller-66fd74568d-xpgns          1/1     Running   0             35s
tekton-pipelines             tekton-triggers-core-interceptors-66456f8cf6-ql5wb   1/1     Running   0             33s
tekton-pipelines             tekton-triggers-webhook-55c8dd895f-4982w             1/1     Running   0             35s
```

$ minikube image ls
```
registry.k8s.io/pause:3.10.1
registry.k8s.io/kube-scheduler:v1.35.1
registry.k8s.io/kube-proxy:v1.35.1
registry.k8s.io/kube-controller-manager:v1.35.1
registry.k8s.io/kube-apiserver:v1.35.1
registry.k8s.io/etcd:3.6.6-0
registry.k8s.io/coredns/coredns:v1.13.1
ghcr.io/tektoncd/triggers/webhook-dd1edc925ee1772a9f76e2c1bc291ef6:<none>
ghcr.io/tektoncd/triggers/interceptors-3176d6a3f314c3655b30bfd36e421dd5:<none>
ghcr.io/tektoncd/triggers/controller-f656ca31de179ab913fa76abc255c315:<none>
ghcr.io/tektoncd/pipeline/webhook-d4749e605405422fd87700164e31b2d1:<none>
ghcr.io/tektoncd/pipeline/resolvers-ff86b24f130c42b88983d3c13993056d:<none>
ghcr.io/tektoncd/pipeline/events-a9042f7efb0cbade2a868a1ee5ddd52c:<none>
ghcr.io/tektoncd/pipeline/controller-10a3e32792f33651396d02b6855a6e36:<none>
gcr.io/k8s-minikube/storage-provisioner:v5
```

### Installing Tekton Dashboard

$ kubectl apply --filename https://infra.tekton.dev/tekton-releases/dashboard/latest/release.yaml

$ kubectl get pods -A
```
NAMESPACE                    NAME                                                 READY   STATUS    RESTARTS      AGE
kube-system                  coredns-7d764666f9-jrdtg                             1/1     Running   0             36m
kube-system                  etcd-minikube                                        1/1     Running   0             36m
kube-system                  kube-apiserver-minikube                              1/1     Running   0             36m
kube-system                  kube-controller-manager-minikube                     1/1     Running   0             36m
kube-system                  kube-proxy-vrxc7                                     1/1     Running   0             36m
kube-system                  kube-scheduler-minikube                              1/1     Running   0             36m
kube-system                  storage-provisioner                                  1/1     Running   1 (35m ago)   36m
tekton-pipelines-resolvers   tekton-pipelines-remote-resolvers-6945d8fcc-7lwcl    1/1     Running   0             5m9s
tekton-pipelines             tekton-dashboard-65976559b9-n9lkm                    1/1     Running   0             9s
tekton-pipelines             tekton-events-controller-5c9b9758fb-xgsg8            1/1     Running   0             5m10s
tekton-pipelines             tekton-pipelines-controller-6b99c97b7-lzd6k          1/1     Running   0             5m10s
tekton-pipelines             tekton-pipelines-webhook-669d89b9b4-j4wfv            1/1     Running   0             5m9s
tekton-pipelines             tekton-triggers-controller-66fd74568d-xpgns          1/1     Running   0             2m45s
tekton-pipelines             tekton-triggers-core-interceptors-66456f8cf6-ql5wb   1/1     Running   0             2m43s
tekton-pipelines             tekton-triggers-webhook-55c8dd895f-4982w             1/1     Running   0             2m45s
```

$ minikube image ls
```
registry.k8s.io/pause:3.10.1
registry.k8s.io/kube-scheduler:v1.35.1
registry.k8s.io/kube-proxy:v1.35.1
registry.k8s.io/kube-controller-manager:v1.35.1
registry.k8s.io/kube-apiserver:v1.35.1
registry.k8s.io/etcd:3.6.6-0
registry.k8s.io/coredns/coredns:v1.13.1
ghcr.io/tektoncd/triggers/webhook-dd1edc925ee1772a9f76e2c1bc291ef6:<none>
ghcr.io/tektoncd/triggers/interceptors-3176d6a3f314c3655b30bfd36e421dd5:<none>
ghcr.io/tektoncd/triggers/controller-f656ca31de179ab913fa76abc255c315:<none>
ghcr.io/tektoncd/pipeline/webhook-d4749e605405422fd87700164e31b2d1:<none>
ghcr.io/tektoncd/pipeline/resolvers-ff86b24f130c42b88983d3c13993056d:<none>
ghcr.io/tektoncd/pipeline/events-a9042f7efb0cbade2a868a1ee5ddd52c:<none>
ghcr.io/tektoncd/pipeline/controller-10a3e32792f33651396d02b6855a6e36:<none>
ghcr.io/tektoncd/dashboard/dashboard-9623576a202fe86c8b7d1bc489905f86:<none>
gcr.io/k8s-minikube/storage-provisioner:v5
```

#### Using kubectl proxy
$ kubectl proxy

Browse http://localhost:8001/api/v1/namespaces/tekton-pipelines/services/tekton-dashboard:http/proxy/ to access your Dashboard.

#### Using kubectl port-forward
$ kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 9097:9097

Browse http://localhost:9097 to access your Dashboard.

[//]: Links

[tekton]: https://tekton.dev/docs/
[tekton-helm]: https://github.com/cdfoundation/tekton-helm-chart
[testkube]: https://docs.testkube.io/articles/open-source