# k8sIngressSetup
It has been a few years since I setup an ingress controller.  The instructions do not appear to have changed that much, but I discover there are some subtle changes. I summarize the setup and verification steps in this repository here.
## Ingress Controller Installation
The [Ingress-Nginx Controller / Installation Guide](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters) is pretty straight forward.  I am using a kubeadm based cluster that I self host on a series of Ubuntu 22.04 VMs. Thus, I follow the bare-metal instructions.
```
jkozik@knode202:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
knode200   Ready    worker          32h   v1.29.1
knode201   Ready    worker          36h   v1.29.1
knode202   Ready    control-plane   37h   v1.29.5
jkozik@knode202:~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created

jkozik@knode202:~/ingress$ kubectl get services -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   37h   <none>

jkozik@knode202:~/ingress$ kubectl get service -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.109.76.78   <none>        80:32340/TCP,443:30989/TCP   5m31s
ingress-nginx-controller-admission   ClusterIP   10.97.79.4     <none>        443/TCP                      5m31s
```
