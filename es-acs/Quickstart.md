```bash
az group create -l eastus -n IgniteACSDemo
az aks create -g IgniteACSDemo -n Ig23elsearch --os-sku AzureLinux --node-vm-size Standard_D4s_v3 --node-count 3 --enable-azure-container-storage azureDisk
az aks get-credentials --resource-group IgniteACSDemo --name Ig23elsearch
az aks nodepool add  --cluster-name Ig23elsearch --mode User --name espoolz1 --node-vm-size Standard_D4s_v3 --resource-group IgniteACSDemo --enable-cluster-autoscaler --max-count 12 --min-count 6 --node-count 6 --os-sku AzureLinux

```
