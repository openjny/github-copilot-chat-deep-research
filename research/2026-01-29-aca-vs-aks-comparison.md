# Azure Container Apps vs Azure Kubernetes Service Comparison

## Research Overview

### Background

Organizations deploying containerized workloads on Azure face a critical decision between Azure Container Apps (ACA) and Azure Kubernetes Service (AKS). Both services provide container orchestration capabilities, but they differ significantly in architecture, operational complexity, and target use cases. This research provides a factual comparison to support informed decision-making.

### Objectives

1. Document the architectural and feature differences between ACA and AKS
2. Identify appropriate workloads and use cases for each service
3. Compare pricing models and cost characteristics

### Perspectives

| # | Perspective | Focus |
|---|-------------|-------|
| 1 | Architecture and Feature Comparison | Infrastructure differences, capabilities, limitations, networking |
| 2 | Use Cases and Workload Suitability | Workload types, operational complexity, required skill sets |
| 3 | Pricing Model and Cost Comparison | Billing structures, scaling costs, TCO considerations |

## Conclusion

Azure Container Apps (ACA) and Azure Kubernetes Service (AKS) serve distinct purposes within the Azure container ecosystem:

**Architecture**: ACA is a serverless PaaS built on Kubernetes, KEDA, Dapr, and Envoy, abstracting all Kubernetes complexity. AKS is a managed Kubernetes service providing full API access with a shared responsibility model where customers manage node pools and workloads.

**Target Use Cases**:
- **ACA**: Event-driven workloads, HTTP APIs, microservices requiring scale-to-zero, teams without Kubernetes expertise
- **AKS**: Complex workloads requiring Kubernetes API access, Windows containers, GPU workloads, multi-tenant platforms with namespace isolation, lift-and-shift Kubernetes migrations

**Operational Complexity**: ACA requires minimal operational overhead with fully managed infrastructure. AKS requires Kubernetes expertise and active management of node upgrades, security patching, and cluster configuration.

**Pricing**:
- ACA Consumption plan offers free tiers and per-second billing, cost-effective for variable/low-utilization workloads
- ACA Dedicated plan charges $0.10/hr management plus resource consumption
- AKS charges tier-based control plane fees (Free/$73/$438 per month) plus VM costs
- At scale, AKS may be more cost-effective due to fixed VM pricing; for small/intermittent workloads, ACA with scale-to-zero provides significant savings

**Decision Factors**:
| Factor | Choose ACA | Choose AKS |
|--------|------------|------------|
| Kubernetes expertise | Not available | Available |
| Workload type | Stateless HTTP/event-driven | Complex/stateful/Windows/GPU |
| Operational preference | Minimal management | Full control |
| Scale pattern | Variable, scale-to-zero needed | Predictable, always-on |
| Cost priority | Lower entry cost | Lower unit cost at scale |

## Details

### Perspective 1: Architecture and Feature Comparison

#### Underlying Architecture

##### Azure Container Apps (ACA)

Azure Container Apps is a serverless platform that allows running containerized applications without managing underlying infrastructure[^1]. ACA is powered by Kubernetes and open-source technologies including Dapr (Distributed Application Runtime), KEDA (Kubernetes Event-driven Autoscaling), and Envoy[^2].

Key architectural characteristics:
- **Serverless abstraction**: Container Apps abstracts container orchestration APIs while providing built-in access to features like Layer 7 ingress, traffic splitting, and application lifecycle management[^3]
- **Environment concept**: Container Apps operate within an "environment" which is a secure boundary around one or more container apps and jobs. The Container Apps runtime manages each environment by handling OS upgrades, scale operations, failover procedures, and resource balancing[^4]
- **No direct Kubernetes API access**: Azure Container Apps does not provide direct access to the underlying Kubernetes APIs or control plane[^2]
- **Managed control plane**: The runtime manages critical operations like health monitoring and maintenance automatically[^1]

##### Azure Kubernetes Service (AKS)

Azure Kubernetes Service is a managed Kubernetes service that provides full access to the Kubernetes API and control plane[^5]. AKS shifts the complexity and operational overhead of managing Kubernetes to Azure while preserving access to native Kubernetes functionality.

Key architectural characteristics:
- **Managed control plane**: Azure automatically creates and configures a control plane at no cost. Azure manages the AKS control plane, responsible for Kubernetes objects and worker nodes[^5]
- **Customer-managed data plane**: The cluster resides in the customer's subscription, with cluster configurations and operations within customer control and responsibility[^2]
- **Direct Kubernetes API access**: Full access to the Kubernetes API and ability to run any Kubernetes workload[^2]
- **CNCF-certified**: AKS is CNCF-certified and compliant with SOC, ISO, PCI DSS, and HIPAA[^5]

