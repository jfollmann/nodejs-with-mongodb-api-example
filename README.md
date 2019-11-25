## Node.js with MongoDB Example

<img src="https://i.imgur.com/V6k9QVB.png" alt="Swagger Page of that application" title="Swagger Page of that application"/>

### Requirements

- Node.js v10+
- MongoDB running on local instance

#### Environment Variables

- PORT: 3000
- MONGO_URL: localhost:27017

### Running

- Install dependencies - `npm i`
- Build typescript - `npm run build`
- Run project - `npm start`
- Go to swagger page - `localhost:3000/`

### Development with Watch Compiler

- Run project - `npm run dev`

# ![Markdown Here logo](https://camo.githubusercontent.com/9cf76f2bd4230c106e7ea7c8b557a5afb4df85d7/68747470733a2f2f7261772e6769746875622e636f6d2f6164616d2d702f6d61726b646f776e2d686572652f6d61737465722f7372632f636f6d6d6f6e2f696d616765732f69636f6e34382e706e67) Docker, Kubernetes, AzCLI, ACS, AKS

### Conteúdo
**[Azure CLI](#azure-cli)**<br>
**[Azure Container Registry (ACR)](#Azure-Container-Registry-(ACR))**<br>
**[Azure Container Service (ACS)](#Azure-Container-Service-(ACS))**<br>
**[Azure Kubernetes Service (AKS)](#Azure-Kubernetes-Service-(AKS))**<br>
**[Kubectl](#Kubectl)**<br>
**[Kubectl - Pods](#Kubectl---Pods)**<br>
**[Kubectl - Services](#Kubectl---Services)**<br>
**[Kubectl - Deployments](#Kubectl---Deployments)**<br>
**[Kubectl - Secrets](#Kubectl---Secrets)**<br>
**[Kubectl - ReplicaSets](#Kubectl---ReplicaSets)**<br>

## Azure CLI

Login:
``` bash
az login
```

Show account list:
``` bash
az account list
```

Set subscription default: 
``` bash
az account set --subscription "Microsoft Azure Enterprise"
```

Create resource group:
``` bash
az group create --name k8s-rg --location eastus
```

## Azure Container Registry (ACR)

Create registry:
``` bash
az acr create --resource-group k8s-rg --name k8sregistry --sku Basic k8sregistry.azurecr.io
```

Login registry:
``` bash
az acr login --name k8sregistry
```

List images:
``` bash
az acr list --resource-group k8s-rg --output table
``` 

Get credentials (username & password):
``` bash
az acr update -n k8sregistry --admin-enabled true
az acr credential show -n k8sregistry --query username
az acr credential show -n k8sregistry --query "passwords[0].value"
```

## Azure Container Service (ACS)

Create ACS (Sample 1):
``` bash
az container create --resource-group k8s-rg \
  --name mongodb --image mongo \
  --cpu 1 --memory 1 \
  --port 27017 \
  --ip-address public
```

Create ACS (Sample 2):
```bash
az container create --resource-group k8s-rg \
  --name api-herois --image k8sregistry.azurecr.io/api-herois:v1 \
  --cpu 1 --memory 1 \
  --port 3000 \
  --environment-variables MONGO_URL=52.188.216.44 \
  --registry-username k8sregistry \
  --registry-password Ns07GHBgz2mZGM1DnWcWiMjFt=H371FE \
  --ip-address public
```

Get logs:
``` bash
az container logs --resource-group k8s-rg --name mongodb
```

Get IP:
``` bash
az container show --resource-group k8s-rg --name mongodb --query ipAddress.ip
```

Delete ACS: 
```bash
az container delete --resource-group k8s-rg --name api-herois --yes
```

## Azure Kubernetes Service (AKS)

Create cluster AKS:
```bash
az aks create -g k8s-rg \
  --name k8s-cluster \
  --dns-name-prefix k8s-cluster \
  --generate-ssh-keys \
  --node-count 2
```

Get credentials:
```bash
az aks get-credentials --resource-group k8s-rg --name k8s-cluster
```

Kubernetes browse:
```bash
az aks browse --resource-group k8s-rg --name k8s-cluster
```

Install kubectl (on host):
```bash
az aks install-cli
``` 

## Kubectl

Get version:
```bash
kubectl version
```

Show configs:
```bash
kubectl config view
```

Get current context:
```bash
kubectl config current-context
```

## Kubectl - Pods

Run pod (mongo sample): 
```bash
kubectl run mongodb --image mongo --port 27017
```

Get all pods: 
```bash
kubectl get pods
```

Get all pod with watch:
```bash
kubectl get pods -w
```

Delete pod:
```bash
kubectl delete pod mongodb-66499768f9-4s4q4
```

Describe pod:
```bash
kubectl describe pods mongodb-66499768f9-j4bm9
```

Get CPU & memory:
```bash
kubectl top pods mongodb-66499768f9-j4bm9
```

Get IP pod:
```bash
kubectl get pods --output=wide
```

Run pod (api-herois - Docker Registry):
```bash
kubectl run api-herois \
  --image jfollmann/api-herois:v1 \
  --env "MONGO_URL=10.244.1.4" \
  --env "PORT=4000" \
  --replicas 2
```

Pod Log:
```bash
kubectl logs api-herois-5d65b9d64f-n2dk7
```

Expose Port to Internet (Create Service):
```bash
kubectl expose pod api-herois-5d65b9d64f-n2dk7 --port 4000 --type LoadBalancer
```

Create pod with declarative mode:

Criação do arquivo manifest => .json ou yaml

heroes-pod.json:
``` json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "api-herois-pod",
    "labels": {
      "version": "v1",
      "app": "api-heroes"
    }
  },
  "spec": {
    "containers": [
      {
        "name": "api-herois",
        "image": "jfollmann/api-herois:v1",
        "ports": [
          {
            "containerPort": 4000
          }
        ],
        "env": [
          {
            "name": "MONGO_URL",
            "value": "10.244.1.4"
          },
          {
            "name": "PORT",
            "value": "4000"
          }
        ]
      }
    ]
  }
}
```

```bash
kubectl create -f 1.pods/heroes-pod.json
```

Delete pod:
```bash
kubectl delete pods api-herois-pod
```

Describe pod (get labels):
```bash
kubectl describe pods api-herois-pod
```

Delete pod by label:
```bash
kubectl delete pod -l version=v1
```

Connect pod with iterative mode:
```bash
kubectl exec -it api-herois-pod -- /bin/sh
```

Explain pod (json):
```bash
kubectl explain pods
```

Export configs of pod (json/yaml):
```bash
kubectl get pod api-herois-pod -o json
kubectl get pod api-herois-pod -o yaml
kubectl get pod api-herois-pod -o yaml | grep podIP
```

## Kubectl - Services

Get all services:
```bash
kubectl get service -w
```

Delete service:
```bash
kubectl delete service api-herois-5d65b9d64f-n2dk7
```

Expose Service (externo):
```bash
kubectl expose -f 3.replicasets/heroes-rc.json \
  --port 4000 \
  --type LoadBalancer # acesso externo
  # --type NodePort - seriço interno
```

## Kubectl - Deployments

Get all Deployments:
```bash
kubectl get deployment
```

## Kubectl - Secrets

Create Secret:
```bash
kubectl create secret docker-registry acr-credentials \
  --docker-server=k8sregistry.azurecr.io \
  --docker-username=k8sregistry \
  --docker-password=Ns07GHBgz2mZGM1DnWcWiMjFt=H371FE \
  --docker-email=jeff.follmann@gmail.com
```

Get all secrets:
```bash
kubectl get secrets
```

Get secret:
```bash
kubectl get secrets acr-credentials
```

Describe secret:
```bash
kubectl describe secret acr-credentials
```

Delete secret:
```bash
kubectl delete secrets acr-credentials
```

## Kubectl - ReplicaSets

Modifica pods existentes:
```bash
kubectl apply -f 3.replicasets/heroes-rc.json
```

Criar novos
```bash
kubectl create -f 3.replicasets/heroes-rc.json
```

Explain replicaset (json):
```bash
kubectl explain rc
```

Delete ReplicaSet:
```bash
kubectl delete -f 3.replicasets/heroes-rc.json
```