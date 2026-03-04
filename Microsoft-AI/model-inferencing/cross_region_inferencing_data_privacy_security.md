# Cross-Region Inferencing: Data Privacy and Security

Data privacy and security details for Azure Direct Models, including Azure OpenAI, in Microsoft Foundry.

## Helpful Links

- [Data, privacy, and security for Azure Direct Models in Microsoft Foundry - Microsoft Foundry | Microsoft Learn](https://learn.microsoft.com/azure/ai-foundry/)
- [Licensing Documents](https://www.microsoft.com/licensing/docs/view/Microsoft-Products-and-Services-Data-Protection-Addendum-DPA)

---

## Technical Deep Dive: Data Protection In Transit

### 1. Encryption In Transit

All communication between the customer application, Azure AI Foundry, and Azure OpenAI model endpoints is encrypted in transit. Azure enforces strong cipher suites. (For specific TLS version and cipher requirements, see the Azure encryption in transit documentation.)

- Strong cipher suites enforced by Azure platform policy
- Certificate-based authentication
- Applies regardless of region or country

### 2. Microsoft Private Global Backbone

When Private Endpoints (Azure Private Link) are configured, network traffic between the client and the service traverses the virtual network and Microsoft's Azure backbone — eliminating exposure to the public internet. Without Private Link, TLS encryption protects data in transit over the public endpoint, but the endpoint remains publicly addressable. For regulated environments requiring provable network isolation, Private Endpoints are the correct control.

### 3. Private Connectivity Options (Optional)

For higher-assurance environments, customers can use Private Endpoints (Azure Private Link), VNet-integrated Foundry resources, outbound traffic restrictions, and Network Security Perimeters. This ensures no public IP exposure, no inbound internet access, and explicit east-west traffic control.

---

## Data Residency vs. Inference Location

These are distinct concepts that must be clearly separated for regulatory and legal analysis:

| Concept | Definition |
|---------|------------|
| Data Residency | Where data is stored at rest |
| Inference Location | Where GPU computation occurs |
| Data Sovereignty | Which legal framework governs the data |

Data stored at rest — including uploaded data — is always stored within the same geography as the customer's Foundry resource, regardless of deployment type. The inference location does not determine where data lives or which legal framework governs it.

---

## How Data Flows in This Architecture

### What Happens During Inference

1. The application in Region A sends a prompt to Azure Direct Models (Foundry)
2. The request is TLS-encrypted. If Private Endpoints are configured, traffic traverses the Microsoft Azure backbone and avoids the public internet. Public endpoints are also TLS-encrypted but use the public-facing Azure endpoint.
3. The model in Region B processes the request in memory (stateless — no data written to disk)
4. The response is returned immediately to Region A

### What Does NOT Happen

- Data is **not stored inside the model** — models are stateless; prompts and completions are not used to train, retrain, or improve base models
- Data is **not made available** to OpenAI or other Azure Direct Model providers, and is not used to improve their models or services
- Data is **not persisted** outside the customer's Azure tenant unless stateful features are explicitly enabled
- Data is **not shared** with other tenants — strict tenant isolation is enforced at the compute layer
