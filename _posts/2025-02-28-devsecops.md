---
title: "Scalable Cloud Security and Compliancy on Microsoft Azure"
date: 2025-02-28
author: "Vincent ter Maat"
tags: security microsoft-identity-platform managed-identities azure-entra-id
---

As cloud security becomes increasingly important, it presents a challenge to balance efficiency with maintaining a compliant and secure cloud platform. This complexity led me to explore how this balance can be achieved effectively. In this post, I’ll explain why this is crucial for cloud architects and engineers and how you can use the [Microsoft Identity Platform](https://learn.microsoft.com/en-us/entra/identity-platform/v2-overview) to support a scalable [Zero Trust Architecture](https://learn.microsoft.com/en-us/security/zero-trust/zero-trust-overview) within the Azure-platform.

#### Regulations
At December 17, 2024 the U.S. government released a [binding operational directive](https://www.cisa.gov/news-events/directives/bod-25-01-implementing-secure-practices-cloud-services) mandating that federal agencies maintain a secure configuration baseline within the cloud. This directive aims to raise the bar for cloud security, requiring stricter adherence to cloud-security best practices. In response, _Steve Faehl, Federal Security Chief Technology Officer at Microsoft_ outlined [New Microsoft guidance for the CISA Zero Trust Maturity Model](https://www.microsoft.com/en-us/security/blog/2024/12/19/new-microsoft-guidance-for-the-cisa-zero-trust-maturity-model/).

While these guidelines are more specific, they build upon existing standards like [ISO/IEC 27001:2022](https://www.iso.org/standard/27001) which many organizations already implement for security and compliance. These guidelines go beyond just securing cloud-environments. In this post, I'll focus specifically on how the Microsoft Identity Platform can be used to design and build a scalable Zero Trust Architecture for Azure-solutions.

#### A Brief History of Cloud Security
As _Mark Simons, Lead Cybersecurity Architect at Microsoft_, [noted in 2019](https://www.microsoft.com/en-us/security/blog/2019/11/11/zero-trust-strategy-what-good-looks-like/), traditional security models relied heavily on a _"Trusted Network" security strategy_. As he mentions, this approach involves creating a strong network perimeter where the inside is trusted, and the outside is not. However, as organizations move to the cloud, this network-centric model is becoming increasingly complex, expensive, and inflexible - especially when applied to dynamic, microservices-based environments like those found in Azure.

These challenges with perimeter-based security have pushed the industry toward [identity-based authentication](https://learn.microsoft.com/en-us/security/benchmark/azure/mcsb-identity-management). In this paradigm, rather than relying on securing the perimeter, it relies on securing the resources themselves. This has resulted in the development of new [security guidelines and tools](https://learn.microsoft.com/en-us/security/benchmark/azure/introduction). With this shift, the responsibility for choosing scalable infrastructure components (Azure-resources) and protecting these resources now lies with the development-team expected to deliver business value.

#### Tools and challenges

As infrastructure and security responsibilities move into the hands of developers and architects, it’s crucial to ensure that the necessary knowledge is available within the team. It highlights the importance of [DevSecOps](https://learn.microsoft.com/en-us/devops/devsecops/enable-devsecops-azure-github) (development, security, and operations). Which is an evolution of traditional [DevOps](https://learn.microsoft.com/en-us/devops/what-is-devops), integrating security practices directly into the development pipeline.

Microsoft Azure provides several tools and mechanisms to ensure compliance. Examples include:
-  [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-regulatory-compliance-standards) to assess and monitor the security mechanisms that are in place on your resources
-  [Managed Identities](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview) combined with [Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview) to [provide access without secret-management](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/migrate-applications-from-secrets).
-  [App Registrations and service principals](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals)
-  Resources specific mechanisms like [built-in authentication](https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization)

 These tools become increasingly important, especially in environments where traditional network perimeters are not in place - such as in a vanilla Azure environment. Leveraging these tools helps to mitigate security risks while maintaining flexibility and scalability.

In a future post I'll show some concrete examples of how these tools and practices can be used.

In the mean time, you could already look at my blog where I show you how to [Use Managed Identity to call an Azure Container App or App Service API]({% post_url 2025-01-26-cloudnativesecurity %}) 🙌.