##### Control Plane and Data Plane Differences

| Aspect | Azure Container Apps | Azure Kubernetes Service |
|--------|---------------------|--------------------------|
| Control Plane | Fully managed by Azure, not exposed to users | Managed by Azure at no cost, API access provided |
| Data Plane | Managed by Azure within environments | Customer-managed nodes in customer subscription |
| API Access | No Kubernetes API access | Full Kubernetes API access |
| Infrastructure Visibility | Abstracted from users | VMs and infrastructure visible in subscription |

#### Core Features Comparison

##### Container Orchestration Capabilities

| Feature | Azure Container Apps | Azure Kubernetes Service |
|---------|---------------------|--------------------------|
| Orchestration Model | Serverless abstraction on Kubernetes | Native Kubernetes orchestration |
| Workload Types | Container apps and jobs (on-demand, scheduled, event-driven)[^1] | Any Kubernetes workload including pods, deployments, StatefulSets, DaemonSets |
| Windows Containers | Not supported | Supported via Windows node pools[^5] |
| Multi-Container Pods | Supported (sidecars, init containers) | Full Kubernetes pod specification support |
| Custom Resource Definitions | Not supported | Fully supported |

##### Scaling Mechanisms

**Azure Container Apps Scaling:**
- Powered by KEDA (Kubernetes Event-driven Autoscaling)[^6]
- Scale rules: HTTP, TCP, and custom (KEDA scalers)[^6]
- Scale to zero supported (except for CPU/memory-based scaling)[^1]
- Minimum replicas: 0, Maximum replicas: 1,000 (configurable)[^6]
- Scaling behavior[^6]:
  - Polling interval: 30 seconds
  - Cool down period: 300 seconds
  - Scale up step: 1, 4, 8, 16, 32, ... up to maximum
  - Scale down step: 100% of replicas that need to shut down

**Azure Kubernetes Service Scaling:**
- Horizontal Pod Autoscaler (HPA): Scales pods based on CPU, memory, or custom metrics. Checks Metrics API every 15 seconds, updated every 60 seconds[^7]
- Cluster Autoscaler: Adjusts the number of nodes based on pod scheduling requirements. Checks Metrics API server every 10 seconds[^7]
- KEDA add-on: Available as an optional add-on for event-driven autoscaling[^7]
- Node Autoprovisioning (preview): Uses Karpenter to dynamically provision nodes based on pending pod requirements[^7]
- Virtual Nodes: Burst to Azure Container Instances for rapid scaling[^7]

| Scaling Feature | Azure Container Apps | Azure Kubernetes Service |
|-----------------|---------------------|--------------------------|
| Scale to Zero | Yes (HTTP/TCP/event-driven) | With KEDA add-on |
| HTTP-based Scaling | Built-in | Via HPA or KEDA |
| Event-driven Scaling | Built-in (KEDA) | KEDA add-on required |
| Node Scaling | Managed automatically | Cluster Autoscaler (manual configuration) |
| Custom Metrics | KEDA scalers supported | HPA with custom metrics, KEDA add-on |
| Vertical Pod Autoscaling | Not supported[^6] | Supported |

##### Revision Management and Deployment Strategies

**Azure Container Apps:**
- Immutable revisions with automatic versioning[^8]
- Revision modes: Single (default), Multiple, Deployment Labels (preview)[^8]
- Traffic splitting between revisions with percentage-based rules[^8]
- Labels for direct revision access via unique URLs[^8]
- Zero downtime deployment: New revision must be ready before old revision deactivation[^8]
- Change types[^8]:
  - Revision-scope changes: Create new revision (container config, scale rules)
  - Application-scope changes: Apply globally (secrets, ingress config, Dapr settings)
- Maximum 100 inactive revisions maintained[^8]

**Azure Kubernetes Service:**
- Native Kubernetes deployment strategies (RollingUpdate, Recreate)
- Blue/green and canary deployments via ingress controllers or service mesh
- Full control over deployment specifications and rollout
- kubectl rollout commands for revision management
- Custom deployment controllers supported

##### Service Discovery and Load Balancing

**Azure Container Apps:**
- Built-in DNS-based service discovery[^9]
- Container apps can be called by name within the same environment (`http://<APP_NAME>`)[^9]
- Envoy-based HTTP edge proxy for traffic routing[^10]
- Internal load balancer (ILB) for internal environments[^10]
- Automatic TLS termination at ingress point[^11]

**Azure Kubernetes Service:**
- Native Kubernetes service discovery via CoreDNS
- Service types: ClusterIP, NodePort, LoadBalancer, ExternalName
- Azure Load Balancer integration (Standard or Basic SKU)
- Ingress controllers (NGINX, Application Gateway, etc.)
- Service mesh options for advanced routing

