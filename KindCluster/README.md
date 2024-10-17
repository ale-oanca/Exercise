![alt text](<Screenshot 2024-10-08 at 21.35.39.png>)

I have to create this infrastructure.
I bought a space on contabo for this assignment

# I created a public key in my local machine to access remote machine
    ssh-keygen -b 4096 -> creation
    ls ~/.ssh -> find my public key
# Insert the public key in the remote machine 
    -> .ssh/authorized_keys
    systemctl reload sshd
    I can access now with 

# Create a cluster on a single node

# Install docker ( https://docs.docker.com/engine/install/debian/ )
    Install using apt repository
    Install docker package -> sudo apt-get install docker-ce docker-ce-cli docker-buildx-plugin docker-compose-plugin
    Verify the installation -> sudo docker run hello-world

# Install kind and create cluster( https://kind.sigs.k8s.io/ )
    [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
    chmod +x ./kind
    sudo mv ./kind /usr/local/bin/kind

    kind create cluster

# Install Prometheus and Grafana with prometheus operator ( https://computingforgeeks.com/setup-prometheus-and-grafana-on-kubernetes/ )
    - Clone kube-prometheus project
    - Create monitoring namespace, CustomResourceDefinitions & operator pod
    - Deploy Prometheus Monitoring Stack on Kubernetes
    - Access Grafana dashboards with port-forward 
        - kubectl --namespace monitoring port-forward svc/grafana 3000
        - In my local machine launch -> ssh -L 3000:127.0.0.1:3000 -N root@vmi2194190.contaboserver.net
        - Paste the url http://localhost:3000 to a browser

# Install ArgoCD ( https://medium.com/@iamjagadishgowda/setting-up-argocd-into-minikube-cluster-fe48dc2e3c52 )
    - kubectl create namespace argocd
    - kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    - kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 --decode 
    - Access ArgoCD dashboards with port-forward 
        - kubectl port-forward svc/argocd-server -n argocd 8080:443
        - In my local machine launch -> ssh -L 8080:127.0.0.1:8080 -N root@vmi2194190.contaboserver.net
        - Paste the url http://localhost:8080 to a browser

# I create a projects in Gitlab  
    - The private projects is "Test" 
    - Connected the repo to argocd via token
        link : https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/

# Deploy strimzi operator to install Kafka( https://strimzi.io/quickstarts/ )
    - kubectl create namespace kafka
    - kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
    - kubectl apply -f https://strimzi.io/examples/latest/kafka/kraft/kafka-single-node.yaml -n kafka

    Test Kafka:
    - Run this command: 
        kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.43.0-kafka-3.8.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic
    - Open a different terminal and run this command:
        kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.43.0-kafka-3.8.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning

# Install terraform
    Follow the link to install: https://developer.hashicorp.com/terraform/install#linux

# Topic Kafka
    Saved in /terraform/main.tf
    ......

# Ingress controller
    .....

# Delete cluster
    kind delete cluster

# Troubleshooting
## Grafana
### Can't login
    When you install and delete grafana locally, you may probably have problems when logging in
    To reset the password:
    - find / -type f -name grafana.db
    - Follow the steps at the link: https://stackoverflow.com/questions/71140109/grafana-cannot-login-to-http-localhost3000-login
        cd <PATH>
        sudo sqlite3 grafana.db
            update user set password = '59acf18b94d7eb0694c61e60ce44c110c7a683ac6a8f09580d626f90f4a242000746579358d77dd9e570e83fa24faa88a8a6', salt = 'F3FAxVm33R' where login = 'admin';
            .exit

### Graphana dashboards show no data 
    Edit the panel and remove name=~"^k8s_.*", from all the panels that aren't showing data ( https://github.com/prometheus-operator/kube-prometheus/issues/1695 )-> probably because the filter is associated with clusters installed with kubeadm

## Command top pod/node doesn't work
    Follow this link: https://medium.com/@sainathmitalakar/kubernetes-addons-and-metrics-server-53abc91d76a1

## Change password argocd
    Follow the link: https://github.com/argoproj/argo-cd/blob/master/docs/faq.md#i-forgot-the-admin-password-how-do-i-reset-it

## Terraform installation
    Initially I used kubeadm and this resulted in having files in the system. When installing terraform I encountered problems and had to comment out a url within the kubernetes folder at the path ( /etc/apt/sources.list.d/kubernetes.list ) in order to install the command.
    See issue: https://github.com/kubernetes/release/issues/3564
