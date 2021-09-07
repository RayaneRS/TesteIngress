# Teste de http utilizando o Ingress Controller NGINX:

## Passo1: Criação do cluster

```
$ gcloud config set compute/zone <Zona>
$ gcloud container clusters create <NomedoCluster> --num-nodes=2
```

Ou criar pela interface.

## Passo2: Instalar HELM
```
$ curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
$ sudo apt-get install apt-transport-https --yes
$ echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
$ sudo apt-get update
$ sudo apt-get install helm.
$ helm version
```
 
## Passo3: Configurar o cluster GKE para os testes:
Foi necessário desabilitar a opção Balanceamento de carga HTTP do cluster criado no passo 1:
- Acessar as configurações do cluster;
- Na categoria de REDE, desabilitar a opção de Balanceamento de carga HTTP ou já criar o cluster sem.

## Passo4: Conectar ao Cluster:
```
$ gcloud container clusters get-credentials <NomedoCluster> --zone <Zona> --project <IDProjeto>
```
 
## Passo5: Atualizar as permissões do usuário:
```
$ echo -n "
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: cluster-admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: User
  name: <Email>
  namespace: kube-system" | kubectl apply -f -
```

## Passo6: Instalação do NGINX:
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.48.1/deploy/static/provider/cloud/deploy.yaml
```
ou
```
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
$ helm install ingress-nginx ingress-nginx/ingress-nginx
```

## Passo7: Criar uma variável de ambiente com o IP:
```
$ kubectl get services -n ingress-nginx
$ export PROXY_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" service -n ingress-nginx ingress-nginx-controller)
$ echo $PROXY_IP/foo
```

## Passo7: Service e Deployment GRPC:
```
$ echo "
apiVersion: v1
kind: Service
metadata:
  name: grpcbin
  labels:
    app: grpcbin
spec:
  ports:
  - name: grpc
    port: 9001
    targetPort: 9001
  selector:
    app: grpcbin
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grpcbin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grpcbin
  template:
    metadata:
      labels:
        app: grpcbin
    spec:
      containers:
      - image: moul/grpcbin
        name: grpcbin
        ports:
        - containerPort: 9001
" | kubectl apply -f -
```

# HELP - Não consigo achar uma forma simples de funcionar o grpc T-T
## Passo8: Proxy:
```
$ echo "
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
spec:
  spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: grpcbin
          servicePort: 9001" | kubectl apply -f -
"
```
## Passo9: Testar a conexão:
```
$ grpcurl -v -d '{"greeting": "Hello world!"}' -insecure $PROXY_IP:443 hello.HelloService.SayHello

```