#### Networking Capabilities

| Networking Feature | Azure Container Apps | Azure Kubernetes Service |
|--------------------|---------------------|--------------------------|
| VNet Integration | Supported (dedicated subnet required)[^10] | Supported (multiple CNI options) |
| Ingress Controller | Built-in Envoy-based, managed | User choice (NGINX, AGIC, Istio, etc.) |
| External Ingress | HTTP/HTTPS (ports 80/443)[^11] | Any port/protocol |
| TCP Ingress | Supported (VNet required for external)[^11] | Native Kubernetes support |
| Private Endpoints | Supported (workload profiles environment)[^10] | Supported |
| Network Policies | Not exposed | Full Kubernetes Network Policy support |
| User-Defined Routes | Supported (workload profiles environment)[^10] | Supported |
| NAT Gateway | Supported (workload profiles environment)[^10] | Supported |
| Azure Firewall Integration | Supported[^10] | Supported |
| Service Mesh | Dapr (managed)[^12] | Istio add-on, custom installations |
| mTLS | Peer-to-peer encryption configurable[^11] | Via service mesh or Dapr |

##### Service Mesh Support

**Azure Container Apps:**
- Dapr (Distributed Application Runtime) is natively integrated[^12]
- Dapr sidecar runs on HTTP port 3500 and gRPC port 50001[^12]
- Managed building block APIs: Service-to-service invocation, State management, Pub/sub, Bindings, Actors, Secrets, Configuration[^12]
- Automatic mTLS between Dapr-enabled apps[^12]
- No traditional service mesh like Istio supported

**Azure Kubernetes Service:**
- Istio-based service mesh add-on (managed by Microsoft)[^13]
- Features: TLS encryption, load balancing, traffic control, observability[^13]
- Custom service mesh installations supported (Istio, Linkerd, etc.)
- Dapr available as cluster extension[^14]

#### Extensibility and Customization

| Extensibility Feature | Azure Container Apps | Azure Kubernetes Service |
|-----------------------|---------------------|--------------------------|
| Kubernetes API Access | Not available[^2] | Full access |
| Custom Resource Definitions | Not supported | Fully supported |
| Helm Charts | Not applicable | Supported |
| Operators | Not applicable | Supported |
| Add-ons/Extensions | Limited (environment-level) | Core and Standard extensions[^14] |
| Dapr Integration | Built-in, managed[^12] | Available as extension[^14] |
| GitOps | Not native | Flux extension supported[^14] |

##### AKS Available Extensions

AKS provides cluster extensions for lifecycle management[^14]:
- Dapr
- Azure App Configuration
- Azure Machine Learning
- Flux (GitOps)
- Azure Container Storage
- Azure Backup for AKS
- Istio service mesh add-on[^13]
- Marketplace Kubernetes applications

##### Dapr Integration Details

**Azure Container Apps Dapr:**
- Fully managed Dapr APIs exposed through sidecar[^12]
- Tier 1 components (fully supported): Azure Cosmos DB, Azure Blob Storage, Azure Service Bus, Azure Event Hubs, Azure Key Vault[^12]
- Tier 2 components (supported with lower priority): PostgreSQL, MySQL, Redis, Kafka[^12]
- Limitations: Dapr Configuration spec not supported, Dapr not supported for jobs, actor reminders require minReplicas ≥ 1[^12]

#### Limitations

##### Azure Container Apps Limitations

- No direct access to Kubernetes APIs or control plane[^2]
- No support for Custom Resource Definitions (CRDs)
- No Kubernetes Network Policies (intra-environment communication unrestricted)[^3]
- Windows containers not supported
- Vertical scaling not supported[^6]
- Maximum 5 additional TCP ports per app[^11]
- Single security boundary per environment (no namespace-level isolation)[^3]
- Dapr not supported for jobs[^12]
- Limited customization of Istio/advanced service mesh features
- Blocked Dapr custom resources: ProxyConfig, WorkloadEntry, WorkloadGroup, IstioOperator, WasmPlugin[^12]

##### Azure Container Apps Quotas[^15]

- Maximum replicas per revision: 1,000 (configurable)
- Environment-level core quotas (Consumption, General Purpose, Memory Optimized)
- Region-scoped environment count limits

##### AKS Considerations

- Requires Kubernetes expertise for cluster management
- Customer responsible for node OS patching and upgrades (unless using auto-upgrade)
- More complex initial setup and ongoing operations
- Higher operational overhead compared to serverless options[^3]

#### Architecture and Feature Comparison Summary Table

