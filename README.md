# How to run
1. Install EKS cluster

```
eksctl create cluster eks-cluster.yaml
```
2. Take a tour
```
kubectl get nodes
kubectl get namespaces
kubectl get pods -A
kubectl get svc -A
```

3. Install a sampleapp ( of service type LoadBalancer )
```
cd sampleapp
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

kubectl get pods -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
default       sampleapp-ff9ffdfd-wp9rv   1/1     Running   0          9m14s
kube-system   aws-node-gbn47             1/1     Running   0          20m
kube-system   aws-node-v5nmr             1/1     Running   0          20m
kube-system   aws-node-xt7rj             1/1     Running   0          20m
kube-system   coredns-55bd85dd5d-fnstn   1/1     Running   0          31m
kube-system   coredns-55bd85dd5d-xvtpg   1/1     Running   0          31m
kube-system   kube-proxy-78k6z           1/1     Running   0          20m
kube-system   kube-proxy-mztqb           1/1     Running   0          20m
kube-system   kube-proxy-w7z4j           1/1     Running   0          20m

kubectl get svc -A
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)        AGE
kubernetes      ClusterIP      10.100.0.1      <none>                                                                   443/TCP        23m
sampleapp-svc   LoadBalancer   10.100.123.96   a17114cf8f43446e7a273783c223bcfe-986239722.eu-west-2.elb.amazonaws.com   80:32053/TCP   7s
```
4. Access sampleapp

Paste the EXTERNAL-URL in the browser. ( You should see Welcome to nginx! )
a17114cf8f43446e7a273783c223bcfe-986239722.eu-west-2.elb.amazonaws.com

As you access the sampleapp, you should see the following in the logs of the sampleapp pod
```
kubectl logs sampleapp-ff9ffdfd-wp9rv --follow
192.168.42.62 - - [17/Feb/2023:08:18:03 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
192.168.42.62 - - [17/Feb/2023:08:18:07 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
192.168.42.62 - - [17/Feb/2023:08:18:07 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
```

5. Modify home page of sampleapp
```
kubectl exec -it sampleapp-7f7b9b9d9d-4j2xg -- /bin/bash
apt update && apt install nano -y
nano /usr/share/nginx/html/index.html # and amend the content
# or 
sed -i s/nginx/sampleapp/g /usr/share/nginx/html/index.html
```
Access EXTERNAL-URL again. You should see the new content.

6. Install ingress controller
```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install my-release nginx-stable/nginx-ingress
```
7. Take a tour
```
kubectl get svc -A
NAMESPACE     NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                      AGE
default       kubernetes                 ClusterIP      10.100.0.1       <none>                                                                   443/TCP                      42m
default       my-release-nginx-ingress   LoadBalancer   10.100.18.108    a8cb8806fc7fe4028bf0103f38287122-755455926.eu-west-2.elb.amazonaws.com   80:30442/TCP,443:31236/TCP   10m
default       sampleapp-svc              LoadBalancer   10.100.123.96    a17114cf8f43446e7a273783c223bcfe-986239722.eu-west-2.elb.amazonaws.com   80:32053/TCP                 19m
kube-system   kube-dns                   ClusterIP      10.100.0.10      <none>                                                                   53/UDP,53/TCP                42m

k get pods -A
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
default       my-release-nginx-ingress-684c4589b8-ndw5z   1/1     Running   0          10m
default       sampleapp-ff9ffdfd-wp9rv                    1/1     Running   0          20m
kube-system   aws-node-gbn47                              1/1     Running   0          32m
kube-system   aws-node-v5nmr                              1/1     Running   0          32m
kube-system   aws-node-xt7rj                              1/1     Running   0          32m
kube-system   coredns-55bd85dd5d-fnstn                    1/1     Running   0          42m
kube-system   coredns-55bd85dd5d-xvtpg                    1/1     Running   0          42m
kube-system   kube-proxy-78k6z                            1/1     Running   0          32m
kube-system   kube-proxy-mztqb                            1/1     Running   0          32m
kube-system   kube-proxy-w7z4j                            1/1     Running   0          32m
```
8. Access External-URL of ingress controller and follow logs of ingress controller
```
kubectl logs my-release-nginx-ingress-684c4589b8-ndw5z --follow
192.168.59.3 - - [17/Feb/2023:08:25:26 +0000] "GET / HTTP/1.1" 404 555 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
192.168.59.3 - - [17/Feb/2023:08:25:26 +0000] "GET / HTTP/1.1" 404 555 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
192.168.59.3 - - [17/Feb/2023:08:25:27 +0000] "GET / HTTP/1.1" 404 555 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
```
Paste the EXTERNAL-URL in the browser. ( You should see 404! )

