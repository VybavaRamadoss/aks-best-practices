```bash
az group create -l eastus -n IgniteACSDemo
az aks create -g IgniteACSDemo -n Ig23elsearch --os-sku AzureLinux --node-vm-size Standard_D4s_v3 --node-count 3 --enable-azure-container-storage azureDisk
az aks get-credentials --resource-group IgniteACSDemo --name Ig23elsearch ![image](https://github.com/VybavaRamadoss/aks-best-practices/assets/8097055/bd506e31-0025-4e62-8e92-832aaed18acd)
az aks nodepool add  --cluster-name Ig23elsearch --mode User --name espoolz1 --node-vm-size Standard_D4s_v3 --resource-group IgniteACSDemo --enable-cluster-autoscaler --max-count 12 --min-count 6 --node-count 6![image](https://github.com/VybavaRamadoss/aks-best-practices/assets/8097055/5a159f58-8fb7-4e3b-a100-2a0f910dd919)
```