| Aspect | Azure Container Apps | Azure Kubernetes Service |
|--------|---------------------|--------------------------|
| **Service Model** | Serverless PaaS | Managed Kubernetes (IaaS/PaaS mix) |
| **Target Audience** | Developers, teams without Kubernetes expertise | Platform teams, Kubernetes-experienced teams |
| **Control Plane** | Fully managed, hidden | Managed, API exposed |
| **Data Plane** | Managed | Customer-managed nodes |
| **Kubernetes API** | Not accessible | Full access |
| **Scaling** | KEDA-based, automatic | HPA, Cluster Autoscaler, KEDA add-on |
| **Scale to Zero** | Native | KEDA add-on required |
| **Deployment** | Revision-based | Native Kubernetes deployments |
| **Ingress** | Built-in Envoy | User-managed controllers |
| **Service Mesh** | Dapr (managed) | Istio add-on, custom options |
| **Networking** | Simplified VNet integration | Full CNI options |
| **Network Policies** | Not supported | Full support |
| **CRDs/Operators** | Not supported | Fully supported |
| **Extensions** | Environment-level | Cluster extensions |
| **Windows Containers** | Not supported | Supported |
| **Operational Effort** | Low | High |

[^1]: Azure Container Apps overview, https://learn.microsoft.com/en-us/azure/container-apps/overview

[^2]: Comparing Container Apps with other Azure container options, https://learn.microsoft.com/en-us/azure/container-apps/compare-options

[^3]: Choose an Azure container service, https://learn.microsoft.com/en-us/azure/architecture/guide/choose-azure-container-service

[^4]: Azure Container Apps environments, https://learn.microsoft.com/en-us/azure/container-apps/environment

[^5]: What is Azure Kubernetes Service (AKS)?, https://learn.microsoft.com/en-us/azure/aks/what-is-aks

[^6]: Set scaling rules in Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/scale-app

[^7]: Scaling options for applications in Azure Kubernetes Service (AKS), https://learn.microsoft.com/en-us/azure/aks/concepts-scale

[^8]: Update and deploy changes in Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/revisions

[^9]: Communicate between container apps in Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/connect-apps

[^10]: Networking in Azure Container Apps environment, https://learn.microsoft.com/en-us/azure/container-apps/networking

[^11]: Ingress in Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/ingress-overview

[^12]: Microservice APIs powered by Dapr, https://learn.microsoft.com/en-us/azure/container-apps/dapr-overview

[^13]: Istio-based service mesh add-on for Azure Kubernetes Service, https://learn.microsoft.com/en-us/azure/aks/istio-about

[^14]: Deploy and manage cluster extensions for Azure Kubernetes Service (AKS), https://learn.microsoft.com/en-us/azure/aks/cluster-extensions

[^15]: Quotas for Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/quotas

---

### Perspective 2: Use Cases and Workload Suitability

#### Recommended Workload Types

##### Azure Container Apps (ACA)

Azure Container Apps is a serverless platform optimized for the following use cases[^16]:

- **API endpoints**: Deploying HTTP-based APIs with automatic scaling
- **Background processing jobs**: Running on-demand, scheduled, or event-driven jobs
- **Event-driven processing**: Handling events from sources like queues with scale-to-zero capability
- **Microservices**: Building serverless microservices with built-in service discovery and traffic splitting

Container Apps provides serverless containers with the following characteristics[^17]:
- Optimized for general-purpose containers, especially applications spanning many microservices
- Powered by Kubernetes and open-source technologies (Dapr, KEDA, Envoy)
- Supports Kubernetes-style apps with service discovery and traffic splitting
- Enables event-driven architectures with scale based on traffic and event sources
- Supports scale to zero (except for CPU/memory-based scaling)
- Does not provide direct access to underlying Kubernetes APIs

##### Azure Kubernetes Service (AKS)

AKS is recommended for the following use cases[^18]:

- **Lift and shift to containers**: Migrate existing applications to containers in a fully managed Kubernetes environment
- **Microservices**: Simplified deployment and management with horizontal scaling, self-healing, load balancing, and secret management
- **Secure DevOps**: Implementing secure DevOps practices with Kubernetes
- **Bursting with ACI**: Using virtual nodes to provision pods in Azure Container Instances for elastic scaling
- **Machine learning model training**: Training models using large datasets with tools like TensorFlow and Kubeflow
- **Data streaming**: Ingesting and processing real-time data streams with millions of data points
- **Windows containers**: Running Windows Server containers to modernize Windows applications

AKS provides full Kubernetes API access and supports any Kubernetes workload[^17].

#### Operational Complexity

##### Azure Container Apps - Managed Responsibilities

Container Apps is a PaaS offering that abstracts container orchestration[^19]:

