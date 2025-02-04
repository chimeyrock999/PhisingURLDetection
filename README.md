# PhisingURLDetection
This is a Web application that allow you to detect phising urls with data crawled from <https://phishtank.org/> on daily basis and a Random Forest Classifier retrained weekly.

## Prerequites
- Kubernetes: You can choose PaaS services from any Cloud Providers
- Helm: Follow this document to intall Helm <https://helm.sh/docs/intro/install/>

## Components:
- Web service: the UI that allow user to predict phising url.
- Airflow: Workflow Oschestrator to manage daily crawl data task and weekly retrain model task.
- Spark: Compute Engine to extract useful features form urls, train classifier model and predict phising url.
- ArgoCD: GitOps tool to deploy Airflow, Spark with Helm Chart and CI/CD pipeline.
## Installation
### ArgoCD
1. To install ArgoCD, we use offical Helm Chart:
```bash
kubectl create namespace argocd
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd -n argocd
```
2. To use ArgoCD UI, please run command:
```bash
kubectl port-forward service/argocd-server -n argocd 8080:443
```
3. Open the browser on <http://localhost:8080/> and accept the certificate. Login with username `admin` and password is the result of command:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
4. Update ArgoCD password via UI: User Info > Update Password.
5. Install `argocd-cli` by following document: <https://argo-cd.readthedocs.io/en/stable/cli_installation/>
6. Login AgroCD via `argocd-cli` with username `admin` and your new password.
```
argocd login localhost:8080
```
### Ingress Controller and Cert Manager'
To expose K8S services through domain and use Let's Enscrypt certificate, please install `ingress-nginx` and `cert-manager`.
1. Add Helm repositories to ArgoCD:
```bash
argocd repo add
```

## CI/CD Pipeline
1. I've use GitHub persional access token to setup CI/CD pipeline with 
self-host runner deployed by using Helm Chart and ArgoCD (check `mainfest/argocd/application/gha-runner-scale-set-controller` and `mainfest/argocd/application/gha-runner-scale-set` for more detail).

To perform the authentication, follow the [document](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/authenticating-to-the-github-api#authenticating-arc-with-a-personal-access-token).

2. In the `values.yaml` of `gha-runner-scale-set`, there is a configuration `githubConfigSecret: github-token-secret`. Because of security problems, I don't want to use GitOps Pipeline for this `Secret`.
So I create a `github-token-secret`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: github-token-secret
  namespace: arc-runners
data:
  github_token: <YOUR_BASE64_ENCODED_TOKEN>
type: Opaque
```
And then use command to create the `Secret`:
```bash
kubectl apply -f github-token-secret.yaml
``` 

Or you can use a command line bellow:
```bash
kubectl create secret generic github-token-secret \
    --namespace=arc-runners \
   --from-literal=github_token='<YOUR_TOKEN>'
```
3. Because I use ArgoCD for continous delivery, so I have to install 
## Authors
* Trinh Van Thoai-chimeyrock999
