helm create mychars

helm lint

helm template .

helm template . | kubectl apply -f -

helm install my-charts .

helm list
helm uninstall exmple
