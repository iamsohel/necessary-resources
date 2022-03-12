helm create mychars

helm lint ./mycharts

helm install --dry-run --debug ./my-charts/ --generate-name

helm install exmple ./mycharts/ --set service.type=Nodeport

helm list
helm uninstall exmple
