#### argocd-with-kind-deployments
This is an minimalist approach towards achieving all types of deployment using argocd on kind cluster. 

```
➜  argocd-with-kind-deployments git:(main) kind get clusters
No kind clusters found.
➜  argocd-with-kind-deployments git:(main) 
```

```
➜  argocd-with-kind-deployments git:(main) kind create cluster --name argocd --config kind-config.yaml
Creating cluster "argocd" ...
 ✓ Ensuring node image (kindest/node:v1.30.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-argocd"
You can now use your cluster with:

kubectl cluster-info --context kind-argocd

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```



```
➜  argocd-with-kind-deployments git:(main) ✗ helm install argocd argo/argo-cd --namespace argocd --create-namespace

NAME: argocd
LAST DEPLOYED: Tue Aug 26 03:08:37 2025
NAMESPACE: argocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argocd-server -n argocd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts
```

After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:


```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)
➜  argocd-with-kind-deployments git:(main) ✗ 
➜  argocd-with-kind-deployments git:(main) ✗ kubectl get pods -A
NAMESPACE            NAME                                                READY   STATUS      RESTARTS   AGE
argocd               argocd-application-controller-0                     1/1     Running     0          47s
argocd               argocd-applicationset-controller-695db44dc7-f5pgf   1/1     Running     0          47s
argocd               argocd-dex-server-75bf447bb6-9tw9z                  1/1     Running     0          47s
argocd               argocd-notifications-controller-d99578cd-65655      1/1     Running     0          47s
argocd               argocd-redis-7df596bff-fvsxd                        1/1     Running     0          47s
argocd               argocd-redis-secret-init-745p9                      0/1     Completed   0          59s
argocd               argocd-repo-server-9b46fd666-lb2xl                  1/1     Running     0          47s
argocd               argocd-server-58fbf8bfb7-knzsc                      1/1     Running     0          47s
kube-system          coredns-7db6d8ff4d-4fsh9                            1/1     Running     0          2m22s
kube-system          coredns-7db6d8ff4d-tszcx                            1/1     Running     0          2m22s
kube-system          etcd-argocd-control-plane                           1/1     Running     0          2m37s
kube-system          kindnet-8fggw                                       1/1     Running     0          2m22s
kube-system          kube-apiserver-argocd-control-plane                 1/1     Running     0          2m37s
kube-system          kube-controller-manager-argocd-control-plane        1/1     Running     0          2m37s
kube-system          kube-proxy-cf6sn                                    1/1     Running     0          2m22s
kube-system          kube-scheduler-argocd-control-plane                 1/1     Running     0          2m37s
local-path-storage   local-path-provisioner-988d74bc-jstt9               1/1     Running     0          2m22s
```

and then check the services of argocd 

```
➜  argocd-with-kind-deployments git:(main) kubectl get svc -n argocd
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
argocd-applicationset-controller   ClusterIP   10.96.89.125    <none>        7000/TCP            19m
argocd-dex-server                  ClusterIP   10.96.31.153    <none>        5556/TCP,5557/TCP   19m
argocd-redis                       ClusterIP   10.96.86.246    <none>        6379/TCP            19m
argocd-repo-server                 ClusterIP   10.96.150.100   <none>        8081/TCP            19m
argocd-server                      ClusterIP   10.96.32.50     <none>        80/TCP,443/TCP      19m
```

By default, argocd-server is deployed in ClusterIP type. now, change that to NodePort and also make sure the TCP ports are same as initially configured ports in the kind-config yaml file. So, we can access it via Browser.


```
➜  argocd-with-kind-deployments git:(main) ✗ kubectl get svc -n argocd               
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller   ClusterIP   10.96.129.15    <none>        7000/TCP                     5m50s
argocd-dex-server                  ClusterIP   10.96.158.39    <none>        5556/TCP,5557/TCP            5m50s
argocd-redis                       ClusterIP   10.96.18.89     <none>        6379/TCP                     5m50s
argocd-repo-server                 ClusterIP   10.96.248.169   <none>        8081/TCP                     5m50s
argocd-server                      NodePort    10.96.59.38     <none>        80:31594/TCP,443:31698/TCP   5m50s


➜  argocd-with-kind-deployments git:(main) ✗ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
iZwdP-BfT40iGoTC

```


Argocd UI is accessible now.

Lets, deploy all types of ArgoCD deployment strategies one after another, the main deployment types to focus on include:

1.Basic Application Deployment (Manual or Automatic Sync)

2.Rolling Update Deployment

3.Blue-Green Deployment

4.Canary Deployment

5.A/B Testing Deployment