| Managed by Azure | Customer Responsibility |
|------------------|------------------------|
| Infrastructure provisioning | Application code and containers |
| Container orchestration | Application configuration |
| Scaling automation | Ingress settings |
| Security patching (infrastructure) | Application secrets |
| Load balancing | Application-level authentication |
| Certificate management (built-in) | Custom domain configuration |

Container Apps does not require Kubernetes expertise for operation[^17].

##### Azure Kubernetes Service - Shared Responsibility Model

AKS operates on a shared responsibility model[^20]:

**Microsoft manages (Control Plane)**:
- Kubernetes API servers (kube-apiserver)
- etcd (key-value store with automated backups every 30 minutes)
- DNS services (CoreDNS)
- Kubernetes scheduler (kube-scheduler)
- Controller manager (kube-controller-manager)
- Cloud controller manager

**Customer responsibilities (Agent Nodes)**:
- Applying OS patches and node image upgrades
- Keeping Kubernetes version within the support window
- Application workload management
- Third-party software and tools deployment
- Network policy configuration
- Custom configurations via DaemonSets

Agent nodes receive OS patches automatically, but customers must apply node image upgrades regularly[^21].

##### Day-2 Operations Comparison

| Operation | ACA | AKS |
|-----------|-----|-----|
| Kubernetes version upgrades | Managed by Azure | Customer responsibility (auto-upgrade available) |
| Node OS patching | Managed by Azure | Automatic patches, manual image upgrades |
| Certificate management | Built-in | Customer manages (manual or cert-manager) |
| Scaling | Automatic (KEDA-based) | Customer configures HPA/VPA/Cluster Autoscaler |
| Monitoring setup | Integrated Log Analytics | Requires Container Insights enablement |

AKS provides automatic upgrade channels (none, patch, stable, rapid) that can be configured to reduce operational overhead[^22].

#### Team Skill Requirements

##### Azure Container Apps

- Kubernetes expertise: **Not required**
- Container knowledge: Basic understanding needed
- Learning curve: Lower barrier to entry
- DevOps requirements: Standard CI/CD practices
- Platform engineering: Minimal overhead

Many teams prefer to start with Container Apps if they don't require direct access to Kubernetes APIs and cluster management[^17].

##### Azure Kubernetes Service

- Kubernetes expertise: **Required for effective operation**
- Container knowledge: Deep understanding recommended
- Learning curve: Significant for teams new to Kubernetes
- DevOps requirements: Kubernetes-native CI/CD pipelines
- Platform engineering: Substantial ongoing effort

AKS provides access to the Kubernetes API server, enabling customization of container orchestration[^19]. Teams unfamiliar with containerized solutions must consider the learning curve.

AKS Automatic (preview) reduces operational complexity by automating cluster management tasks, providing a more PaaS-like experience while maintaining Kubernetes flexibility[^19].

#### Integration with Azure Ecosystem

##### Azure Monitor and Log Analytics

| Feature | ACA | AKS |
|---------|-----|-----|
| Azure Monitor metrics | Integrated | Integrated |
| Log Analytics | Integrated (app logs configuration) | Requires diagnostic settings |
| Container Insights | Not applicable | Full support with containerized Azure Monitor Agent |
| Managed Prometheus | Not applicable | Full support with Azure Monitor workspace |
| Azure Managed Grafana | Not applicable | Full support for visualization |
| Live logs/metrics | Not available | Real-time streaming supported |

ACA uses built-in Log Analytics integration with minimal configuration[^24].

AKS requires explicit configuration of Container Insights and optional Prometheus metrics collection[^23].

##### Authentication and Identity

**Azure Container Apps**[^25]:
- Built-in authentication ("Easy Auth") with minimal code
- Supports Microsoft Entra ID, Facebook, GitHub, Google, X, and OpenID Connect providers
- Authentication middleware runs as sidecar container
- Managed identity support for Azure resource access

**Azure Kubernetes Service**[^26]:
- Kubernetes RBAC for cluster resource access control
- Azure RBAC integration for Kubernetes authorization
- Microsoft Entra ID integration for user authentication
- Managed identity for cluster operations and Azure resource access
- Support for Workload Identity Federation

##### CI/CD Integration

**Azure Container Apps**[^27]:
- GitHub Actions with `azure/container-apps-deploy-action`
- Supports building from Dockerfile or source code
- Managed identity for Azure Container Registry authentication
- Azure DevOps pipeline integration

**Azure Kubernetes Service**:
- GitHub Actions with multiple Kubernetes deployment actions
- Azure DevOps integration with Kubernetes service connections
- Helm chart deployments
- GitOps support with Flux (Azure Arc extension)
- Draft for AKS for preparing source code for production

##### Integration with Other Azure Services

