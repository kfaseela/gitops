This repo has some sample manifests used to deploy Istio in different deployment models using ArgoCD

## Install ArgoCD

Refer https://argo-cd.readthedocs.io/en/stable/getting_started/

## Install Istio with Multiple ControlPlanes in a single CLuster with resource isolation

{{< text bash >}}
$ # This ArgoCD manifest will deploy two Istio control planes in system namespaces tenant-1 and tenant-2
$ kubectl apply -f argocd/multitenancy/multiple-cp-controlplane.yaml
$ kubectl get pods -n tenant-1
NAME                               READY   STATUS    RESTARTS   AGE
istiod-tenant-1-5f688886fc-xglp2   1/1     Running   0          9m16s
$
$ kubectl get pods -n tenant-2
NAME                              READY   STATUS    RESTARTS   AGE
istiod-tenant-2-5cc8bb495-thbw6   1/1     Running   0          9m19s
{{< /text>}}
