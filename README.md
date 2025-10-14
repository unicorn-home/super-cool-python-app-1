# python-test-app

## Build the Image
docker build -t super-cool-python-app-1:v1
docker tag super-cool-python-app-1:v1 javamllama/super-cool-python-app-1:v1
docker push

## run the k8s cluster
kind create cluster --config ./kind-cluster-config.yml

## create the ingress controller
Note - From: https://kind.sigs.k8s.io/docs/user/ingress/
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml

## Deploy the app

### Using Kubectl apply
kubectl apply -f k8s/deploy.yml
kubectl apply -f k8s/service.yml
kubectl apply -f k8s/ingress.yml

### Using Helm
helm install super-cool-python-app-1 -n super-cool-python-app-1-dev . --create-namespace

## Test the App
Note: first you need to add 127.0.0.1 super-cool-python-app-1.test.com to the /etc/hosts file
curl super-cool-python-app-1.test.com:80/api/v1/info

## Deploy ArgoCD
helm upgrade --install argocd argo/argo-cd -n argocd --create-namespace -f values.yaml --create-namespace

## Load Argo CD
Note: then add 127.0.0.1 argocd.test.com to the /etc/hosts file
open https://argocd.test.com/ in your browser

## Deploy Github actions runner
From: https://github.com/actions/actions-runner-controller/blob/master/docs/quickstart.md

### Deploy the controller
helm upgrade --install --namespace actions-runner-system --create-namespace\
  --set=authSecret.create=true\
  --set=authSecret.github_token="GITHUB_PAT"\
  --wait actions-runner-controller actions-runner-controller/actions-runner-controller

### Deploy the runner
kubectl apply -n actions-runner-system -f k8s/runnerdeployment.yaml

## Deploy ArgoCD CLI
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

## login to the CLI
argocd login argocd.test.com --insecure --grpc-web --username THE_USER --password THE_PASSWORD

## Sync your app via the CLI
argocd app sync super-cool-python-app-1
