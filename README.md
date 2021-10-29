# edx-platform-gha-ci

- create the cluster
  ```
  eksctl create cluster -f cluster.yaml
  ```
- install cert-manager
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml
```
- create the namespace

```
kubectl create namespace actions-runner-system
```

- create a secret named controller-manager that has `github_app_id`, `github_app_installation_id` and `github_app_private_key` or we can specify these in values.yaml under authSecret, for actions-controller helm chart
```
kubectl apply -f app-secrets.yaml
```
or
```
kubectl create secret generic controller-manager -n actions-runner-system --from-literal=github_app_id=“GITHUB_APP_ID” --from-literal=github_app_installation_id=“GITHUB_APP_INSTALLATION_ID" --from-file=github_app_private_key=“GITHUB_APP_PRIVATE_KEY.private-key.pem”
```

- we need to install the actions-runner-controller via their helm chart since that has support for github webhook server and pod autoscaling bases on github events we've to make sure controller-manager secret already exists or has been added to values.yaml so that secret gets craeted before installing the actions-controller helm chart

```
helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller  
helm install -f values.yaml --wait --namespace actions-runner-system actions-runner-controller actions-runner-controller/actions-runner-controller
```

- and finally deploy the runner deployment
```
kubectl apply -f runner-deployment.yaml
```
