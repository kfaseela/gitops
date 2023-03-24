This repo has some sample manifests used to deploy Istio in different deployment models using ArgoCD

## Install ArgoCD

Refer https://argo-cd.readthedocs.io/en/stable/getting_started/

## Install Istio with multiple control planes in a single cluster with resource isolation

```bash
$ # This ArgoCD manifest will deploy two Istio control planes in system namespaces tenant-1 and tenant-2
$ kubectl apply -f argocd/multitenancy/multiple-cp-controlplane.yaml
$ kubectl get pods -n tenant-1
NAME                               READY   STATUS    RESTARTS   AGE
istiod-tenant-1-5f688886fc-xglp2   1/1     Running   0          9m16s
$
$ kubectl get pods -n tenant-2
NAME                              READY   STATUS    RESTARTS   AGE
istiod-tenant-2-5cc8bb495-thbw6   1/1     Running   0          9m19s
```

## Deploy test applications to verify tenant isolation

```bash
$ # This ArgoCD manifest will deploy three test application namespaces, app-ns-1 under tenant-1 and app-ns-2,app-ns-3 under tenant-2
$ kubectl apply -f argocd/multitenancy/multiple-cp-applications.yaml
$ istioctl ps -i tenant-1
NAME                                  CLUSTER        CDS        LDS        EDS        RDS        ECDS         ISTIOD                               VERSION
httpbin-797587ddc5-729kw.app-ns-1     Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-tenant-1-5f688886fc-xglp2     1.18-alpha.f7393dd36802a2c40b4efe60895213de87badbf7
sleep-78ff5975c6-spgng.app-ns-1       Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-tenant-1-5f688886fc-xglp2     1.18-alpha.f7393dd36802a2c40b4efe60895213de87badbf7
$
$ istioctl ps -i tenant-2
NAME                                  CLUSTER        CDS        LDS        EDS        RDS        ECDS         ISTIOD                              VERSION
httpbin-797587ddc5-cmt65.app-ns-3     Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-tenant-2-5cc8bb495-thbw6     1.18-alpha.f7393dd36802a2c40b4efe60895213de87badbf7
httpbin-797587ddc5-t2kml.app-ns-2     Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-tenant-2-5cc8bb495-thbw6     1.18-alpha.f7393dd36802a2c40b4efe60895213de87badbf7
sleep-78ff5975c6-5xqxc.app-ns-2       Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-tenant-2-5cc8bb495-thbw6     1.18-alpha.f7393dd36802a2c40b4efe60895213de87badbf7
sleep-78ff5975c6-r6pcx.app-ns-3       Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-tenant-2-5cc8bb495-thbw6     1.18-alpha.f7393dd36802a2c40b4efe60895213de87badbf7
$
$ # Confirm app-ns-1 cannot talk to app-ns-2
$ kubectl -n app-ns-1 exec  deploy/sleep -- curl -sIL http://httpbin.app-ns-2.svc.cluster.local:8000
HTTP/1.1 503 Service Unavailable
content-length: 95
content-type: text/plain
date: Fri, 24 Mar 2023 10:40:43 GMT
server: envoy
$
$ # Verify app-ns-2 pods can talk to app-ns-3 pods
$ kubectl -n app-ns-2 exec  deploy/sleep -- curl -sIL http://httpbin.app-ns-3.svc.cluster.local:8000
HTTP/1.1 200 OK
server: envoy
date: Fri, 24 Mar 2023 10:41:41 GMT
content-type: text/html; charset=utf-8
content-length: 9593
access-control-allow-origin: *
access-control-allow-credentials: true
x-envoy-upstream-service-time: 11
```
