# Teste de http utilizando o Ingress Controller NGINX:

## Passo1: Criação do cluster

```
$ gcloud config set compute/zone southamerica-east1-a
$ gcloud container clusters create teste --num-nodes=2
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
$ gcloud container clusters get-credentials teste --zone southamerica-east1-a --project prismatic-vim-326823
```
 
## Passo5: Configurar o controle de acesso com base em papéis:
```
echo -n "
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
  name: douglas.santos@aluno.ufr.edu.br
  namespace: kube-system" | kubectl apply -f -
```

## Passo6: 

### Instalação do NGINX:
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.48.1/deploy/static/provider/cloud/deploy.yaml
```
ou
```
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
$ helm install ingress-nginx ingress-nginx/ingress-nginx
```
### Instalação do Kong:
```
$ kubectl create -f https://bit.ly/k4k8s
```
ou 
```
$ helm repo add kong https://charts.konghq.com
$ helm repo update
$ helm install kong/kong --generate-name --set ingressController.installCRDs=false
```

### Instalação do HAproxy:
```
$ helm repo add haproxytech https://haproxytech.github.io/helm-charts
$ helm repo update
$ helm install kubernetes-ingress haproxytech/kubernetes-ingress \
    --set controller.service.type=LoadBalancer
```
## Passo7: Criar uma variável de ambiente com o IP:
```
$ kubectl get services
$ export EXEMPLO_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" service -n <informe o Namespace> <informe o Service>)
$echo $EXEMPLO_IP
```

## Passo7: Service e Deployment:
```
$ echo "
apiVersion: v1
kind: Service
metadata:
  labels:
    app: echo
  name: echo
spec:
  ports:
  - port: 8080
    name: high
    protocol: TCP
    targetPort: 8080
  - port: 80
    name: low
    protocol: TCP
    targetPort: 8080
  selector:
    app: echo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: echo
  name: echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: echo
    spec:
      containers:
      - image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
        name: echo
        ports:
        - containerPort: 8080
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
        resources: {}
" | kubectl apply -f -
```
## Passo8: Configuração do Ingress:
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo
  annotations:
    kubernetes.io/ingress.class: "nginx" 
    nginx.org/websocket-services: "websocket-teste"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
spec:
  rules:
  - host: ${EXEMPLO_IP}.nip.io
    http:
      paths:
      - path: /teste
        backend:
          serviceName: echo
          servicePort: 80
EOF
```
ou 
```
cat <<EOF | kubectl create -f -

EOF
```
ou
```
cat <<EOF | kubectl create -f -

EOF
```

## Passo9: Testar a conexão:
```
$ curl -i $EXEMPLO_IP/foo
```

curl -i -N -H "Connection: Upgrade" \
        -H "Upgrade: websocket" \
        -H "Origin: http://localhost" \
        -H "Host: ${EXEMPLO_IP}.nip.io" \
        -H "Sec-Websocket-Version: 13" \
        -H "Sec-WebSocket-Key: 123" \
        $EXEMPLO_IP/foo
```

