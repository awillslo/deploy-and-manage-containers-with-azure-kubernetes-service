# Lab - Deploy applications to Azure Kubernetes Service (AKS)

## Objectives

This lab consist of the following exercises:

+ Exercise 1: Provision Azure Container Registry (ACR) and Azure Kubernetes Service (AKS)
+ Exercise 2: Build a Linux and Windows container images and store them in ACR
+ Exercise 3: Deploy container images to AKS 
+ Exercise 4: Review the deployment and deprovision all resources

## Exercise 1: Provision Azure Container Registry (ACR) and Azure Kubernetes Service (AKS)
In this exercise, you will create an Azure Container registry and an AKS cluster.

### Task 1: Create an Azure Container registry
In this task, you will create an Azure Container registry

1. From your computer, open a web browser window and navigate to the Azure portal at https://portal.azure.com.
1. When prompted, sign in by using a user account that has the Owner role in the Azure subscription you will be using in this lab. 
1. Sign in to the Azure portal. 
1. In the Azure portal, in the **Search** text box, search for and select **Container registries**.
1. On the **Container registries** page, select **+ Create** and specify the following settings:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you will be using in this lab|
    |Resource Group|The name of a new resource group **acr-01-RG**|
    |Registry name|Any valid, globally unique name consisting of between 5 and 50 alphanumeric characters|
    |Region|Any Azure region in which you can create an Azure Container registry and an AKS cluster|
    |Availability zones|**None**|
    |SKU|**Basic**|

1. On the **Container registries** page, select **Review + create** and, on the **Review + create** tab, select **Create**.

   > **Note:** Proceed to the next exercise without waiting for the provisioning of the Azure Container registry to complete.

### Task 2: Create an Azure virtual network and an AKS cluster
In this task, you will create an Azure virtual network and deploy an AKS cluster including a Windows node pool into that virtual network.

> **Note:** While you could create a virtual network when provisioning an AKS cluster, this is not a typical scenario. More importantly, deploying AKS clusters into an existing virtual network requires some additional considerations that you should be familiar with.

1. In the Azure portal, in the **Search** text box, search for and select **Virtual networks**.
1. On the **Virtual networks** page, select **+ Create** and then, on the **Basics** tab of the **Create virtual network** page, specify the following settings:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you selected in the first exercise of this lab|
    |Resource Group|The name of a new resource group **aks-01-RG**|
    |Virtual network name|**vnet-01**|
    |Region|The same Azure region you selected in the first exercise of this lab|

1. On the **Basics** tab of the **Create virtual network** page, select **Next**.
1. On the **Security** tab of the **Create virtual network** page, accept the default settings and select **Next**.
1. On the **IP addresses** tab of the **Create virtual network** page, ensure that the IP address space is set to **10.0.0.0/16**, delete the **default** subnet, select **Review + create** and, on the **Review + create** tab, select **Create**.

   > **Note:** Creating a virtual network should take only a few seconds, so you should be able to proceed directly to the next step.

1. In the Azure portal, in the **Search** text box, search for and select **Kubernetes services**.
1. On the **Kubernetes services** page, select **+ Create**, in the dropdown list, select **Create a Kubernetes cluster**, and then, on the **Basics** tab of the **Create Kubernetes cluster** page, specify the following settings:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you selected in the first exercise of this lab|
    |Resource Group|**aks-01-RG**|
    |Cluster preset configuration|**Dev/Test**|
    |Kubernetes cluster name|**aks-01**|
    |Region|The same Azure region you selected in the first exercise of this lab|
    |Availability zones|**None**|
    |AKS pricing tier|**Free**|
    |Kubernetes version|Accept the default value|
    |Automatic upgrade|Disabled|
    |Node size|**Standard B4ms**|
    |Scale method|**Manual**|
    |Node count|**2**|

   > **Note:** You might need to increase vCPU quotas or change the VM SKU to accommodate the node size and node count values. For information regarding the procedure to increase vCPU quotas, refer to the Microsoft Learn article [Increase VM-family vCPU quotas](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests).

1. On the **Basics** tab of the **Create Kubernetes cluster** page, select **Next: Node pools >**.
1. On the **Node pools** tab of the **Create Kubernetes cluster** page, select **Next: Access >**.
1. On the **Access** tab of the **Create Kubernetes cluster** page, select **Next: Networking >**.
1. On the **Networking** tab of the **Create Kubernetes cluster** page, select the **Azure CNI** option, in the **Virtual network** dropdown list, select **vnet-01**, and below the **Cluster subnet** textbox, select **Managed subnet configuration**.
1. On the **vnet-01 \| Subnets** page, select **+ Subnet**.
1. On the **Add subnets** page, specify the following settings and select **Save**:

    |Setting|Value|
    |---|---|
    |Name|**aks-subnet**|
    |Subnet address range|**10.0.0.0/20**|

1. Back on the **vnet-01 \| Subnets** page, in the breadcrumb trail in the top-left part of the page, select **Create Kubernetes cluster**. 
1. Back on the **Networking** tab of the **Create Kubernetes cluster** page, specify the following settings:

    |Setting|Value|
    |---|---|
    |Cluster subnet|**aks-subnet (10.0.0.0/20)**|
    |Kubernetes service address range|**172.16.0.0/22**|
    |Kubernetes DNS service IP address|**172.16.3.254**|
    |DNS name prefix|**aks-01-dns**|
    |Enable private cluster|Disabled|
    |Set authorized IP ranges|Disabled|
    |Network policy|**None**|

1. On the **Networking** tab of the **Create Kubernetes cluster** page, select the **Node pools** tab.

   > **Note:** You will add a Windows node pool to the cluster. This required changing the network configuration to **Azure CNI** from the default **Kubenet**. The Kubenet network configuration does not support Windows node pools.

1. On the **Node pools** tab of the **Create Kubernetes cluster** page, select **+ Add node pool**.
1. On the **Add node pool** page, specify the following settings:

    |Setting|Value|
    |---|---|
    |Node pool name|**w1pool**|
    |Mode|**User**|
    |OS type|**Windows**|
    |Availability zone|**None**|
    |Enable Azure spot instances|Disabled|
    |Node size|**Standard B4s_v2**|
    |Scale method|**Manual**|
    |Node count|**2**|
    |Max pods per node|**30**|
    |Enable public IP per node|Disabled|

   > **Note:** You might need to increase vCPU quotas or change the VM SKU to accommodate the node size and node count values. For information regarding the procedure to increase vCPU quotas, refer to the Microsoft Learn article [Increase VM-family vCPU quotas](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests).

1. On the **Add node pool** page, select **Add**.
1. Back on the **Node pools** tab of the **Create Kubernetes cluster** page, select the **Integrations** tab.
1. On the **Integration** tab of the **Create Kubernetes cluster** page, in the **Container registry** dropdown list, select the entry representing the Azure Container registry you created in the previous exercise, disable the **Enable recommended alert rules** checkbox, ensure that **Azure Policy** option is disabled, and select **Review + create**.
1. On the **Review + create** tab of the **Create Kubernetes cluster** page, select **Create**.

   > **Note:** Proceed to the next exercise without waiting for the provisioning of AKS cluster to complete. The provisioning process might take about 5 minutes.