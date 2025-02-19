kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl create secret generic -n build argocd-credentials-secret --from-literal=argocd-user-password='password!@#$' --from-literal=argocd-user-id='admin'

