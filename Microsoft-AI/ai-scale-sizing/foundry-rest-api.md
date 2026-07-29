Not one single Microsoft API currently gives you everything (model availability by region/deployment type + PAYGO pricing + PTU pricing).

You typically need to combine three data sources:

1. Model availability by region and deployment type

Microsoft exposes model metadata through the Azure Cognitive Services/Foundry control plane APIs. Third-party sites such as FoundryMap are sourcing from:

GET /subscriptions/{subscriptionId}/providers/Microsoft.CognitiveServices/locations/{location}/models

The official model availability documentation is built from these same sources and shows region availability for:

Global Standard
Data Zone Standard
Regional Standard
Provisioned
Batch

For deployed resources, Foundry also provides:

GET /deployments

which returns deployments and deployment types within a project.

2. Runtime model catalog API

You can list models available from a Foundry endpoint using:

GET {endpoint}/openai/v1/models

This returns model metadata and availability information for the endpoint you're querying.

Example:

curl https://<resource>.services.ai.azure.com/openai/v1/models \
 -H "api-key: <key>"

3. Pricing (PAYGO)

For pricing, Microsoft provides the Azure Retail Prices API:

https://prices.azure.com/api/retail/prices

This is the official API used to retrieve Azure retail rates programmatically.

Example filter:

https://prices.azure.com/api/retail/prices?$filter=serviceName eq 'Azure OpenAI Service'

You can retrieve:

Model PAYGO pricing
Regional SKUs
Meter information
Currency-specific retail pricing
PTU Pricing Caveat

PTU pricing is the tricky part.

While PTUs are documented on the Azure OpenAI pricing page, Microsoft does not currently expose a simple, Foundry-specific "PTU pricing API" that tells you:

GPT-5
East US 2
Provisioned
Current PTU cost = X

in a single response. PTU pricing is generally published through Azure pricing meters and the pricing page rather than a dedicated Foundry pricing endpoint.

What I would build

If your goal is a dashboard or agent that answers:

"What models can I deploy in West US 3 today and what are the PAYGO and PTU costs?"

I'd combine:

Model Availability API

Microsoft.CognitiveServices/locations/{region}/models

Azure Retail Prices API

pricing lookup

Foundry Deployment APIs

deployment-type capability validation

This gives you a near real-time inventory across all Azure regions. Many community sites that track Foundry model availability are effectively aggregating these same underlying APIs and documentation.
