helm create mychars == create new charts

helm lint

helm template .   == check helm charts syntax 

helm template . | kubectl apply -f -

helm install my-charts .

helm install my-charts2 . -f newValye.yml [for multiple values file]

helm list

helm uninstall exmple

kubectl delete all --all --namespace default

![image](https://github.com/iamsohel/necessary-resources/assets/9135426/54ef7610-b111-4bf0-ab9f-ac0581712895)
![image](https://github.com/iamsohel/necessary-resources/assets/9135426/6ddd3460-f915-4e8c-a61b-d0a54d5f7596)

Monitor

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm install prometheus prometheus-community/prometheus

helm show values prometheus-community/kube-prometheus-stack
helm show values prometheus-community/kube-prometheus-stack > values.yaml 

https://bitnami.com/stack/prometheus/helm

kubectl edit svc prometheus-server 

https://bitnami.com/stack/grafana/helm

dashboard id: 3119
pod overview dashboard: 6781
https://devops4solutions.com/monitor-kubernetes-cluster-using-prometheus-and-grafana/
http://localhost:9090/metrics

kubectl get secret grafana-admin --namespace default -o jsonpath="{.data.GF_SECURITY_ADMIN_PASSWORD}" | base64 -d)
```
echo "User: admin"
echo "Password: $(kubectl get secret grafana-admin --namespace default -o jsonpath="{.data.GF_SECURITY_ADMIN_PASSWORD}" | base64 -d)"
```

https://cylab.be/blog/197/deploy-loki-on-kubernetes-and-monitor-the-logs-of-your-pods

https://grafana.github.io/loki/charts/

https://github.com/leventebalogh/playground-nodejs-loki-grafana/blob/master/server.js

https://www.youtube.com/watch?v=VEGYgPiAazk

kubectl --namespace default port-forward service/loki 3100


https://devops4solutions.com/monitor-kubernetes-cluster-using-prometheus-and-grafana/
