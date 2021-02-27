Install kops
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64 sudo mv kops-linux-amd64 /usr/local/bin/kops

Install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl chmod +x ./kubectl sudo mv ./kubectl /usr/local/bin/kubectl kubectl version --client

Export bucket from s3
export bucket_name=belajarlinux-states-stores
#create nama cluster
export KOPS_CLUSTER_NAME=kube.belajarlinux.web.id
export KOPS_STATE_STORE=s3://${bucket_name}
  
Create cluster , dont forget generate ssh key
kops create cluster --zones=ap-southeast-1a --node-count=2 --master-count=1 --node-size=t2.micro --master-size=t2.medium --name=${KOPS_CLUSTER_NAME} --ssh-public-key=~/.ssh/id_rsa.pub

Update cluster
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes --admin

Validate Cluster
kops validate cluster

Install Ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/mandatory.yaml 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/provider/aws/service-l4.yaml 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/provider/aws/patch-configmap-l4.yaml

git clone https://github.com/kubernetes/ingress-nginx cd ingress-nginx/deploy/static/provider/aws/ kubectl apply -f deploy.yaml

Create namespace
kubectl create namespace staging kubectl create namespace production

Install HELM for Certificate Authority
snap install helm --classic ln -s /snap/bin/helm /usr/local/bin/helm kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.3/cert-manager.crds.yaml kubectl create namespace cert-manager helm repo add jetstack https://charts.jetstack.io helm repo update helm --version helm install cert-manager --namespace cert-manager --version v0.14.3 jetstack/cert-manager kubectl get pods --namespace cert-manager

Deploy Landing Page
kubectl create namespace landing-page-production
kubectl apply -f landingpage.yml
kubectl apply -f cert.yml

Deploy Pesbuk
kubectl create namespace pesbuk-production
kubectl apply -f nginx.yml
kubectl apply -f cert.yml
kubectl apply -f secret.yml

Deploy Wordpress
kubectl create namespace wp-staging
kubectl apply -f deployment.yml

