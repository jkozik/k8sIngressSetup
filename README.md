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
## Ingress Controller Verification
### Install test nginx server -- deployment, service, ingress
```
jkozik@knode202:~$ git clone https://github.com/jkozik/k8sIngressSetup.git
Cloning into 'k8sIngressSetup'...
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 11 (delta 0), reused 5 (delta 0), pack-reused 0
Receiving objects: 100% (11/11), done.
jkozik@knode202:~$ cd k8sIngressSetup/
jkozik@knode202:~/k8sIngressSetup$ kubectl apply -f .
deployment.apps/nginx-deployment created
ingress.networking.k8s.io/nginx-ingress created
service/nginx-svc created
jkozik@knode202:~/k8sIngressSetup$ kubectl get pods,ingress,svc -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
pod/nginx-deployment-5bb85d69d8-77bkb   1/1     Running   0          14m   10.10.101.69   knode201   <none>           <none>
pod/nginx-deployment-5bb85d69d8-gm5wd   1/1     Running   0          14m   10.10.12.72    knode200   <none>           <none>

NAME                                      CLASS   HOSTS           ADDRESS           PORTS   AGE
ingress.networking.k8s.io/nginx-ingress   nginx   k8s.kozik.net   192.168.100.200   80      14m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        44h   <none>
service/nginx-svc    NodePort    10.107.119.236   <none>        80:31125/TCP   14m   app=nginx
jkozik@knode202:~/k8sIngressSetup$
```
### Verify nginx server can be accessed
While running on the master node, one can access the service directly
```
jkozik@knode202:~/k8sIngressSetup$ kubectl get pods,ingress,svc -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
pod/nginx-deployment-5bb85d69d8-77bkb   1/1     Running   0          14m   10.10.101.69   knode201   <none>           <none>
pod/nginx-deployment-5bb85d69d8-gm5wd   1/1     Running   0          14m   10.10.12.72    knode200   <none>           <none>

NAME                                      CLASS   HOSTS           ADDRESS           PORTS   AGE
ingress.networking.k8s.io/nginx-ingress   nginx   k8s.kozik.net   192.168.100.200   80      14m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        44h   <none>
service/nginx-svc    NodePort    10.107.119.236   <none>        80:31125/TCP   14m   app=nginx
jkozik@knode202:~/k8sIngressSetup$ curl 10.107.119.236
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
Or, anywhere on the home LAN, one can access the nginx server through the ingress controller
```
jkozik@knode202:~/k8sIngressSetup$ curl -H "Host: k8s.kozik.net" 192.168.100.200:31125
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
jkozik@knode202:~/k8sIngressSetup$
```
### Through home LAN reverse proxy
I have a home LAN that has a mapping from 'k8s.kozik.net' on my external internet IP address to the kubernetes cluster --> https://192.168.100.200:31125
![Screenshot 2024-05-19 154423](https://github.com/jkozik/k8sIngressSetup/assets/1474681/8d8cc90d-49ea-4acf-a6dd-56f6a793831f)

