---
title: "Use Managed Identity to call an Azure Container App or App Service API"
date: 2025-01-26
author: "Vincent ter Maat"
tags: security microsoft-identity-platform managed-identities azure-entra-id
---

Microsoft Azure provides a couple of built-in security mechanisms to secure and authenticate API resources that can be provisioned during deployment.
To secure and access a WebApi using these built-in mechanisms you need to do a couple of things.

![Image]({{ site.baseurl }}/images/image.png)

## 1. Provide Entra ID access to service-principle

Your deployment service-principle needs rights to create an app-registration in Entra-ID.
Run the following powershell script to achieve this. Relevant Microsoft documentation is included at the top of the script.

```powershell
##
# This script applies MS Entra ID Graph API permissions to a service-principle so bicep can be used to deploy Entra ID resources
#
# Needed permissions are based on:
# https://learn.microsoft.com/en-us/graph/templates/reference/applications?view=graph-bicep-1.0#permissions
# https://learn.microsoft.com/en-us/graph/templates/reference/federatedidentitycredentials?view=graph-bicep-1.0#permissions
# https://learn.microsoft.com/en-us/graph/templates/reference/approleassignedto?view=graph-bicep-1.0#permissions
#
# Example: https://portal.azure.com/#view/Microsoft_AAD_IAM/ManagedAppMenuBlade/~/Permissions/objectId/1de0a50a-31fc-45e2-b056-db80c8bd5495/appId/f14038e9-b2e0-4ee4-93d0-e51e23e055d6
#
##

$DestinationTenantId = "252319a9-613a-4bf1-b184-c67f748e573a" #Noldus Tenant ID
$MsiName = "NameOfServicePrinciple" # Name of system-assigned or user-assigned managed service identity to which permissions should be granted

$oPermissions = @(
  "Application.ReadWrite.All"
  "AppRoleAssignment.ReadWrite.All"
  "Group.ReadWrite.All"
  "DelegatedPermissionGrant.ReadWrite.All"
)

$GraphAppId = "00000003-0000-0000-c000-000000000000" #Microsoft Entra ID Graph API Application ID (applies to all tenants)

$oMsi = Get-AzADServicePrincipal -Filter "displayName eq '$MsiName'"
$oGraphSpn = Get-AzADServicePrincipal -Filter "appId eq '$GraphAppId'"

$oAppRole = $oGraphSpn.AppRole | Where-Object {($_.Value -in $oPermissions) -and ($_.AllowedMemberType -contains "Application")}

Connect-MgGraph -TenantId $DestinationTenantId

foreach($AppRole in $oAppRole)
{
  $oAppRoleAssignment = @{
    "PrincipalId" = $oMSI.Id
    #"ResourceId" = $GraphAppId
    "ResourceId" = $oGraphSpn.Id
    "AppRoleId" = $AppRole.Id
  }
  
  New-MgServicePrincipalAppRoleAssignment `
    -ServicePrincipalId $oAppRoleAssignment.PrincipalId `
    -BodyParameter $oAppRoleAssignment `
    -Verbose
}
```

## 2. Enable Bicep extensibility

Add a `bicepconfig.json` that includes enablement of Bicep extensibility and use of Azure MS Graph API that is used to communicate with Entra ID.

```json
{
    "experimentalFeaturesEnabled": {
        "extensibility": true
    },
    "extensions": {  
        "microsoftGraphV1_0": "br:mcr.microsoft.com/bicep/extensions/microsoftgraph/v1.0:0.1.8-preview"  
    }
}
```

## 3. Deploy an app-registration for target

Write a bicep script that deploys an App-Registration to be referenced on the target WebApi

```bicep
//Microsoft Azure CLI Application ID for Local Development - https://learn.microsoft.com/en-us/troubleshoot/azure/entra/entra-id/governance/verify-first-party-apps-sign-in
var microsoftAzureCLIApplicationId = '04b07795-8ddb-461a-bbee-02f9e1bf7b46'

