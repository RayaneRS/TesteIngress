## Criar o cluster

  ```
  gcloud config set compute/zone southamerica-east1-a
  ```
      *Retorna informação similar a:*
      ```
      Updated property [compute/zone].
      ```
  ```
  gcloud container clusters create teste --num-nodes=2
  ```
      *Retorna informação similar a:*
      ```
      Created [https://container.googleapis.com/v1/projects/prismatic-vim-326823/zones/southamerica-east1-a/clusters/teste].
      To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/southamerica-east1-a/teste?project=prismatic-vim-326823
      kubeconfig entry generated for teste.
      NAME: teste
      LOCATION: southamerica-east1-a
      MASTER_VERSION: 1.21.5-gke.1302
      MASTER_IP: 35.247.248.107
      MACHINE_TYPE: e2-medium
      NODE_VERSION: 1.21.5-gke.1302
      NUM_NODES: 2
      STATUS: RUNNING
      ```

Ou criar pela interface.

## Instalar o HELM
  ```
  curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
  sudo apt-get install apt-transport-https --yes
  echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
  sudo apt-get update
  sudo apt-get install helm.
  helm version
  ```
 
## Configurar o cluster GKE para os testes:
**Foi necessário desabilitar a opção Balanceamento de carga HTTP do cluster criado no passo 1:**
* Acessar as configurações do cluster;
* Na categoria de REDE, desabilitar a opção de Balanceamento de carga HTTP.

## Conectar ao Cluster:
  ```
  gcloud container clusters get-credentials teste --zone southamerica-east1-a --project $(gcloud config get-value project)
  ```
      *Retorna informação similar a:*
      ```
      Your active configuration is: [cloudshell-26145]
      Fetching cluster endpoint and auth data.
      kubeconfig entry generated for teste.
      ```
## Configurar o controle de acesso com base em papéis:

  ```
  kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)
  ```
      *Retorna informação similar a:*
      ```
      Your active configuration is: [cloudshell-26145]
      clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-binding created
      ```
