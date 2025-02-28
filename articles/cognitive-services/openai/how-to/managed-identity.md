---
title: How to Configure Azure OpenAI Service with Managed Identities
titleSuffix: Azure OpenAI
description: Provides guidance on how to set managed identity with Azure Active Directory
ms.service: cognitive-services
ms.subservice: openai
ms.topic: how-to 
ms.date: 06/24/2022
author: ChrisHMSFT
ms.author: chrhoder
recommendations: false
ms.custom: devx-track-azurecli
---

# How to Configure Azure OpenAI Service with Managed Identities

More complex security scenarios require Azure role-based access control (Azure RBAC). This document covers how to authenticate to your OpenAI resource using Azure Active Directory (Azure AD).

In the following sections, you'll use  the Azure CLI to assign roles, and obtain a bearer token to call the OpenAI resource. If you get stuck, links are provided in each section with all available options for each command in Azure Cloud Shell/Azure CLI.

## Prerequisites

- An Azure subscription - <a href="https://azure.microsoft.com/free/cognitive-services" target="_blank">Create one for free</a>
- Access granted to the Azure OpenAI service in the desired Azure subscription

    Currently, access to this service is granted only by application. You can apply for access to Azure OpenAI by completing the form at <a href="https://aka.ms/oai/access" target="_blank">https://aka.ms/oai/access</a>. Open an issue on this repo to contact us if you have an issue.
- Azure CLI - [Installation Guide](/cli/azure/install-azure-cli)
- The following Python libraries: os, requests, json

## Sign into the Azure CLI

To sign-in to the Azure CLI, run the following command and complete the sign-in. You may need to do it again if your session has been idle for too long.

```azurecli
az login
```

## Assign yourself to the Cognitive Services User role

Assigning yourself to the Cognitive Services User role will allow you to use your account for access to the specific cognitive services resource

1. Get your user information

    ```azurecli
    export user=$(az account show -o json | jq -r .user.name)
    ```

2. Assign yourself to “Cognitive Services User” role.

    ```azurecli
    export resourceId=$(az group show -g $myResourceGroupName -o json | jq -r .id)
    az role assignment create --role "Cognitive Services User" --assignee $user --scope $resourceId
    ```

    > [!NOTE]
    > Role assignment change will take ~5 mins to become effective.

3. Acquire an Azure AD access token. Access tokens expire in one hour. you'll then need to acquire another one.

    ```azurecli
    export accessToken=$(az account get-access-token --resource https://cognitiveservices.azure.com -o json | jq -r .accessToken)
    ```

4. Make an API call
Use the access token to authorize your API call by setting the `Authorization` header value.

    ```bash
    curl ${endpoint%/}/openai/deployments/YOUR_DEPLOYMENT_NAME/completions?api-version=2022-12-01 \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $accessToken" \
    -d '{ "prompt": "Once upon a time" }'
    ```

## Authorize access to managed identities

OpenAI supports Azure Active Directory (Azure AD) authentication with [managed identities for Azure resources](../../../active-directory/managed-identities-azure-resources/overview.md). Managed identities for Azure resources can authorize access to Cognitive Services resources using Azure AD credentials from applications running in Azure virtual machines (VMs), function apps, virtual machine scale sets, and other services. By using managed identities for Azure resources together with Azure AD authentication, you can avoid storing credentials with your applications that run in the cloud.  

## Enable managed identities on a VM

Before you can use managed identities for Azure resources to authorize access to Cognitive Services resources from your VM, you must enable managed identities for Azure resources on the VM. To learn how to enable managed identities for Azure Resources, see:

- [Azure portal](../../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Azure Resource Manager template](../../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Azure Resource Manager client libraries](../../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

For more information about managed identities, see [Managed identities for Azure resources](../../../active-directory/managed-identities-azure-resources/overview.md).
