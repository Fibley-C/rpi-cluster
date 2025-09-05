# Bootstrapping ArgoCD

We're going to be using GitOps to manage all aspects of this cluster. To do so we need ArgoCD, which will take over management of all deployments within this cluster. Ideally the end goal here would be to have the complete cluster to be bootstrapable (if that's a word???) from our Git repo https://github.com/Fibley-C/gitops

## Bootstrap ArgoCD via Helm

```
#Add argocd repo
helm repo add argo https://argoproj.github.io/argo-helm

#Install
helm install argocd argo/argo-cd --namespace argocd --create-namespace --set server.service.type=LoadBalancer --version 8.3.0
```

> Just make sure to take note of the **--version** that you install (8.3.0 here but may be a later version available at the time of following this) as you will need to reference this in the argocd application manifest argocd uses to manage itself when setting up the GItOps workflow.

Verify ArgoCD has been deployed with

`kubectl get pods -n argocd`

You should see

```
root@k3s-master:~/gitops/apps# kubectl get pods -n argocd
NAME                                               READY   STATUS    RESTARTS      AGE
argocd-application-controller-0                    1/1     Running   1 (24h ago)   35h
argocd-applicationset-controller-59f7fc666-q2kbp   1/1     Running   1 (24h ago)   35h
argocd-dex-server-8b99cf66f-568pf                  1/1     Running   1 (24h ago)   35h
argocd-notifications-controller-b7bf48757-n7c52    1/1     Running   1 (24h ago)   35h
argocd-redis-84897889d6-gghrm                      1/1     Running   1 (24h ago)   35h
argocd-repo-server-77654bf79c-c5mtk                1/1     Running   1 (24h ago)   35h
argocd-server-66bc5bd49b-z24h9                     1/1     Running   1 (24h ago)   35h
```

Assuming you've already configured metallb from `/metallb` you should be able to view the loadbalancer service from

```
root@k3s-master:~/gitops/apps# kubectl get svc -n argocd
NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
argocd-applicationset-controller   ClusterIP      10.43.229.196   <none>         7000/TCP                     35h
argocd-dex-server                  ClusterIP      10.43.219.103   <none>         5556/TCP,5557/TCP            35h
argocd-redis                       ClusterIP      10.43.181.150   <none>         6379/TCP                     35h
argocd-repo-server                 ClusterIP      10.43.162.171   <none>         8081/TCP                     35h
argocd-server                      LoadBalancer   10.43.200.232   192.168.1.42   80:30529/TCP,443:31289/TCP   35h
```

Your looking for the result with the `LoadBalancer` type, you can access the ArgoCD UI by going to this address in your browser
