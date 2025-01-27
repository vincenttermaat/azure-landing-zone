---
title: "Use Managed Identity to call an Azure Container App or App Service API"
date: 2025-01-26
author: "Vincent ter Maat"
tags: security microsoft-identity-platform managed-identities azure-entra-id
---

How to leverage cloud-native security mechanisms to secure and authenticate API resources in Azure.

1. Enable Bicep extensibility (preview) to enable Microsoft Graph API support [Use extensions in Bicep - Azure Resource Manager - Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-extension)
2. Deploy app-reg for target [Bicep templates for Microsoft Graph documentation - Microsoft Learn](https://learn.microsoft.com/en-us/graph/templates/)
3. Configured App-reg on target
4. Enable managed identity on client-api
5. Grant Managed Identity of client access to App-Registration
6. Use [DefaultAzureCredential](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredentialEnvironment) to request a token in client and token is put on authorization-header towards target


....