Both services integrate with:
- Azure Container Registry (ACR)
- Azure Key Vault (secrets management)
- Azure Virtual Networks
- Azure Load Balancer (AKS) / Built-in ingress (ACA)
- Azure Service Bus and Event Hubs (event triggers)

ACA provides native integration with Dapr for microservices communication patterns[^16].

#### Migration Scenarios

##### When to Choose ACA over AKS

- Teams without Kubernetes expertise seeking simplified container deployment
- Event-driven workloads requiring scale-to-zero
- Microservices architectures not requiring Kubernetes API access
- Applications prioritizing operational simplicity over infrastructure control
- Workloads with predictable, HTTP-based traffic patterns

##### When to Choose AKS over ACA

- Workloads requiring direct Kubernetes API access
- Complex scheduling requirements or custom controllers
- Multi-tenant clusters with namespace isolation
- Stateful applications requiring persistent storage control
- Windows container workloads
- GPU workloads for machine learning
- Regulatory requirements mandating specific infrastructure configurations

##### Portability Considerations

| Aspect | ACA | AKS |
|--------|-----|-----|
| Container image portability | High | High |
| Configuration portability | Azure-specific (ARM/Bicep/YAML) | Standard Kubernetes manifests |
| Vendor lock-in risk | Higher (Azure-specific abstractions) | Lower (standard Kubernetes) |
| Multi-cloud flexibility | Limited | Kubernetes ecosystem compatible |

AKS clusters are CNCF-certified and compliant with Kubernetes standards[^18], enabling portability to other Kubernetes environments.

#### Decision Matrix

| Scenario | Recommended Service | Rationale |
|----------|---------------------|-----------|
| Simple web APIs with variable traffic | ACA | Built-in scaling, minimal operations |
| Event-driven microservices | ACA | Native KEDA integration, scale-to-zero |
| Background job processing | ACA | Jobs feature with scheduled/event triggers |
| Complex microservices requiring K8s APIs | AKS | Full Kubernetes control plane access |
| Windows container workloads | AKS | Native Windows node pool support |
| Machine learning training workloads | AKS | GPU node pools, specialized schedulers |
| Lift-and-shift Kubernetes migrations | AKS | Compatible Kubernetes manifests |
| Multi-team platform with isolation | AKS | Namespace-based access control |
| Rapid prototyping and MVP development | ACA | Simplified deployment, lower overhead |
| Strict compliance requirements | AKS | Granular infrastructure control |

---

[^16]: Azure Container Apps overview, https://learn.microsoft.com/en-us/azure/container-apps/overview

[^17]: Comparing Container Apps with other Azure container options, https://learn.microsoft.com/en-us/azure/container-apps/compare-options

[^18]: What is Azure Kubernetes Service (AKS)?, https://learn.microsoft.com/en-us/azure/aks/what-is-aks

[^19]: Choose an Azure container service, https://learn.microsoft.com/en-us/azure/architecture/guide/choose-azure-container-service

[^20]: Support policies for Azure Kubernetes Service, https://learn.microsoft.com/en-us/azure/aks/support-policies

[^21]: AKS support coverage for agent nodes, https://learn.microsoft.com/en-us/azure/aks/support-policies#aks-support-coverage-for-agent-nodes

[^22]: Automatically upgrade an Azure Kubernetes Service (AKS) cluster, https://learn.microsoft.com/en-us/azure/aks/auto-upgrade-cluster

[^23]: Monitor Azure Kubernetes Service (AKS), https://learn.microsoft.com/en-us/azure/aks/monitor-aks

[^24]: Observability in Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/observability

[^25]: Authentication and authorization in Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/authentication

[^26]: Access and identity options for Azure Kubernetes Service (AKS), https://learn.microsoft.com/en-us/azure/aks/concepts-identity

[^27]: Deploy to Azure Container Apps with GitHub Actions, https://learn.microsoft.com/en-us/azure/container-apps/github-actions

---

### Perspective 3: Pricing Model and Cost Comparison

> **Note:** Pricing information is based on data available as of January 2026 and is subject to change. Always verify current pricing at the official Azure pricing pages.

#### Pricing Structure

##### Azure Container Apps (ACA)

ACA offers two pricing plans based on workload profile selection[^28][^29]:

**Consumption Plan:**
- Per-second billing based on resource allocation and requests
- Monthly free grants per subscription:
  - 180,000 vCPU-seconds
  - 360,000 GiB-seconds
  - 2 million HTTP requests
- Active usage rates (Pay-as-you-go):
  - vCPU: $0.000024 per second
  - Memory: $0.000003 per GiB-second
- Idle usage rates (reduced when replicas are at minimum count and inactive):
  - vCPU: $0.000003 per second
  - Memory: $0.000003 per GiB-second
