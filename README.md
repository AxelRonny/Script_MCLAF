# Script_MCLAF
#!/bin/bash
# <FullScript>

# Variable block
let "randomIdentifier=$RANDOM*$RANDOM"
location="francecentral"
resourceGroup="B7group2mclaf"-$randomIdentifier"
tag="load-balance-wordpress"
availabilitySet="mclaf-availablity-set-lb-$randomIdentifier"
vNet="mclaf-vnet-lb-$randomIdentifier"
subnet="mclaf-subnet-lb-$randomIdentifier"
loadBalancerPublicIp="mclaf-public-ip-lb-$randomIdentifier"
loadBalancerFrontEndConfig1="mclaf-contoso-lb-$randomIdentifier"
loadBalancerFrontEndConfig2="mclaf-fabrikam-lb-$randomIdentifier"
loadBalancer="mclaf-load-balancer-$randomIdentifier"
frontEndIp="mclaf-front-end-ip-lb-$randomIdentifier"
frontEndIpConfig1="mclaf-front-end-ip-config-contoso-lb-$randomIdentifier"
frontEndIpConfig2="mclaf-front-end-ip-config-fabrikam-lb-$randomIdentifier"
backEndPool=" mclaf -back-end-pool-lb-$randomIdentifier"
backEndAddressPool1=" mclaf -back-end-address-pool-contoso-lb-$randomIdentifier"
backEndAddressPool2=" mclaf -back-end-address-pool-fabrikam-lb-$randomIdentifier"
probe80=" mclaf -port80-health-probe-lb-$randomIdentifier"
lbRuleConfig1=" mclaf -contoso-load-balancer-rule-$randomIdentifier"
lbRuleConfig2=" mclaf -fabrikam-load-balancer-rule-$randomIdentifier"
publicIpVm1=" mclaf -public-ip-vm1-lb-$randomIdentifier"
nicVm1=" mclaf -nic-vm1-$randomIdentifier"
ipConfig1=" mclaf -ipconfig-config1-lb-$randomIdentifier"
ipConfig2=" mclaf -ipconfig-config2-lb-$randomIdentifier"
vm1=" mclaf -vm1-lb-$randomIdentifier"
publicIpVm2=" mclaf -public-ip-vm2-lb-$randomIdentifier"
ipSku="Standard"
nicVm2=" mclaf -nic-vm2-lb-$randomIdentifier"
ipConfig1=" mclaf -ipconfig-config1-lb-$randomIdentifier"
ipConfig2=" mclaf -ipconfig-config2-lb-$randomIdentifier"
vm2=" mclaf -vm2-lb-$randomIdentifier"
image="UbuntuLTS"
login="b7group2"

# Create a resource group
echo "Creating $resourceGroup in "$location"..."
az group create --name $resourceGroup --location "$location" --tags $tag

# Create an availability set for the two VMs that host both websites.
echo "Creating $availabilitySet"
az vm availability-set create --resource-group $resourceGroup --location "$location" --name $availabilitySet --platform-fault-domain-count 2 --platform-update-domain-count 2

# Create a virtual network and a subnet.
echo "Creating $vNet and $subnet"
az network vnet create --resource-group $resourceGroup --name $vNet --address-prefix 10.0.0.0/16 --location "$location" --subnet-name $subnet --subnet-prefix 10.0.0.0/24

# Create three public IP addresses; one for the load balancer and two for the front-end IP configurations.
echo "Creating $loadBalancerPublicIp"
az network public-ip create --resource-group $resourceGroup --name $loadBalancerPublicIp --allocation-method Dynamic

echo "Creating $loadBalancerFrontEndConfig1"  
az network public-ip create --resource-group $resourceGroup --name $loadBalancerFrontEndConfig1 --allocation-method Dynamic

echo "Creating $loadBalancerFrontEndConfig2"
az network public-ip create --resource-group $resourceGroup --name $loadBalancerFrontEndConfig2 --allocation-method Dynamic

# Create an Azure Load Balancer.
echo "Creating $loadBalancer with $frontEndIP and $backEndPool"
az network lb create --resource-group $resourceGroup --location "$location" --name $loadBalancer --frontend-ip-name $frontEndIp --backend-pool-name $backEndPool --public-ip-address $loadBalancerPublicIp

# Create two front-end IP configurations for both web sites.
echo "Creating $frontEndIpConfig1 for $loadBalancerFrontEndConfig1"
az network lb frontend-ip create --resource-group $resourceGroup --lb-name $loadBalancer --public-ip-address $loadBalancerFrontEndConfig1 --name $frontEndIpConfig1

echo "Creating $frontEndIpConfig2 for $loadBalancerFrontEndConfig2"  
az network lb frontend-ip create --resource-group $resourceGroup --lb-name $loadBalancer --public-ip-address $loadBalancerFrontEndConfig2 --name $frontEndIpConfig2

