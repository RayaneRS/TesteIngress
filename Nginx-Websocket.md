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
```

## Passo7: Service e Deployment:
```
$ echo "
apiVersion: v1
kind: Service
metadata:
  name: websocket-teste
  labels:
    app: websocket-teste
spec:
  ports:
  - name: websocket-teste
    port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: websocket-teste
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websocket-teste
spec:
  replicas: 1
  selector:
    matchLabels:
      app: websocket-teste
  template:
    metadata:
      labels:
        app: websocket-teste
    spec:
      containers:
      - name: websocket-teste
        image: k8s.gcr.io/echoserver:1.4
        ports:
        - containerPort: 8080
" | kubectl apply -f -
```
## Passo8: Proxy:
```
$ cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: websocket-teste
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
spec:
  rules:
  - host: websocket-teste.domain.com
    http:
      paths:
      - path: /
        backend:
          serviceName: websocket-teste
          servicePort: 80
EOF
```

## Passo9: Testar a conexão:
```

```