- Serverless GPU (consumption-based):
  - NC T4 v3: $0.00009 per second
  - NC A100 v4: $0.000651 per second
- Requests: $0.40 per million (after free tier)

**Dedicated Plan:**
- Single-tenancy guarantee with access to specialized hardware
- Billing based on vCPU-seconds and GiB-seconds across instances
- Management fee: $0.10 per hour
- vCPU: $0.0571 per hour
- Memory: $0.0050 per GiB-hour
- No idle billing model - charged for provisioned resources

##### Azure Kubernetes Service (AKS)

AKS offers tiered pricing for cluster management plus underlying VM costs[^30]:

**Control Plane Tiers:**

| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | No SLA, up to 1,000 nodes (10 recommended) |
| Standard | $73/cluster/month | Financially-backed API server uptime SLA, up to 5,000 nodes |
| Premium (LTS) | $438/cluster/month | Long-term support with 2 years extended Kubernetes support |

**AKS Automatic:**
- Control plane: $116.80/cluster/month
- Additional per-vCPU charges based on VM type:
  - General Purpose: $7.05/vCPU/month
  - Compute Optimized: $10.96/vCPU/month
  - Memory Optimized: $11.16/vCPU/month
  - Storage Optimized: $13.01/vCPU/month
  - Confidential Compute: $10.88/vCPU/month
  - High Performance Compute: $6.57/vCPU/month
  - GPU Accelerated: $32.29/vCPU/month

**Node Pool Costs:**
- VMs are billed separately based on VM type, size, and duration
- Example D-series VMs (East US, pay-as-you-go):
  - D2s v5 (2 vCPU, 8 GiB): ~$70.08/month
  - D4s v5 (4 vCPU, 16 GiB): ~$140.16/month
  - D8s v5 (8 vCPU, 32 GiB): ~$280.32/month

#### Cost Component Comparison

| Cost Component | Azure Container Apps | Azure Kubernetes Service |
|----------------|---------------------|-------------------------|
| Control Plane | Included in resource charges | Free tier: $0; Standard: $73/month; Premium: $438/month |
| Compute (Pay-as-you-go) | Per-second: vCPU $0.000024/sec, Memory $0.000003/GiB-sec | VM-based pricing (e.g., D2s v5: ~$70/month) |
| Dedicated Compute | vCPU $0.0571/hr, Memory $0.0050/GiB-hr | VM pricing applies |
| Management Fee | Dedicated plan only: $0.10/hour | Included in tier pricing |
| HTTP Requests | $0.40/million (first 2M free) | Not charged separately |
| Monthly Free Grants | 180,000 vCPU-sec, 360,000 GiB-sec, 2M requests | None |
| SLA | Included with service | Requires Standard or Premium tier |

#### Networking Costs

Both services incur additional Azure networking charges[^31][^32]:

**Load Balancer (Standard):**
- First 5 rules: $0.025/hour
- Additional rules: $0.01/rule/hour
- Data processed: $0.005/GB
- Inbound NAT rules: Free

**Bandwidth/Egress:**
- Data transfer in: Free
- Data transfer within same Availability Zone: Free
- Internet Egress (North America/Europe): First 100GB free, then $0.087/GB
- Inter-region (within North America/Europe): $0.02/GB

**ACA-specific:**
- Ingress from outside Container Apps environment is billable (request charges)
- Internal traffic between apps in same environment is not billable

**AKS-specific:**
- Public IP addresses may incur additional charges
- Internal load balancers for private clusters

#### Storage Costs

Storage is billed separately for both services:

**AKS Node Storage:**
- Managed disks billed separately from VMs
- Premium SSD, Standard SSD, and Standard HDD options available

**ACA:**
- Temporary storage included with container instances
- Persistent storage requires Azure Files or other Azure storage services

#### Monitoring and Logging Costs

Both services utilize Azure Monitor for observability[^33][^34]:

**Azure Monitor Log Analytics:**
- Analytics Logs: $2.30/GB ingested (Pay-as-you-go)
- Basic Logs: $0.50/GB ingested
- Auxiliary Logs: $0.05/GB ingested
- Commitment tiers offer discounts (15-36% savings for 100GB-50TB/day)
- Retention: 31-90 days included; extended retention $0.10-$0.02/GB/month

**Container Insights (for AKS):**
- Uses Log Analytics pricing
- Managed Prometheus available for cost-effective metrics collection
- Data filtering options available to reduce ingestion costs

**ACA:**
- OpenTelemetry agent runs at no extra compute cost when enabled
- Log data sent to Azure Monitor charged at standard rates

#### Scaling Cost Characteristics

