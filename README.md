# Capstone-Project

### Mendeploy LMS Moodle menggunakan Helm ke Azure Kubernetes Service

1. Buat Resource Group

```
az group create --name capstoneProject --locaation southeastasia
```
2. Buat Cluster Azure Kubernetes Service

```
az aks create \
--resource-group capstoneProject \
--name AKS-Cluster \
--node-count 1 \
--vm-set-type VirtualMAchineScaleSets \
--load-balancer-sku standart \
--min-count 1 \
--max-count 3
```
3. Mendapatkan AKS kredensial
```
az aks get-credentials --name AKS-Cluster --resource-group capstoneProject
```
4. Deploy Moodle menggunakan Helm
```
helm repo add azure-marketplace https://marketplace.azurecr.io/helm/v1/repo
```
```
helm install aks azure-marketplace/moodle
```
5. Melihat Pod AKS
```
kubectl get pods -w
```
6. Melihat IP Eksternal Load Balancer
```
export SERVICE_IP=$(kubectl get svc --namespace default aks-moodle --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
echo "Moodle&trade; URL: http://$SERVICE_IP/"
```
7. Membuat user dan password admin
```
echo Username: user
echo Password: $(kubectl get secret --namespace default aks-moodle -o jsonpath="{.data.moodle-password}" | base64 --decode)
```
8. Membuat script bash fqdn untuk setting DNS di AKS
```
touch FQDN
nano FQDN
```
```
!/bin/bash

# Public IP address
IP=20.212.40.144

# Name to associate with public IP address
DNSNAME=belajarkoding

# Get resource group and public ip name
RESOURCEGROUP=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[resourceGroup]" --output tsv)
PIPNAME=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[name]" --output tsv)

# Update public ip address with dns name
az network public-ip update --resource-group $RESOURCEGROUP --name  $PIPNAME --dns-name $DNSNAME
```
Menjalankan script bash
```
bash FQDN
```
### Akses ke LMS Moodle menggunakan DNS dan login menggunakan admin

