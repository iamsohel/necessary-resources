helm create mychars == create new charts

helm lint

helm template .   == check helm charts syntax 

helm template . | kubectl apply -f -

helm install my-charts .

helm install my-charts2 . -f newValye.yml [for multiple values file]

helm list

helm uninstall exmple
