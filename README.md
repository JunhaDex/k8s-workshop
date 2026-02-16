How to run

```shell
docker build -t k8s-ws-api ./c0_setup/api
docker build -t k8s-ws-client ./c0_setup/client
minikube image load k8s-ws-api
minikube image load k8s-ws-client

kubectl apply -f ./c1_pod/101-simple-pod.yml
kubectl port-forward pod/simple-api-pod 3000:3000



```