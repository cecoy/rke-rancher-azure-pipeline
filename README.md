# RKE using ARM and Azure Pipelines
This package is meant to deploy deploy a production ready RKE cluster on Microsoft Azure using Dynamic DNS and KeyVault. Further it is meant to be rerunable to maintain state.

## Pipeline Outline
The code is divided up into three stages,
.   the azure resource stage
.   the rke cluster stage
.   the in cluster setup, rancher and any other services
The pipeline is in `azure-pipeline.yaml`

### Stage 1: azure resources
    - Cluster Resources
      ubunutu agent
      Infra
        Step 1: Deploy Security group
        Step 2: Deploy Cluster
        Step 3: Write Node Info to KeyVault
        Step 4: Wait until nodes are updated

### Stage 2: Kubernetes
    - ubunutu agent
      RKE Cluster
      job rke:
        Step 1: Load keyvault values
        Step 2: Load rkestate from blob storage
        Step 3: convert to files: kubeconfig
        Step 4: convert to files: ssh keys
        step 5: download rke binary
        step 6: load rke
        step 7: save state to blob storage
        step 8: save kubeconfig to keyvault

### Stage 3: Services
    - ubuntu agent
      Cluster Services
      Step 1: Load keyvault
      Step 2: upgrade/install cert manager
      Step 3: upgrade/install rancher


# RKE using ARM and Azure Pipelines
This package is meant to deploy deploy a production ready RKE cluster on Microsoft Azure using Dynamic DNS and KeyVault. Further it is meant to be rerunable to maintain state.

## Components
The packages are broken down into the following components.
### Input Variables
* SUBSCRIPTION: Just the subscription id
* RMLINK: The link to the subscription
* DEPLOYMENTNAME: name of the deployment
* RESOURCEGROUP: azure resource group
* HOSTNAME: hostname
* LOCATION: region
* CLUSTERNAME: name of cluster and components
* NODECOUNT: 3
* NODESIZE: 'Standard_DS1_v2'
* NODEADMINUSER: 'ubuntu'
* RKEVERSION: 'Version of RKE, default v1.2.1'
* KEYVAULTNAME: Name of keyvault
* STORAGENAME: Name of storage account
* CONTAINERNAME: 'rketest'
* CLOUDINIT: Base64 encoded cloud init script contents
* RESTORE: filename of restore etcd snapshot without .zip
* RANCHERVERSION: version of rancher, default v2.4.10

### Cloud Provider Secrets
* SA_CLIENTID: Service Account App/Client ID
* SA_CLIENTSECRET:  Client Secret / Password
* SA_TENANTID: Tenant Id

### Keyvault Secrets
* SSHKEY: the base64 encoded SSH key
* SSHPUBKEY: They non base64 pub key
* STORAGEKEY: they priv key to the storage account.
* KUBECONFIG: base64 encoded kube config
* ENCKEY: key to encrypt and decrypt the rke state.
* CRT: ssl pfx certificate with chain, key, and no password

#### Automatically configured and stored in KeyVault
* LBIP: Load balancer ip 
* NODES: Node Info
* NODESPRIV: Node private ips, line separated
* NODESPUB: node public IPs, line separated


### Azure Resources: (ARM)
* resource group
* keyvault 
* domain zone
* vmss
* load balancer port 80
* load balancer port 443
* temp port 22

### The Cluster: (RKE CLI)
* RKE config
* RKE state
* Cluster Config
* snapshot restore file

### The Cluster Components: (Helm)
* Ingress - this is installed as part of rke install as an addon.
* Cert Manager
* Rancher
