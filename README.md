# Tutorial on Azure Red Hat OpenShift

Azure Red Hat OpenShift is Managed OpenShift solution that you can deploy OpenShift cluster quickly and run conntainers on it without the burden of maintaining underline infrastructures. 

## Deploy OpenShift cluster

Make sure latest az cli tool (v2.0.65 or above) is installed. 

```
$ az --version
azure-cli                         2.0.76
```

Login
```
$ az login
```
Register required az providers and freatures
```
az feature register --namespace Microsoft.ContainerService -n AROGA
az provider register -n Microsoft.Storage --wait
az provider register -n Microsoft.Compute --wait
az provider register -n Microsoft.Solutions --wait
az provider register -n Microsoft.Network --wait
az provider register -n Microsoft.KeyVault --wait
az provider register -n Microsoft.ContainerService --wait
```
Create an Azure Active Directory (Azure AD) tenant which will be intergrated with OpenShift/Kubernetes RBAC features.
https://docs.microsoft.com/en-us/azure/openshift/howto-create-tenant

Create Azure AD app object and user
https://docs.microsoft.com/en-us/azure/openshift/howto-aad-app-configuration

Set variables below
```
CLUSTER_NAME=myarocluster
LOCATION=southeastasia

# In this tutorial, credentials are managed as secrets using Azure Key Vault. Retreive secrets from the keyvault.
keyvault=$(az resource list --tag arocluster=myarocluster | jq -r .[0].name)
TENANT=$(az keyvault secret show -n myarocluster-tenant-id --vault-name ${keyvault} | jq -r .value) && echo " - tenant ID: ${TENANT}"
GROUPID=$(az keyvault secret show -n myarocluster-customer-admin-group-id --vault-name ${keyvault} | jq -r .value) && echo " - group ID: ${GROUPID}"
APPID=$(az keyvault secret show -n myarocluster-aad-client-app-id --vault-name ${keyvault} | jq -r .value) && echo " - group ID: ${APPID}"
SECRET=$(az keyvault secret show -n myarocluster-aad-client-app-secret --vault-name ${keyvault} | jq -r .value) && echo " - group ID: ${SECRET}"

# create resource group
az group create --name $CLUSTER_NAME --location $LOCATION

# create Azure Red Hat OpenShift cluster
az openshift create --resource-group $CLUSTER_NAME --name $CLUSTER_NAME -l $LOCATION --aad-client-app-id $APPID --aad-client-app-secret $SECRET --aad-tenant-id $TENANT --customer-admin-group-id $GROUPID
```

Get OpenShift console (OC) sign in URL for your cluster
```
az openshift show -n $CLUSTER_NAME -g $CLUSTER_NAME
```

Update your app registration redirect URI (https://docs.microsoft.com/en-us/azure/openshift/tutorial-create-cluster#step-3-update-your-app-registration-redirect-uri)

Sign in with user account created in the Azure AD tenant which is created earlier.

![Azure Red Hat OpenShift amdin console](aro-console.png)



