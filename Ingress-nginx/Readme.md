# Ingress NGINX
#
# Kubernetes
#
kind create cluster

#
# MetalLB
#
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml

#
# MetalLB Wait
#
kubectl wait \
        --namespace metallb-system \
        --for=condition=ready pod \
        --selector=app=metallb \
        --timeout=90s

#
# MetalLB config
#
docker network inspect -f '{{.IPAM.Config}}' kind
cat metallb-L2-IPpool.yaml
kubectl apply -f metallb-L2-IPpool.yaml

#
# HELM doc ingress-nginx
#
https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx?modal=install

#
# HELM install ingress-nginx
#
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install my-ingress-nginx ingress-nginx/ingress-nginx

kubectl wait \
        --namespace default \
        --for=condition=ready pod \
        --selector=app.kubernetes.io/name=ingress-nginx \
        --timeout=90s

#
# Deployment NGINX web server
#
kubectl apply -f deployment-nginx.yaml
kubectl apply -f deployment-http-echo.yaml
kubectl apply -f deployment-whoami.yaml
kubectl apply -f deployment-fake-service.yaml

#
# SVC for NGINX deployment
#
kubectl apply -f service-svc-nginx-clusterip.yaml
kubectl apply -f service-svc-http-echo-clusterip.yaml
kubectl apply -f service-svc-whoami-clusterip.yaml
kubectl apply -f service-svc-fake-service-clusterip.yaml

#
# TEST SVC NGINX with port-forward to service
#
kubectl port-forward services/svc-nginx 8080:8080
kubectl port-forward services/svc-http-echo 8080:8080
kubectl port-forward services/svc-whoami 8080:8080
kubectl port-forward services/svc-fake-service 8080:8080
curl localhost:8080
open -a "Google Chrome" http://127.0.0.1:8080/dashboard/

#
# Ingress to svc-nginx
#
kubectl apply -f ingress.yaml
kubectl get ingress

#
# TEST
#
docker container run -it --rm --network=kind busybox wget --header "Host: nginx.example.local" -qO- http://172.18.255.200:80
docker container run -it --rm --network=kind busybox wget --header "Host: http-echo.example.local" -qO- http://172.18.255.200:80
docker container run -it --rm --network=kind busybox wget --header "Host: whoami.example.local" -qO- http://172.18.255.200:80
docker container run -it --rm --network=kind busybox wget --header "Host: fake-service.example.local" -qO- http://172.18.255.200:80
docker container run -it --rm --network=kind busybox wget -qO- http://172.18.255.200:80 # expected because no route