resource appRegistration 'Microsoft.Graph/applications@v1.0' = {
  uniqueName: smartAnnotatorMLApiName
  displayName: smartAnnotatorMLApiName
  signInAudience: 'AzureADMyOrg'
  api: {
    requestedAccessTokenVersion: 2
    oauth2PermissionScopes: [
      {
        adminConsentDescription: 'Allow the application to access ${WebApiName} on behalf of the signed-in user.'
        adminConsentDisplayName: 'Access ${WebApiName}'
        id: '2515e297-14c1-4139-8c52-ac09bc7eea0b'
        isEnabled: true
        type: 'User'
        userConsentDescription: 'Allow the application to access ${WebApiName} on your behalf.'
        userConsentDisplayName: 'Access ${WebApiName}'
        value: 'user_impersonation'
      }
    ]
    preAuthorizedApplications: [
			{
				appId: microsoftAzureCLIApplicationId
				delegatedPermissionIds: [
					'2515e297-14c1-4139-8c52-ac09bc7eea0b' //user_impersonation
				]
			}
		]
  }
  web: {
    homePageUrl: 'https://${WebApiName.properties.defaultHostName}'
    redirectUris: ['https://${WebApiName.properties.defaultHostName}/.auth/login/aad/callback']
    implicitGrantSettings: {
      enableAccessTokenIssuance: false
      enableIdTokenIssuance: true
    }
  }
  requiredResourceAccess: [
    {
     resourceAppId: '00000003-0000-0000-c000-000000000000'
     resourceAccess: [
       // User.Read: Sign in and read user profile
       {id: 'e1fe6dd8-ba31-4d61-89e7-88639da4683d', type: 'Scope'}
       // openid: Sign users in
       {id: '37f7f235-527c-4136-accd-4a02d197296e', type: 'Scope'}
       // profile: View users' basic profile
       {id: '14dad69e-099b-42c9-810b-d002981feec1', type: 'Scope'}
       // email: View users' email address
       {id: '64a6cdd6-aab1-4aaf-94b8-3cc8405e90d0', type: 'Scope'}
     ]
    }
  ]
}
```

Add an identifier-url which needs the app-reg ID and can only be assigned after creation:

```bicep
resource WebApiAppRegistrationIdentifierUri 'Microsoft.Graph/applications@v1.0' = {
  uniqueName: smartAnnotatorMLApiName
  displayName: smartAnnotatorMLApiName
  identifierUris: [
      'api://${appRegistration.appId}'
    ]
  }
```

## 4. Configure this App-reg on target

Adjust the bicep where you deploy your Azure WebApi to reference the App-registration deployed in previous step

```bicep
resource WebApi 'Microsoft.Web/sites@2022-09-01' = {
  name: WebApiName
  location: location
  kind: 'app,linux,container'
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true
    siteConfig: {
      acrUseManagedIdentityCreds: true
      alwaysOn: true
    }
  }
}

resource WebApiAuthSettings 'Microsoft.Web/sites/config@2022-09-01' = {
  parent: satMlWebApi
  name: 'authsettingsV2'
  kind: 'string'
  properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'RedirectToLoginPage'
      redirectToProvider: 'azureActiveDirectory'
    }
    identityProviders: {
      azureActiveDirectory: {
        enabled: true
        registration: {
          clientId: appRegistration.appId
          openIdIssuer: 'https://sts.windows.net/252319a9-613a-4bf1-b184-c67f748e573a/v2.0'
        }
        validation: {
          allowedAudiences: [
            'api://${appRegistration.appId}'
          ]
          defaultAuthorizationPolicy: {
            allowedApplications: [
                appRegistration.appId
                appServiceWebApiIdentity.properties.clientId
                microsoftAzureCLIApplicationId
              ]
          }
        }
      }
    }
    login: {
      routes: {}
      tokenStore: {
        enabled: true
        tokenRefreshExtensionHours: json('72.0')
        fileSystem: {}
        azureBlobStorage: {}
      }
      preserveUrlFragmentsForLogins: false
      cookieExpiration: {
        convention: 'FixedTime'
        timeToExpiration: '08:00:00'
      }
      nonce: {
        validateNonce: true
        nonceExpirationInterval: '00:05:00'
      }
    }
    httpSettings: {
      requireHttps: true
      routes: {
        apiPrefix: '/.auth'
      }
      forwardProxy: {
        convention: 'NoProxy'
      }
    }
  }
}
```

## 5. Enable managed identity on client

Adjust the bicep to deploy your Azure WebAPI to enable built-in OAuth authentication

```bicep
resource satMlWebApi 'Microsoft.Web/sites@2022-09-01' = {
  name: smartAnnotatorMLApiName
  location: location
  kind: 'app,linux,container'
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true
    siteConfig: {
      acrUseManagedIdentityCreds: true
      alwaysOn: true
    }
  }
}
```

## 6. Gain access from client-application

Use [DefaultAzureCredential](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential) to request a token in client put this token on authorization-header towards target.