---
title: "Scalable Security and Compliancy on Microsoft Azure"
date: 2025-02-28
author: "Vincent ter Maat"
tags: security microsoft-identity-platform managed-identities azure-entra-id
---

As cloud security becomes increasingly important, it presents a challenge to balance efficiency with maintaining a compliant and secure cloud platform. In this post, I'll explain why this is important for cloud-architects and engineers and how you can use the [Microsoft Identity Platform](https://learn.microsoft.com/en-us/entra/identity-platform/v2-overview) to support a scalable [Zero Trust Architecture](https://learn.microsoft.com/en-us/security/zero-trust/zero-trust-overview) on the Azure-platform.

Microsoft [mentions](https://learn.microsoft.com/en-us/security/benchmark/azure/introduction) the following challenge:
>The cloud moves fast, developers move fast, and attackers also move fast. How do you keep up and make sure that your cloud deployments are secure?

#### Regulations: A Changing Landscape
At December 17, 2024 the U.S. government released a [binding operational directive](https://www.cisa.gov/news-events/directives/bod-25-01-implementing-secure-practices-cloud-services) mandating that federal agencies maintain a secure configuration baseline within the cloud. This directive aims to raise the bar for cloud security, requiring stricter adherence to cloud-security best practices. In response, _Steve Faehl, Federal Security Chief Technology Officer at Microsoft_ outlined [New Microsoft guidance for the CISA Zero Trust Maturity Model](https://www.microsoft.com/en-us/security/blog/2024/12/19/new-microsoft-guidance-for-the-cisa-zero-trust-maturity-model/).

While these guidelines are more specific, they build upon existing ones like [ISO/IEC 27001:2022](https://www.iso.org/standard/27001) which many organizations already implement for security and compliance. This standard goes beyond securing cloud-environments. In this post, I'll focus specifically on securing Azure-solutions.

#### A Brief History of Cloud Security
As _Mark Simons, Lead Cybersecurity Architect at Microsoft_, [noted in 2019](https://www.microsoft.com/en-us/security/blog/2019/11/11/zero-trust-strategy-what-good-looks-like/), traditional security models rely heavily on a _"Trusted Network"_. As he mentions, this approach involves creating a strong network perimeter where the inside is trusted, and the outside is not. However, as organizations move to the cloud, this network-centric model can become increasingly complex, expensive, and inflexible - especially when applied to dynamic, microservices-based environments like those found in Azure.

These challenges with perimeter-based security have pushed the industry towards [identity-based authentication](https://learn.microsoft.com/en-us/security/benchmark/azure/mcsb-identity-management). In this paradigm, rather than relying on securing the perimeter, it relies on securing and applying Zero Trust to the resources themselves. This has resulted in the development of new [security guidelines and tools](https://learn.microsoft.com/en-us/security/benchmark/azure/). With this shift, the responsibility for choosing scalable infrastructure components (Azure-resources) and protecting these now lies within the dev-team expected to deliver business value.

#### Tools and challenges

As infrastructure and security responsibilities move into the hands of developers and architects, it’s crucial to ensure that the necessary knowledge is available within the team. It highlights the importance of [DevSecOps](https://learn.microsoft.com/en-us/devops/operate/security-in-devops) (development, security, and operations). Which is an evolution of traditional [DevOps](https://learn.microsoft.com/en-us/devops/what-is-devops), integrating security practices directly into the development pipeline.

Microsoft Azure provides several cloud-native tools and mechanisms to help achieve this. Including:
-  [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-regulatory-compliance-standards) to assess and monitor the security mechanisms that are in place on your resources
-  [Managed Identities](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview) combined with [Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview) to [provide access without secret-management](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/migrate-applications-from-secrets).
-  [App Registrations and service principals](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals)
-  Resource specific mechanisms like [built-in authentication](https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization)
-  [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/overview) to enforce certain practices

Leveraging these tools helps to mitigate security risks while maintaining flexibility and scalability.