# Create the back-end address pools.
echo "Creating $backEndAddressPool1"
az network lb address-pool create --resource-group $resourceGroup --lb-name $loadBalancer --name $backEndAddressPool1

echo "Creating $backEndAddressPool2"
az network lb address-pool create --resource-group $resourceGroup --lb-name $loadBalancer --name $backEndAddressPool2

# Create a probe on port 80.
echo "Creating $probe80 in $loadBalancer"
az network lb probe create --resource-group $resourceGroup --lb-name $loadBalancer --name $probe80 --protocol Http --port 80 --path /

# Create the load balancing rules forport 80.
echo "Creating $lbRuleConfig1 for $loadBalancer"
az network lb rule create --resource-group $resourceGroup --lb-name $loadBalancer --name $lbRuleConfig1 --protocol Tcp --probe-name $probe80 --frontend-port 5000 --backend-port 5000 --frontend-ip-name $frontEndIpConfig1 --backend-pool-name $backEndAddressPool1

echo "Creating $lbRuleConfig2 for $loadBalancer"
az network lb rule create --resource-group $resourceGroup --lb-name $loadBalancer --name $lbRuleConfig2 --protocol Tcp --probe-name $probe80 --frontend-port 5000 --backend-port 5000 --frontend-ip-name $frontEndIpConfig2 --backend-pool-name $backEndAddressPool2

# VM1 #

# Create an Public IP for the first VM.
echo "Creating $publicIpVm1"
az network public-ip create --resource-group $resourceGroup --name $publicIpVm1 --allocation-method Dynamic

# Create a network interface for VM1.
echo "Creating $nicVm1 in $vNet and $subnet"
az network nic create --resource-group $resourceGroup --vnet-name $vNet --subnet $subnet --name $nicVm1 --public-ip-address $publicIpVm1

# Create IP configurations for VM1.
echo "Creating $ipConfig1 for $nicVm1"
az network nic ip-config create --resource-group $resourceGroup --name $ipConfig1 --nic-name $nicVm1 --lb-name $loadBalancer --lb-address-pools $backEndAddressPool1

echo "Creating $ipConfig2 for $nicVm1"
az network nic ip-config create --resource-group $resourceGroup --name $ipConfig2 --nic-name $nicVm1 --lb-name $loadBalancer --lb-address-pools $backEndAddressPool2

# Create Vm1.
echo "Creating $vm1"
az vm create --resource-group $resourceGroup --name $vm1 --nics $nicVm1 --image $image --availability-set $availabilitySet --public-ip-sku $ipSku --admin-username $login --generate-ssh-keys

# VM2 #

# Create an Public IP for the second VM2.
echo "Creating $publicIpVm2"
az network public-ip create --resource-group $resourceGroup --name $publicIpVm2 --allocation-method Dynamic

# Create a network interface for VM2.
echo "Creating $nicVm2 in $vNet and $subnet"
az network nic create --resource-group $resourceGroup --vnet-name $vNet --subnet $subnet --name $nicVm2 --public-ip-address $publicIpVm2

# Create IP-Configs for Vm2.
echo "Creating $ipConfig1 for $nicVm2"
az network nic ip-config create --resource-group $resourceGroup --name $ipConfig1 --nic-name $nicVm2 --lb-name $loadBalancer --lb-address-pools $backEndAddressPool1

echo "Creating $ipConfig2 for $nicVm2"
az network nic ip-config create --resource-group $resourceGroup --name $ipConfig2 --nic-name $nicVm2 --lb-name $loadBalancer --lb-address-pools $backEndAddressPool2

# Create Vm2.
echo "Creating $vm2"
az vm create --resource-group $resourceGroup --name $vm2 --nics $nicVm2 --image UbuntuLTS --availability-set $availabilitySet --public-ip-sku $ipSku --admin-username $login --generate-ssh-keys

# List the virtual machines
az vm list --resource-group $resourceGroup
# </FullScript>

# echo "Deleting all resources"
# az group delete --name $resourceGroup -y 
az mariadb server create \
    --resource-group $GroupeDeRessource \
    --name $mydbserv \
    --location francecentral \
    --admin-user $UserMDB \
    --admin-password Denyro69!Ando? \
    --sku-name GP_Gen5_2 \
    --version 10.2

az mariadb server firewall-rule create \
    --resource-group $mclaf-load-balancer-rg \
    --server $mydbserv \
    --name AllowMyIP \
    --start-ip-address 0.0.0.0 \ 
    --end-ip-address 0.0.0.0 

az mariadb server show \
    --resource-group $mclaf-load-balancer-rg \
    --name $mydbserv