9. Explore ingress controller
```
kubectl exec -it my-release-nginx-ingress-684c4589b8-ndw5z -- /bin/bash
cd /etc/nginx/conf.d
ls
cat /etc/nginx/nginx.conf
```
10. Install second app (studio app) in a new namespace (service type left blank, therefore ClusterIP)
```
cd studio
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods -A
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
default       my-release-nginx-ingress-684c4589b8-ndw5z   1/1     Running   0          10m
default       sampleapp-ff9ffdfd-wp9rv                    1/1     Running   0          20m
kube-system   aws-node-gbn47                              1/1     Running   0          32m
kube-system   aws-node-v5nmr                              1/1     Running   0          32m
kube-system   aws-node-xt7rj                              1/1     Running   0          32m
kube-system   coredns-55bd85dd5d-fnstn                    1/1     Running   0          42m
kube-system   coredns-55bd85dd5d-xvtpg                    1/1     Running   0          42m
kube-system   kube-proxy-78k6z                            1/1     Running   0          32m
kube-system   kube-proxy-mztqb                            1/1     Running   0          32m
kube-system   kube-proxy-w7z4j                            1/1     Running   0          32m
studio        studio-86866cd6c6-hc6rw                     1/1     Running   0          7m47s

kubectl get svc -A
NAMESPACE     NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                      AGE
default       kubernetes                 ClusterIP      10.100.0.1       <none>                                                                   443/TCP                      60m
default       my-release-nginx-ingress   LoadBalancer   10.100.18.108    a8cb8806fc7fe4028bf0103f38287122-755455926.eu-west-2.elb.amazonaws.com   80:30442/TCP,443:31236/TCP   27m
default       sampleapp-svc              LoadBalancer   10.100.123.96    a17114cf8f43446e7a273783c223bcfe-986239722.eu-west-2.elb.amazonaws.com   80:32053/TCP                 36m
kube-system   kube-dns                   ClusterIP      10.100.0.10      <none>                                                                   53/UDP,53/TCP                59m
studio        studio-svc                 ClusterIP      10.100.103.208   <none>                                                                   80/TCP                       24m

# Amend home page of studio app
kubectl exec -it studio-86866cd6c6-hc6rw -- /bin/bash
cd /usr/share/nginx/html
sed -i 's/nginx/studio/g' index.html
```
11. Ensure studio app is accessible from inside the cluster
```
kubectl exec -it sampleapp-ff9ffdfd-wp9rv -- /bin/bash
curl 10.100.103.208
```

12. Create ingress rule for studio app
```
cd studio
kubectl apply -f ingress.yaml
kubectl get ingress -A
NAMESPACE   NAME             CLASS   HOSTS                        ADDRESS                                                                  PORTS   AGE
studio      studio-ingress   nginx   studio.myborlingstudio.com   a8cb8806fc7fe4028bf0103f38287122-755455926.eu-west-2.elb.amazonaws.com   80      42m
```
I am using a random domain name here. You can use your own domain name.

13. Ensure nginx ingress controller has picked up your ingress rule
```
kubectl logs my-release-nginx-ingress-684c4589b8-ndw5z --follow
I0217 07:59:01.911763       1 event.go:285] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"studio", Name:"studio-ingress", UID:"506cbf1d-074d-4f48-b0be-4a80708c1101", APIVersion:"networking.k8s.io/v1", ResourceVersion:"9213", FieldPath:""}): type: 'Normal' reason: 'AddedOrUpdated' Configuration for studio/studio-ingress was added or updated

kubectl exec -it my-release-nginx-ingress-684c4589b8-ndw5z -- /bin/bash
cd /etc/nginx/conf.d
ls
cat studio.myborlingstudio.com.conf
```
14. Get the Public IP address of the ingress controller
```
ping a8cb8806fc7fe4028bf0103f38287122-755455926.eu-west-2.elb.amazonaws.com
PING a8cb8806fc7fe4028bf0103f38287122-755455926.eu-west-2.elb.amazonaws.com (18.169.246.254) 56(84) bytes of data
```
15. Add an entry in your local hosts file
```
18.169.246.254 studio.myborlingstudio.com
```
16. Access the studio app from your browser by typing studio.myborlingstudio.com
Follow the logs of the ingress controller

17. Delete the cluster
```
eksctl delete cluster --name my-eks-cluster
```