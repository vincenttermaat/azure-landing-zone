---
title: "Use Managed Identity to call an Azure Container App or App Service API"
date: 2025-01-26
author: "Vincent ter Maat"
tags: security microsoft-identity-platform managed-identities azure-entra-id
---

In order to leverage the Azure cloud-native security mechanisms to secure and authenticate API resources including deployment you need to:

{:toc}

# 1. Enable Bicep extensibility

Add a bicep.config that includes enablement of Bicep extensibility and use of Azure MS Graph API that is used to communicate with Entra ID.

# 2. Deploy an app-reg for target

Write a bicep script that deploys an app-Registration to be referenced on the target Webapp-api

# 3. Configure this App-reg on target

Adjust the bicep where you deploy your Azure WebApi to reference the App-registration deployed in previous step

# 4. Enable managed identity on client

Adjust the bicep to deploy your Azure WebAPI to enable built-in OAuth authentication

# 5. Grant Managed Identity of client access to App-Registration

# 6. Gain access from client-application

Use [DefaultAzureCredential](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredentialEnvironment) to request a token in client put this token on authorization-header towards target.

7. Configure your local development envrionment

Add 
....