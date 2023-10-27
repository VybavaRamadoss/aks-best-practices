```bash
LOCATION=eastus # Location 
AKS_NAME=Ig23elsearch
RG=IgniteACSDemo
AKS_VNET_NAME=$AKS_NAME-vnet # The VNET where AKS will reside
AKS_CLUSTER_NAME=$AKS_NAME # name of the cluster
AKS_VNET_CIDR=172.16.0.0/16
AKS_NODES_SUBNET_NAME=$AKS_NAME-subnet # the AKS nodes subnet
AKS_NODES_NSG_NAME=$AKS_NAME-nsg # the AKS nodes subnet
AKS_NODES_SUBNET_PREFIX=172.16.0.0/23
SERVICE_CIDR=10.0.0.0/16
DNS_IP=10.0.0.10
NETWORK_PLUGIN=azure # use Azure CNI 
SYSTEM_NODE_COUNT=5  # system node pool size 
USER_NODE_COUNT=6 # change this to match your needs
NODES_SKU=Standard_D4ds_v4 # node VM type (change this to match your needs)
K8S_VERSION=1.27
SYSTEM_POOL_NAME=systempool
STORAGE_POOL_ZONE1_NAME=espoolz1
IDENTITY_NAME=$AKS_NAME`date +"%d%m%y"`
```

```bash
az identity create --name $IDENTITY_NAME --resource-group $RG
IDENTITY_ID=$(az identity show --name $IDENTITY_NAME --resource-group $RG --query id -o tsv)
IDENTITY_CLIENT_ID=$(az identity show --name $IDENTITY_NAME --resource-group $RG --query clientId -o tsv)

```

```bash
az network vnet create \
  --name $AKS_VNET_NAME \
  --resource-group $RG \
  --location $LOCATION \
  --address-prefix $AKS_VNET_CIDR \
  --subnet-name $AKS_NODES_SUBNET_NAME \
  --subnet-prefix $AKS_NODES_SUBNET_PREFIX \
  --network-security-group $AKS_NODES_NSG_NAME
```

```bash
RG_ID=$(az group show -n $RG  --query id -o tsv)
VNETID=$(az network vnet show -g $RG --name $AKS_VNET_NAME --query id -o tsv)
AKS_VNET_SUBNET_ID=$(az network vnet subnet show --name $AKS_NODES_SUBNET_NAME -g $RG --vnet-name $AKS_VNET_NAME --query "id" -o tsv)

az role assignment create --assignee $IDENTITY_CLIENT_ID --scope $RG_ID --role Contributor
az role assignment create --assignee $IDENTITY_CLIENT_ID --scope $VNETID --role Contributor
```

```bash
az aks create \
-g $RG \
-n $AKS_CLUSTER_NAME \
-l $LOCATION \
--os-sku AzureLinux \
--node-count $SYSTEM_NODE_COUNT \
--node-vm-size $NODES_SKU \
--network-plugin $NETWORK_PLUGIN \
--network-plugin-mode overlay \
--kubernetes-version $K8S_VERSION \
--generate-ssh-keys \
--service-cidr $SERVICE_CIDR \
--dns-service-ip $DNS_IP \
--vnet-subnet-id $AKS_VNET_SUBNET_ID \
--enable-addons monitoring \
--enable-managed-identity \
--assign-identity $IDENTITY_ID \
--nodepool-name $SYSTEM_POOL_NAME \
--uptime-sla \
--zones 1
```

```bash
az aks get-credentials -n $AKS_CLUSTER_NAME -g $RG
kubectl get nodes -o wide
kubectl describe nodes | grep -i topology.kubernetes.io/zone

```

```bash
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME \
--mode User \
--name $STORAGE_POOL_ZONE1_NAME \
--node-vm-size $NODES_SKU \
--resource-group $RG \
--zones 1 \
--enable-cluster-autoscaler \
--max-count 12 \
--min-count 6 \
--node-count $USER_NODE_COUNT \
--os-sku AzureLinux
--labels app=es
```

```bash
az aks update -g IgniteACSDemo -n Ig23elsearch --enable-azure-container-storage azureDisk --azure-container-storage-nodepools $STORAGE_POOL_ZONE1_NAME
az aks nodepool update --cluster-name Ig23elsearch \
                       --name systempool \
                       --resource-group $RG \
                      --labels acstor.azure.com/io-engine=acstor
```

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install elasticsearch-v1 bitnami/elasticsearch -n elasticsearch --values values_acs.yaml
watch kubectl get pods -o wide -n elasticsearch
kubectl get svc -n elasticsearch elasticsearch-v1
```

```bash
kubectl -n elasticsearch port-forward svc/elasticsearch-v1 9200:9200 &
curl http://localhost:9200/
##create an index called "acstor" with 3 replicas 
curl -X PUT "http://localhost:9200/acstor" -H "Content-Type: application/json" -d '{
  "settings": {
    "number_of_replicas": 3
  }
}'

##test the index 
curl -X GET "http://localhost:9200/acstor"
curl -X GET /my-index-000001/_stats
```


```bash
kubectl apply -f ingest-job.yaml

kubectl get pods -l app=log-ingestion 
kubectl logs -l app=log-ingestion -f 
```

```bash
watch kubectl get pods -n elasticsearch
watch kubectl get pvc -n elasticsearch
kubectl get hpa -n elasticsearch 
```

