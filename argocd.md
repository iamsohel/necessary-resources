# install ArgoCD in k8s

kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# access ArgoCD UI

kubectl get svc -n argocd

kubectl port-forward svc/argocd-server 8080:443 -n argocd

# login with admin user and below token (as in documentation):

 kubectl -n argocd get secret argocd-initial-admin-secret -o yaml
 
 echo  generated_password | base64 --decode




you can change and delete init password




Config repo: https://gitlab.com/nanuchi/argocd-app-config


Docker repo: https://hub.docker.com/repository/docker/nanajanashia/argocd-app


Install ArgoCD: https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd


Login to ArgoCD: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli


ArgoCD Configuration: https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/
