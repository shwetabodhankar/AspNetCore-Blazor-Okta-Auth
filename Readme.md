# Azure App Service Landing Zone Accelerator - Implementation Checklist

> **Reference Architecture**: [App Service Secure Baseline - Multi-tenant](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/scenarios/secure-baseline-multitenant/README.md)
>
> **Last Updated**: May 6, 2026
>
> **Purpose**: Comprehensive checklist for modernizing on-premises applications to Azure using App Service with secure baseline architecture

---

## Table of Contents
1. [Identity & Access Management](#1-identity--access-management)
2. [Networking - Hub & Spoke Topology](#2-networking---hub--spoke-topology)
3. [Front-End & Edge Services](#3-front-end--edge-services)
4. [Compute - App Service](#4-compute---app-service)
5. [Data Services - Azure SQL Database](#5-data-services---azure-sql-database)
6. [Caching - Azure Cache for Redis](#6-caching---azure-cache-for-redis)
7. [AI Services - Azure OpenAI](#7-ai-services---azure-openai-optional)
8. [Secrets Management - Azure Key Vault](#8-secrets-management---azure-key-vault)
9. [Monitoring & Observability](#9-monitoring--observability)
10. [Security & Compliance](#10-security--compliance)
11. [DNS Configuration](#11-dns-configuration)
12. [Disaster Recovery & Backup](#12-disaster-recovery--backup)
13. [Deployment & CI/CD](#13-deployment--cicd)
14. [Cost Optimization](#14-cost-optimization)
15. [Governance & Tagging](#15-governance--tagging)
16. [Summary & Timeline](#summary-checklist)

---

## 1. IDENTITY & ACCESS MANAGEMENT

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Configure Identity Provider | Microsoft Entra ID | Authentication for internal apps and B2B scenarios | Set up tenant, configure app registrations, define roles | Tenant: `contoso.onmicrosoft.com`<br>App Registration: `webapp-prod` | ☐ |
| Alternative: Configure B2C | Microsoft Entra ID B2C | Authentication for B2C (customer-facing) scenarios | Create B2C tenant, configure user flows, custom policies | B2C Tenant: `contosocustomers.onmicrosoft.com`<br>User Flow: `SignUpSignIn` | ☐ |
| Enable Managed Identities | System/User Assigned Identity | Service-to-service authentication without credentials | Enable on App Service, grant RBAC permissions | Identity Name: `appservice-identity-prod`<br>Type: `System-assigned` | ☐ |
| Configure RBAC | Azure RBAC | Fine-grained access control to Azure resources | Assign roles to users/groups/service principals | Role: `Contributor`<br>Scope: `/subscriptions/{sub-id}/resourceGroups/rg-prod` | ☐ |

### Key Recommendations:
- ✅ Use **Microsoft Entra ID** for internal/B2B scenarios
- ✅ Use **Microsoft Entra ID B2C** for customer-facing (B2C) applications
- ✅ Enable **Managed Identities** everywhere possible to eliminate credential management
- ✅ Follow **principle of least privilege** for all RBAC assignments

---

## 2. NETWORKING - HUB & SPOKE TOPOLOGY

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Create Hub VNet | Virtual Network | Central hub for shared services and connectivity | Deploy hub VNet with gateway subnet, bastion subnet, firewall subnet | Name: `vnet-hub-prod`<br>Address Space: `10.0.0.0/16`<br>Region: `East US 2` | ☐ |
| Create Spoke VNet | Virtual Network | Isolated network for App Service workload | Deploy spoke VNet with app, data, and PE subnets | Name: `vnet-spoke-appservice-prod`<br>Address Space: `10.1.0.0/16` | ☐ |
| Configure VNet Peering | VNet Peering | Connect Hub and Spoke networks | Enable bidirectional peering, allow gateway transit | Hub to Spoke: `peer-hub-to-spoke`<br>Spoke to Hub: `peer-spoke-to-hub` | ☐ |
| Create App Subnet | Subnet | Dedicated subnet for App Service VNet integration | Delegate subnet to Microsoft.Web/serverFarms | Name: `snet-appservice`<br>Address Range: `10.1.1.0/24`<br>Delegation: `Microsoft.Web/serverFarms` | ☐ |
| Create Private Endpoint Subnet | Subnet | Subnet for private endpoints | Disable private endpoint network policies | Name: `snet-privateendpoints`<br>Address Range: `10.1.2.0/24` | ☐ |
| Create Data Subnet | Subnet | Subnet for data services | Configure NSG for data tier | Name: `snet-data`<br>Address Range: `10.1.3.0/24` | ☐ |
| Configure Network Security Groups | NSG | Control inbound/outbound traffic at subnet level | Define allow/deny rules for protocols and ports | NSG Name: `nsg-appservice`<br>Rules: Allow 443 from AFD, Deny all inbound | ☐ |
| Enable DDoS Protection | DDoS Network Protection | Protect against DDoS attacks | Enable Standard DDoS Protection on spoke VNet | Plan Name: `ddos-protection-plan`<br>Apply to: `vnet-spoke-appservice-prod` | ☐ |

### Key Recommendations:
- ✅ Use **Hub-Spoke topology** for governance, security, and routing
- ✅ **Delegate App Service subnet** to `Microsoft.Web/serverFarms`
- ✅ Enable **Standard DDoS Protection** for production environments
- ✅ Apply **NSGs** to enforce network segmentation and least privilege

### Network Topology:
```
Hub VNet (10.0.0.0/16)
  ├── Gateway Subnet
  ├── Bastion Subnet
  └── Firewall Subnet
      ↓ VNet Peering
Spoke VNet (10.1.0.0/16)
  ├── App Subnet (10.1.1.0/24) - Delegated to App Service
  ├── Private Endpoint Subnet (10.1.2.0/24)
  └── Data Subnet (10.1.3.0/24)
```

---

## 3. FRONT-END & EDGE SERVICES

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Deploy Azure Front Door | Azure Front Door Premium | Global load balancer, CDN, and security layer | Deploy Premium tier for Private Link support | Name: `afd-myapp-prod`<br>SKU: `Premium`<br>Endpoint: `myapp.azurefd.net` | ☐ |
| Configure WAF Policy | Web Application Firewall | Protect against web vulnerabilities and attacks | Enable Microsoft-managed ruleset, set to Prevention mode | Policy Name: `waf-appservice-prod`<br>Mode: `Prevention`<br>Ruleset: `Microsoft Default 2.1` | ☐ |
| Associate WAF to Front Door | WAF Association | Apply WAF protection to all domains | Link WAF policy to Front Door security policy | Security Policy: `default-security-policy`<br>Domains: `All` | ☐ |
| Configure Custom Domains | Custom Domain | Use branded domain names | Add custom domains, enable HTTPS with managed certs | Domain: `www.contoso.com`<br>Certificate: `Azure Managed` | ☐ |
| Enable Caching | Front Door Caching | Improve performance and reduce origin load | Configure caching rules for static content | Cache Duration: `1 hour`<br>Query String: `Include specified` | ☐ |
| Configure Origin | Front Door Origin | Backend App Service connection | Point to App Service, configure health probes | Origin Host: `app-myapp-prod.azurewebsites.net`<br>Priority: `1`, Weight: `1000` | ☐ |
| Optional: Private Link to App Service | Private Link Service | Secure origin connection (Premium only) | Configure Private Link from AFD to App Service | Private Endpoint: `pe-afd-to-appservice` | ☐ |

### Key Recommendations:
- ✅ Deploy **Azure Front Door Premium** for global scale and Private Link support
- ✅ Enable **WAF in Prevention mode** to block malicious traffic
- ✅ Use **Azure-managed certificates** to prevent expiration issues
- ✅ Enable **caching** to improve performance and reduce origin load
- ✅ Azure Front Door is **globally resilient** to zone and region outages

### Azure Government Cloud:
> ⚠️ Azure Front Door Premium is **not available** in Azure Government cloud. Use **Azure Application Gateway** instead.

---

## 4. COMPUTE - APP SERVICE

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Deploy App Service Plan | App Service Plan | Hosting plan for web apps | Deploy Premium v3 tier for production, zone-redundant | Name: `asp-myapp-prod`<br>SKU: `P1v3`<br>Zone Redundancy: `Enabled`<br>OS: `Linux/Windows` | ☐ |
| Deploy App Service | App Service | Web application hosting | Create app service with runtime stack | Name: `app-myapp-prod`<br>Runtime: `.NET 8`, `Node 20`, `Python 3.11`<br>Region: `East US 2` | ☐ |
| Enable VNet Integration | VNet Integration | Connect App Service to VNet for outbound traffic | Integrate with delegated app subnet | VNet: `vnet-spoke-appservice-prod`<br>Subnet: `snet-appservice` | ☐ |
| Configure Access Restrictions | Access Restrictions | Allow only Azure Front Door traffic | Restrict to specific AFD instance, block direct access | Allow from: `Azure Front Door`<br>Service Tag: `AzureFrontDoor.Backend`<br>Header: `X-Azure-FDID` | ☐ |
| Enable HTTPS Only | TLS Configuration | Force HTTPS for all connections | Set minimum TLS version to 1.2 | HTTPS Only: `Enabled`<br>Min TLS: `1.2` | ☐ |
| Configure Managed Identity | Managed Identity | Enable passwordless authentication | Enable system-assigned managed identity | Identity Type: `System-assigned`<br>Object ID: `{generated}` | ☐ |
| Configure Deployment Slots | Deployment Slots | Enable zero-downtime deployments | Create staging slot with auto-swap | Slots: `staging`, `production`<br>Auto-swap: `Enabled from staging` | ☐ |
| Enable Always On | App Settings | Keep app always loaded | Enable Always On for production workloads | Always On: `Enabled` | ☐ |
| Configure App Settings | Configuration | Environment variables and connection strings | Store non-sensitive config, reference Key Vault for secrets | `WEBSITE_NODE_DEFAULT_VERSION`: `20`<br>`KeyVault_URI`: `@Microsoft.KeyVault(...)` | ☐ |
| Enable Defender for App Service | Microsoft Defender | Threat detection and security monitoring | Enable Defender for Cloud on subscription | Plan: `Microsoft Defender for App Service`<br>Coverage: `All App Services` | ☐ |

### Key Recommendations:
- ✅ Use **Premium v3** tier for production (zone redundancy, better performance)
- ✅ Configure **Access Restrictions** to only allow traffic from Azure Front Door
- ✅ Use **VNet Integration** for secure outbound connectivity to Azure services
- ✅ Use **Deployment Slots** for zero-downtime deployments
- ✅ Enable **Defender for App Service** for threat detection

---

## 5. DATA SERVICES - AZURE SQL DATABASE

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Deploy SQL Server | Azure SQL Logical Server | Managed SQL server instance | Create with Entra ID admin, disable public access | Name: `sql-myapp-prod`<br>Admin: `entra-sql-admin-group`<br>Public Access: `Disabled` | ☐ |
| Deploy SQL Database | Azure SQL Database | Relational database | Deploy with zone redundancy, appropriate tier | Name: `sqldb-myapp-prod`<br>SKU: `Standard S3` or `General Purpose`<br>Zone Redundant: `Yes` | ☐ |
| Disable Public Endpoint | Firewall | Block public access | Disable public network access | Public Network Access: `Disabled`<br>Firewall: `No rules` | ☐ |
| Create Private Endpoint | Private Endpoint | Private connectivity to SQL | Deploy PE in private endpoint subnet | PE Name: `pe-sql-myapp-prod`<br>Target: `sqlServer`<br>Subnet: `snet-privateendpoints` | ☐ |
| Configure Private DNS | Private DNS Zone | DNS resolution for private endpoint | Create privatelink.database.windows.net zone | Zone: `privatelink.database.windows.net`<br>Link to: `vnet-spoke-appservice-prod` | ☐ |
| Enable TDE | Transparent Data Encryption | Encrypt data at rest | Use Microsoft-managed or customer-managed key | TDE: `Enabled`<br>Key: `Microsoft-managed` / `BYOK from Key Vault` | ☐ |
| Configure Entra ID Authentication | Entra ID Integration | Use managed identity for DB access | Set Entra admin, grant permissions to app identity | Entra Admin: `sql-admin-group`<br>Grant to: `appservice-identity-prod` | ☐ |
| Enable Advanced Threat Protection | Microsoft Defender for SQL | Detect vulnerabilities and threats | Enable Defender for Azure SQL | Assessment: `Enabled`<br>Alerts: `Enabled` | ☐ |
| Configure Backup Retention | Backup Policy | Point-in-time restore capability | Set retention period for backups | Short-term: `7-35 days`<br>Long-term: `Optional LTR policy` | ☐ |

### Key Recommendations:
- ✅ **Disable public endpoint** - use private endpoints only
- ✅ Enable **Transparent Data Encryption (TDE)** for data at rest
- ✅ Use **Entra ID authentication** with managed identities (passwordless)
- ✅ Enable **Microsoft Defender for SQL** for vulnerability assessment and threat detection
- ✅ Configure **backup retention** based on business requirements

### Connection String (Passwordless):
```csharp
Server=sql-myapp-prod.database.windows.net;
Database=sqldb-myapp-prod;
Authentication=Active Directory Default;
```

---

## 6. CACHING - AZURE CACHE FOR REDIS

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Deploy Redis Cache | Azure Cache for Redis | High-performance distributed cache | Deploy Premium tier for VNet support | Name: `redis-myapp-prod`<br>SKU: `Premium P1`<br>Zone Redundancy: `Yes` | ☐ |
| Disable Public Access | Network Configuration | Block public endpoint | Set publicNetworkAccess to Disabled | Public Network Access: `Disabled` | ☐ |
| Create Private Endpoint | Private Endpoint | Private connectivity to Redis | Deploy PE for Redis (one PE for clustered cache) | PE Name: `pe-redis-myapp-prod`<br>Target: `redisCache`<br>Subnet: `snet-privateendpoints` | ☐ |
| Configure Private DNS | Private DNS Zone | DNS resolution for private endpoint | Create privatelink.redis.cache.windows.net zone | Zone: `privatelink.redis.cache.windows.net`<br>Link to: `vnet-spoke-appservice-prod` | ☐ |
| Configure Access from App Service | Network Access | Allow App Service managed identity | Use connection string with identity authentication | Connection: Use managed identity or access keys from Key Vault | ☐ |

### Key Recommendations:
- ✅ Use **Premium tier** for VNet support and zone redundancy
- ✅ **Disable public access** - use private endpoints only
- ✅ For **clustered cache**, only one private endpoint connection is allowed
- ✅ Store connection strings in **Key Vault**

---

## 7. AI SERVICES - AZURE OPENAI (Optional)

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Deploy Azure OpenAI | Azure OpenAI Service | AI language models (GPT-4, GPT-3.5) | Create cognitive services account | Name: `openai-myapp-prod`<br>SKU: `S0`<br>Region: `East US 2` | ☐ |
| Deploy Models | Model Deployment | Deploy specific AI models | Deploy required models (GPT-4, embeddings) | Model: `gpt-4`<br>Deployment: `gpt-4-deployment`<br>Capacity: `10K TPM` | ☐ |
| Disable Public Access | Network Configuration | Block public endpoint | Configure network access | Public Network Access: `Disabled` | ☐ |
| Create Private Endpoint | Private Endpoint | Private connectivity to OpenAI | Deploy PE for OpenAI | PE Name: `pe-openai-myapp-prod`<br>Target: `account`<br>Subnet: `snet-privateendpoints` | ☐ |
| Configure Private DNS | Private DNS Zone | DNS resolution for private endpoint | Create privatelink.openai.azure.com zone | Zone: `privatelink.openai.azure.com`<br>Link to: `vnet-spoke-appservice-prod` | ☐ |
| Grant App Service Access | RBAC | Allow managed identity to call OpenAI | Assign Cognitive Services User role | Principal: `appservice-identity-prod`<br>Role: `Cognitive Services OpenAI User` | ☐ |

### Key Recommendations:
- ✅ Use **managed identity** for authentication (passwordless)
- ✅ **Disable public access** - use private endpoints only
- ✅ Deploy models based on your needs: GPT-4, GPT-3.5-Turbo, Embeddings
- ✅ Monitor **token usage** and set up cost alerts

---

## 8. SECRETS MANAGEMENT - AZURE KEY VAULT

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Deploy Key Vault | Azure Key Vault | Secure storage for secrets, keys, certificates | Deploy with RBAC model, purge protection | Name: `kv-myapp-prod`<br>SKU: `Standard/Premium`<br>Purge Protection: `Enabled` | ☐ |
| Disable Public Access | Network Configuration | Block public endpoint | Configure firewall and virtual networks | Public Access: `Disabled`<br>Allow: `None` | ☐ |
| Create Private Endpoint | Private Endpoint | Private connectivity to Key Vault | Deploy PE for Key Vault | PE Name: `pe-kv-myapp-prod`<br>Target: `vault`<br>Subnet: `snet-privateendpoints` | ☐ |
| Configure Private DNS | Private DNS Zone | DNS resolution for private endpoint | Create privatelink.vaultcore.azure.net zone | Zone: `privatelink.vaultcore.azure.net`<br>Link to: `vnet-spoke-appservice-prod` | ☐ |
| Grant App Service Access | RBAC | Allow managed identity to read secrets | Assign Key Vault Secrets User role | Principal: `appservice-identity-prod`<br>Role: `Key Vault Secrets User` | ☐ |
| Store Secrets | Secrets | Store connection strings and sensitive data | Add SQL, Redis, OpenAI connection strings | Secret: `sqldb-connectionstring`<br>Value: `Server=sql-myapp-prod...` | ☐ |
| Configure App Service Integration | Key Vault References | Reference secrets in app settings | Use @Microsoft.KeyVault syntax | App Setting: `@Microsoft.KeyVault(SecretUri=https://kv...)` | ☐ |

### Key Recommendations:
- ✅ Use **RBAC model** (not access policies) for modern management
- ✅ Enable **purge protection** to prevent accidental deletion
- ✅ Use **Key Vault references** in App Service settings for seamless integration
- ✅ **Disable public access** - use private endpoints only

### Key Vault Reference Format:
```
@Microsoft.KeyVault(SecretUri=https://kv-myapp-prod.vault.azure.net/secrets/sqldb-connectionstring/)
@Microsoft.KeyVault(VaultName=kv-myapp-prod;SecretName=sqldb-connectionstring)
```

---

## 9. MONITORING & OBSERVABILITY

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Deploy Log Analytics Workspace | Log Analytics | Centralized log storage and analysis | Create workspace for all logs | Name: `log-myapp-prod`<br>Retention: `30-90 days`<br>Region: `East US 2` | ☐ |
| Deploy Application Insights | Application Insights | APM and application telemetry | Create Application Insights instance | Name: `appi-myapp-prod`<br>Type: `Workspace-based`<br>Workspace: `log-myapp-prod` | ☐ |
| Enable App Service Diagnostics | Diagnostic Settings | Send App Service logs to Log Analytics | Configure diagnostic logs | Logs: `AppServiceHTTPLogs`, `AppServiceConsoleLogs`<br>Destination: `log-myapp-prod` | ☐ |
| Enable SQL Diagnostics | Diagnostic Settings | Send SQL logs to Log Analytics | Configure SQL diagnostic logs | Logs: `SQLSecurityAuditEvents`, `QueryStoreRuntimeStatistics`<br>Destination: `log-myapp-prod` | ☐ |
| Enable Front Door Diagnostics | Diagnostic Settings | Send AFD logs to Log Analytics | Configure AFD diagnostic logs | Logs: `FrontDoorAccessLog`, `FrontDoorWebApplicationFirewallLog` | ☐ |
| Configure Application Insights | APM Integration | Integrate App Service with Application Insights | Add connection string to app settings | `APPLICATIONINSIGHTS_CONNECTION_STRING`: `InstrumentationKey=...` | ☐ |
| Create Alerts | Azure Monitor Alerts | Alert on critical conditions | Configure alerts for availability, errors, latency | Alert: `App Service Down`<br>Condition: `Availability < 99%`<br>Action: `Email ops team` | ☐ |
| Create Dashboard | Azure Dashboard | Visualize metrics and health | Build custom dashboard with key metrics | Dashboard: `App Service Production Health` | ☐ |

### Key Recommendations:
- ✅ Use **workspace-based Application Insights** for better log analytics integration
- ✅ Enable **diagnostic settings** on all services to centralize logs
- ✅ Configure **alerts** for proactive monitoring (availability, errors, performance)
- ✅ Create **dashboards** for at-a-glance health monitoring

### Essential Alerts:
- App Service availability < 99%
- HTTP 5xx errors > threshold
- Response time > 3 seconds
- SQL Database DTU > 80%
- Redis cache memory > 90%

---

## 10. SECURITY & COMPLIANCE

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Enable Microsoft Defender | Microsoft Defender for Cloud | Security posture and threat protection | Enable Defender plans for App Service, SQL, Key Vault | Plans: `App Service`, `Azure SQL`, `Key Vault`<br>Coverage: `All resources` | ☐ |
| Configure Security Policies | Azure Policy | Enforce compliance and governance | Assign built-in or custom policies | Initiatives: `Azure Security Benchmark`<br>Scope: `Subscription or RG` | ☐ |
| Enable Private Endpoints | Private Link | All services use private endpoints | Verify all PEs are configured and public access disabled | Status: `All data services use private endpoints` | ☐ |
| Review NSG Rules | Network Security | Principle of least privilege | Audit and minimize NSG rules | Review: `Quarterly`<br>Action: `Remove unused rules` | ☐ |
| Configure Data Encryption | TDE, Encryption at Rest | All data encrypted | SQL TDE, Storage encryption, Key Vault for keys | SQL: `TDE Enabled`<br>Storage: `Encrypted`<br>Keys: `Customer-managed` | ☐ |
| Implement RBAC | Azure RBAC | Principle of least privilege access | Review and assign minimum required roles | Review: `Quarterly`<br>Audit: `Access reviews enabled` | ☐ |
| Enable Audit Logging | Activity Logs | Track all control plane operations | Send activity logs to Log Analytics | Activity Logs → `log-myapp-prod`<br>Retention: `90 days` | ☐ |
| Data Classification | Data Protection | Classify and protect sensitive data | Identify PII, PHI, financial data | Classification: `Sensitivity labels`<br>Protection: `Encryption, access controls` | ☐ |

### Key Recommendations:
- ✅ Implement **Defense in Depth** - multiple layers of security
- ✅ Enable **Microsoft Defender for Cloud** for all services
- ✅ Use **private endpoints** for all data services
- ✅ Follow **principle of least privilege** for all access
- ✅ Enable **encryption** for data at rest and in transit
- ✅ Conduct **quarterly security reviews**

### Security Layers (Defense in Depth):
1. **Network Layer**: NSGs, DDoS Protection, Private Endpoints
2. **Identity Layer**: Entra ID, Managed Identities, RBAC
3. **Perimeter Layer**: Azure Front Door, WAF
4. **Application Layer**: App Service access restrictions, TLS 1.2+
5. **Data Layer**: SQL TDE, Private Endpoints, Entra ID authentication
6. **Monitoring Layer**: Defender for Cloud, Application Insights, Log Analytics

---

## 11. DNS CONFIGURATION

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Configure Public DNS | Azure DNS or External | Public domain resolution | Point domain to Azure Front Door endpoint | Domain: `www.contoso.com`<br>CNAME: `myapp.azurefd.net` | ☐ |
| Create Private DNS Zones | Azure Private DNS | Private endpoint DNS resolution | Create zones for all private endpoints | Zones: 7-8 zones for SQL, Redis, KV, etc. | ☐ |
| Link Private DNS to VNets | VNet Links | Enable DNS resolution in VNets | Link all private DNS zones to hub and spoke VNets | Link to: `vnet-hub-prod`, `vnet-spoke-appservice-prod` | ☐ |
| Configure DNS Zone Groups | Private Endpoint DNS | Auto-create DNS records for PEs | Enable DNS integration on all private endpoints | Configuration: `Integrated with private DNS zones` | ☐ |

### Private DNS Zones Required:
```
privatelink.database.windows.net          (Azure SQL Database)
privatelink.redis.cache.windows.net        (Azure Cache for Redis)
privatelink.vaultcore.azure.net            (Azure Key Vault)
privatelink.openai.azure.com               (Azure OpenAI)
privatelink.azurewebsites.net              (App Service - if using Private Link)
privatelink.blob.core.windows.net          (Storage Account - if used)
privatelink.file.core.windows.net          (Storage Account - if used)
```

### Key Recommendations:
- ✅ Use **Azure DNS** for public domains (recommended)
- ✅ Create **private DNS zones** for all private endpoints
- ✅ Link private DNS zones to **both hub and spoke VNets**
- ✅ Enable **DNS zone groups** for automatic DNS record management

---

## 12. DISASTER RECOVERY & BACKUP

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Enable Zone Redundancy | Availability Zones | Deploy across availability zones | Enable zone redundancy for all services | App Service: `Zone-redundant`<br>SQL: `Zone-redundant`<br>Redis: `Zone-redundant` | ☐ |
| Configure SQL Backups | Automated Backups | Point-in-time restore capability | Configure backup retention | Short-term: `14 days`<br>Long-term: `Monthly for 12 months` | ☐ |
| Plan Multi-Region | Disaster Recovery | Optional multi-region deployment | Deploy secondary region for DR | Primary: `East US 2`<br>Secondary: `West US 2`<br>RPO/RTO: `Define` | ☐ |
| Document Recovery Procedures | Runbooks | Recovery procedures and runbooks | Create and test DR procedures | DR Plan: `Document and test quarterly` | ☐ |

### Key Recommendations:
- ✅ Enable **zone redundancy** for high availability within a region
- ✅ Configure **SQL backup retention** based on RTO/RPO requirements
- ✅ Consider **multi-region deployment** for critical workloads
- ✅ **Document and test** DR procedures regularly

### Availability Targets:
- **Zone Redundancy**: 99.95% SLA (protects against zone failures)
- **Multi-Region**: 99.99% SLA (protects against region failures)
- **Azure Front Door**: Globally resilient (no downtime from region/zone failures)

---

## 13. DEPLOYMENT & CI/CD

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Choose IaC Approach | Bicep/Terraform/ARM | Infrastructure as Code deployment | Select and implement IaC | Options: `Bicep`, `Terraform`, `ARM`<br>Recommended: `Bicep` or `Terraform` | ☐ |
| Configure Source Control | GitHub/Azure DevOps | Version control for code and IaC | Set up repositories | Repo: `github.com/myorg/myapp`<br>Branch Strategy: `main`, `develop`, `feature/*` | ☐ |
| Create CI/CD Pipelines | GitHub Actions/Azure Pipelines | Automated build and deployment | Create workflows for app and infrastructure | Pipeline: `Build → Test → Deploy to Staging → Prod` | ☐ |
| Configure Deployment Slots | Slot Swap Strategy | Blue-green deployments | Use staging slot with auto-swap or manual approval | Strategy: `Deploy to staging → validate → swap` | ☐ |
| Implement Testing | Testing Strategy | Automated testing in pipeline | Unit, integration, security tests | Tests: `Jest`, `Playwright`, `ZAP security scan` | ☐ |

### Key Recommendations:
- ✅ Use **Infrastructure as Code** for repeatable deployments
- ✅ Implement **CI/CD pipelines** for automated deployments
- ✅ Use **deployment slots** for zero-downtime deployments
- ✅ Include **automated testing** in your pipeline (unit, integration, security)

### IaC Options Comparison:
| **Option** | **Pros** | **Cons** | **Recommendation** |
|------------|----------|----------|-------------------|
| **Bicep** | Native Azure, simple syntax, good tooling | Azure-only | ✅ Best for Azure-only |
| **Terraform** | Multi-cloud, large community, mature | Learning curve | ✅ Best for multi-cloud |
| **ARM** | Native Azure, comprehensive | Verbose JSON | Use Bicep instead |

### Deployment Workflow:
```
1. Developer commits code → GitHub/Azure DevOps
2. CI Pipeline triggers: Build → Test → Package
3. CD Pipeline deploys to staging slot
4. Run automated tests against staging
5. Manual approval gate (optional)
6. Swap staging to production
7. Monitor for issues, rollback if needed
```

---

## 14. COST OPTIMIZATION

| **Task** | **Category** | **Description** | **Recommendations** | **Notes** | **Status** |
|----------|-------------|-----------------|----------------------|----------|------------|
| Right-size Resources | Capacity Planning | Match resources to actual needs | Start with smaller SKUs, scale up as needed | Monitor utilization first 30 days | ☐ |
| Use Reserved Instances | Cost Savings | Commit to 1 or 3-year terms | Consider for stable workloads | Savings: Up to 72% | ☐ |
| Configure Auto-scaling | Dynamic Scaling | Scale based on demand | Set up auto-scale rules for App Service | Scale out/in based on CPU, memory, requests | ☐ |
| Implement Caching | Performance | Reduce database and compute load | Use Front Door cache and Redis effectively | Reduces origin requests by 70-90% | ☐ |
| Monitor Costs | Cost Management | Track and optimize spending | Set budgets and alerts | Budget: $X/month, Alert at 80% | ☐ |
| Use Private Endpoint Efficiently | Network Costs | Minimize PE charges | Consolidate where possible | PE Cost: ~$7.30/month + data transfer | ☐ |

### Key Recommendations:
- ✅ **Right-size** resources based on actual usage
- ✅ Use **reserved instances** for predictable workloads (up to 72% savings)
- ✅ Enable **auto-scaling** to match demand
- ✅ Implement **caching** to reduce backend load
- ✅ Set up **cost alerts** and budgets

### Estimated Monthly Costs (Production):
| **Service** | **SKU/Configuration** | **Est. Monthly Cost** |
|-------------|----------------------|----------------------|
| App Service | Premium P1v3 (zone-redundant) | ~$170 |
| Azure Front Door | Premium + data transfer | ~$330 + usage |
| Azure SQL Database | Standard S3 (zone-redundant) | ~$200 |
| Azure Cache for Redis | Premium P1 (zone-redundant) | ~$630 |
| Azure OpenAI | GPT-4 (optional) | Pay-per-use (~$100-500) |
| Private Endpoints | 7-8 endpoints | ~$50-100 |
| Log Analytics | 5-10 GB/day | ~$50-100 |
| Key Vault | Standard | ~$5 |
| DDoS Protection | Standard plan | ~$3,000 (shared) |
| VNet, NSG, DNS | Basic networking | ~$50-100 |
| **TOTAL** | **Full production setup** | **~$1,500-2,500/month** |

> **Note**: Costs vary significantly based on usage, data transfer, and optional services like Azure OpenAI.

### Cost Optimization Tips:
1. Start with **dev/test subscriptions** for non-production (up to 55% savings)
2. Use **Azure Hybrid Benefit** if you have existing licenses
3. Enable **auto-shutdown** for non-production environments
4. Regularly review and **delete unused resources**
5. Use **Azure Advisor** recommendations for cost optimization

---

## 15. GOVERNANCE & TAGGING

| **Task** | **Service** | **Description** | **Configuration Details** | **Sample Input** | **Status** |
|----------|-------------|-----------------|---------------------------|------------------|------------|
| Define Tagging Strategy | Resource Tags | Categorize and track resources | Apply consistent tags to all resources | Environment: `Production`<br>Owner: `AppTeam`<br>CostCenter: `CC-1234`<br>Application: `MyApp` | ☐ |
| Apply Tags | Tags | Tag all resources | Use Azure Policy to enforce tagging | Required Tags: `Environment`, `Owner`, `CostCenter` | ☐ |
| Configure Resource Locks | Management Locks | Prevent accidental deletion | Apply CanNotDelete lock to production RGs | Lock Type: `CanNotDelete`<br>Scope: `Resource Groups` | ☐ |
| Create Resource Groups | Organization | Logical grouping of resources | Separate RGs by environment or function | `rg-myapp-network-prod`<br>`rg-myapp-compute-prod`<br>`rg-myapp-data-prod` | ☐ |

### Key Recommendations:
- ✅ Define and enforce **consistent tagging strategy**
- ✅ Use **Azure Policy** to enforce required tags
- ✅ Apply **resource locks** to prevent accidental deletion
- ✅ Organize resources into **logical resource groups**

### Recommended Tags:
```yaml
Environment: Production | Staging | Development
Owner: TeamName or Email
CostCenter: CC-1234
Application: MyApp
Criticality: High | Medium | Low
DataClassification: Public | Internal | Confidential | Restricted
```

### Resource Group Strategy:
```
rg-myapp-network-prod     (VNets, NSGs, Private DNS)
rg-myapp-compute-prod     (App Service, App Service Plan)
rg-myapp-data-prod        (SQL, Redis, Storage)
rg-myapp-security-prod    (Key Vault, Private Endpoints)
rg-myapp-monitoring-prod  (Log Analytics, Application Insights)
rg-myapp-frontend-prod    (Azure Front Door, WAF)
```

---

## SUMMARY CHECKLIST

| **Phase** | **Total Tasks** | **Critical Items** | **Estimated Time** |
|-----------|----------------|--------------------|--------------------|
| 1. Identity & Access | 4 | Entra ID, Managed Identity, RBAC | 1-2 days |
| 2. Networking | 8 | Hub-Spoke VNets, NSGs, DDoS Protection | 2-3 days |
| 3. Front-End Services | 7 | Azure Front Door Premium, WAF | 1-2 days |
| 4. App Service | 10 | App Service Plan, VNet Integration, Access Restrictions | 1-2 days |
| 5. Data Services | 9 | SQL Database, Private Endpoints, TDE | 2-3 days |
| 6. Caching | 5 | Redis Cache, Private Endpoint | 1 day |
| 7. AI Services | 6 | Azure OpenAI (Optional) | 1 day |
| 8. Key Vault | 7 | Key Vault, Private Endpoint, Secrets | 1 day |
| 9. Monitoring | 8 | Log Analytics, Application Insights, Alerts | 1-2 days |
| 10. Security | 8 | Defender for Cloud, Policies, Encryption | 2 days |
| 11. DNS | 4 | Public and Private DNS configuration | 1 day |
| 12. DR & Backup | 4 | Zone Redundancy, Backup Policies | 1 day |
| 13. CI/CD | 5 | IaC, Pipelines, Testing | 2-3 days |
| 14. Cost Optimization | 6 | Right-sizing, Reserved Instances | Ongoing |
| 15. Governance | 4 | Tagging, Locks, Resource Groups | 1 day |
| **TOTAL** | **95 Tasks** | **All Critical for Production** | **3-4 weeks** |

---

## DEPLOYMENT NOTES

### Prerequisites:
- ✅ Azure subscription with Owner or Contributor + User Access Administrator permissions
- ✅ Azure CLI or PowerShell installed locally
- ✅ GitHub or Azure DevOps account for source control
- ✅ Custom domain registered (if using custom domains)
- ✅ Understanding of your application's requirements (compute, storage, networking)

### Deployment Phases:

#### **Phase 1: Foundation (Week 1)**
1. Identity & Access Management setup
2. Networking (Hub-Spoke topology, subnets, NSGs)
3. Resource groups and governance

#### **Phase 2: Core Services (Week 2)**
4. App Service deployment
5. Azure SQL Database
6. Azure Cache for Redis
7. Private endpoints for all data services

#### **Phase 3: Security & Monitoring (Week 2-3)**
8. Azure Key Vault
9. Log Analytics & Application Insights
10. Microsoft Defender for Cloud
11. Private DNS configuration

#### **Phase 4: Edge & Operations (Week 3-4)**
12. Azure Front Door & WAF
13. Custom domains and certificates
14. CI/CD pipelines
15. Testing and validation

#### **Phase 5: Optimization (Week 4)**
16. Cost optimization
17. Performance tuning
18. Documentation and runbooks
19. Training and handoff

### Deployment Options:

#### **Option 1: Bicep (Recommended)**
```bash
# Clone the repository
git clone https://github.com/Azure/appservice-landing-zone-accelerator.git
cd appservice-landing-zone-accelerator/scenarios/secure-baseline-multitenant/bicep

# Login to Azure
az login

# Deploy
az deployment sub create \
  --location eastus2 \
  --template-file main.bicep \
  --parameters main.parameters.json
```

#### **Option 2: Terraform**
```bash
cd appservice-landing-zone-accelerator/scenarios/secure-baseline-multitenant/terraform

# Initialize Terraform
terraform init

# Plan deployment
terraform plan -out=tfplan

# Apply deployment
terraform apply tfplan
```

#### **Option 3: Azure Portal**
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#view/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fappservice-landing-zone-accelerator%2Fmain%2Fscenarios%2Fsecure-baseline-multitenant%2Fazure-resource-manager%2Fmain.json)

#### **Option 4: Azure Developer CLI (azd)**
```bash
# Create a new codespace or use local environment
cd appservice-landing-zone-accelerator/scenarios/secure-baseline-multitenant

# Login to Azure
azd auth login

# Deploy
azd up

# Provide environment name and subscription when prompted
```

### Post-Deployment Tasks:
1. ✅ Verify all private endpoints are connected
2. ✅ Test application connectivity through Azure Front Door
3. ✅ Validate WAF is blocking malicious requests
4. ✅ Configure custom domains and SSL certificates
5. ✅ Set up monitoring alerts and dashboards
6. ✅ Document connection strings and endpoints
7. ✅ Conduct security review
8. ✅ Perform load testing
9. ✅ Train operations team
10. ✅ Create runbooks for common operations

---

## ADDITIONAL RESOURCES

### Microsoft Documentation:
- [App Service Landing Zone Accelerator](https://github.com/Azure/appservice-landing-zone-accelerator)
- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/architecture/framework/)
- [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)
- [Defense in Depth Security](https://learn.microsoft.com/shows/azure-videos/defense-in-depth-security-in-azure)

### Architecture Diagrams:
- [Multi-tenant Architecture](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Images/Multitenant/AppServiceLandingZoneArchitecture-multitenant.png)
- [ASE Architecture](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/Images/Multitenant/AppServiceLandingZoneArchitecture-ASE.png)
- [Visio Diagram](https://github.com/Azure/appservice-landing-zone-accelerator/blob/main/docs/App-Service-LZA.vsdx)

### Best Practices Guides:
- [Front Door Best Practices](https://learn.microsoft.com/azure/frontdoor/best-practices)
- [App Service Security](https://learn.microsoft.com/azure/app-service/overview-security)
- [SQL Database Security](https://learn.microsoft.com/azure/azure-sql/database/security-best-practices)
- [Private Link Documentation](https://learn.microsoft.com/azure/private-link/private-endpoint-overview)

### Security Baselines:
- [Azure Security Benchmark](https://learn.microsoft.com/security/benchmark/azure/)
- [SQL Database Security Baseline](https://learn.microsoft.com/security/benchmark/azure/baselines/sql-database-security-baseline)
- [Redis Cache Security Baseline](https://learn.microsoft.com/security/benchmark/azure/baselines/azure-cache-for-redis-security-baseline)

---

## VERSION HISTORY

| **Version** | **Date** | **Changes** |
|-------------|----------|-------------|
| 1.0 | May 6, 2026 | Initial comprehensive checklist created |

---

## NOTES SECTION

Use this section to track decisions, issues, and lessons learned during your implementation:

```
Date: ___________
Issue/Decision: _________________________________________________
Resolution: _____________________________________________________
Impact: _________________________________________________________

Date: ___________
Issue/Decision: _________________________________________________
Resolution: _____________________________________________________
Impact: _________________________________________________________
```

---

**End of Checklist** ✅

> For questions or issues, refer to the [GitHub repository](https://github.com/Azure/appservice-landing-zone-accelerator) or consult the Azure documentation.