**Azure Container Apps:**
- Scale-to-zero: No usage charges when scaled to zero replicas[^29]
- Idle billing: Reduced rates when replicas are at minimum count and inactive
- Active billing threshold: vCPU usage > 0.01 cores or data received > 1,000 bytes/sec
- Jobs: Billed at active rate from start to completion; no charges when not running

**Azure Kubernetes Service:**
- Minimum 1 node required per node pool (cannot scale to zero nodes)
- Node autoscaling available but always incurs base VM costs
- KEDA can scale pod counts to zero, but underlying nodes still incur charges
- Cluster autoscaler can reduce node count but not below minimum

#### Cost Optimization Options

**ACA:**
- Azure Savings Plan for Compute: 15-17% savings with 1-3 year commitment
- Scale-to-zero for intermittent workloads
- Use Consumption plan for variable workloads
- Use Dedicated plan for predictable, high-utilization workloads

**AKS:**
- Azure Reserved VM Instances: Up to 72% savings with 1-3 year commitment
- Azure Savings Plan for Compute: 15-30% savings
- Spot VMs: Up to 90% discount for interruptible workloads
- Right-sizing node pools based on workload requirements
- Cluster autoscaler to match capacity to demand

#### Total Cost of Ownership (TCO) Considerations

**Operational Costs:**

| Factor | Azure Container Apps | Azure Kubernetes Service |
|--------|---------------------|-------------------------|
| Required Expertise | Basic container knowledge | Kubernetes administration expertise required |
| Cluster Management | Fully managed | User manages node pools, upgrades, security |
| Networking Configuration | Simplified ingress | Full control requires networking expertise |
| Security Patching | Managed by Azure | User responsible for node OS updates |
| Monitoring Setup | Built-in basic monitoring | Requires Container Insights configuration |

**Hidden Costs to Consider:**

- **ACA:** Private endpoints, planned maintenance, and custom VNet features require Dedicated Plan Management charges regardless of Consumption vs Dedicated plan selection
- **AKS:** Node pool VMs continue running even during low demand; requires proactive scaling management to optimize costs

#### Pricing Scenario Examples

**Scenario 1: Small Web Application**
- Requirements: 1 vCPU, 2 GiB memory, ~100,000 requests/month, 8 hours/day active

*Azure Container Apps (Consumption):*
- Monthly vCPU-seconds: 1 × 8 × 3600 × 22 = 633,600 seconds
  - Billable after free tier: 633,600 - 180,000 = 453,600 seconds
  - Cost: 453,600 × $0.000024 = ~$10.89
- Monthly GiB-seconds: 2 × 8 × 3600 × 22 = 1,267,200 GiB-seconds
  - Billable after free tier: 1,267,200 - 360,000 = 907,200 GiB-seconds
  - Cost: 907,200 × $0.000003 = ~$2.72
- Requests: Free (under 2M)
- **Estimated Total: ~$14/month**

*Azure Kubernetes Service (Standard tier):*
- Control plane: $73/month
- Minimum 1 node (D2s v5): ~$70/month
- **Estimated Total: ~$143/month**

**Scenario 2: Production Microservices**
- Requirements: 10 services, average 4 vCPU/16 GiB per service, 24/7 operation

*Azure Container Apps (Dedicated):*
- Management: $0.10 × 730 hours = $73
- vCPU: 40 vCPU × 730 hours × $0.0571 = ~$1,667
- Memory: 160 GiB × 730 hours × $0.005 = ~$584
- **Estimated Total: ~$2,324/month**

*Azure Kubernetes Service (Standard tier):*
- Control plane: $73/month
- Node pool (e.g., 5× D8s v5 with 8 vCPU/32 GiB each): 5 × $280 = ~$1,400/month
- **Estimated Total: ~$1,473/month**

*Notes:*
- AKS can be more cost-effective at scale due to fixed VM pricing
- ACA Dedicated plan pricing scales linearly with consumption
- Additional costs apply for networking, storage, and monitoring for both

---

[^28]: Azure Container Apps Pricing, https://azure.microsoft.com/en-us/pricing/details/container-apps/

[^29]: Billing in Azure Container Apps, https://learn.microsoft.com/en-us/azure/container-apps/billing

[^30]: Azure Kubernetes Service (AKS) Pricing, https://azure.microsoft.com/en-us/pricing/details/kubernetes-service/

[^31]: Azure Load Balancer Pricing, https://azure.microsoft.com/en-us/pricing/details/load-balancer/

[^32]: Azure Bandwidth Pricing, https://azure.microsoft.com/en-us/pricing/details/bandwidth/

[^33]: Azure Monitor Pricing, https://azure.microsoft.com/en-us/pricing/details/monitor/

[^34]: Optimize monitoring costs for Container insights, https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-cost
