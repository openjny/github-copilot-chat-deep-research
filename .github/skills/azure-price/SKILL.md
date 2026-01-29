---
name: azure-price
description: Retrieve Azure service pricing information using the Azure Retail Prices API. Get detailed pricing data including regions, SKUs, and pricing types (consumption, reservation, savings plans) without authentication. Use when estimating Azure costs, comparing prices across regions or SKUs, checking reservation or savings plan pricing, or collecting data for cost analysis and budget management.
metadata:
  author: openjny
  version: "1.0"
---

# Azure Price API Skill

Retrieve Azure service pricing information using the Azure Retail Prices API without authentication.

## API Endpoint

```
https://prices.azure.com/api/retail/prices
```

### Preview Version (with Savings Plan support)

```
https://prices.azure.com/api/retail/prices?api-version=2023-01-01-preview
```

## Steps

### 1. Basic API Call

Retrieve pricing for all services:

```bash
# Basic call
curl "https://prices.azure.com/api/retail/prices"

# With jq for better readability
curl "https://prices.azure.com/api/retail/prices" | jq '.Items[0:3]'
```

### 2. Filtering

Filter by specific services or conditions using `-G` and `--data-urlencode` for proper URL encoding:

```bash
# Get Virtual Machines pricing only
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "\$filter=serviceName eq 'Virtual Machines'"

# Get reservation pricing only
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "\$filter=priceType eq 'Reservation'"

# Get Virtual Machines reservation pricing
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "\$filter=serviceName eq 'Virtual Machines' and priceType eq 'Reservation'"

# Get Compute family services
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "\$filter=serviceFamily eq 'Compute'"

# Get pricing for specific region (japaneast)
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "\$filter=armRegionName eq 'japaneast'"

# Get pricing for specific SKU
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "\$filter=armSkuName eq 'Standard_D2s_v3'"

# With jq to extract specific fields
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "\$filter=serviceName eq 'Virtual Machines' and armRegionName eq 'japaneast'" \
  | jq '.Items[0:5] | .[] | {sku: .armSkuName, price: .retailPrice, unit: .unitOfMeasure}'
```

### 3. Currency Specification

Default is USD, but other currencies are supported:

```bash
# Get prices in Euro
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "currencyCode=EUR" \
  --data-urlencode "\$filter=serviceFamily eq 'Compute'"

# Get prices in Japanese Yen
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "currencyCode=JPY" \
  --data-urlencode "\$filter=serviceFamily eq 'Compute'"

# With jq to show currency in output
curl -G "https://prices.azure.com/api/retail/prices" \
  --data-urlencode "currencyCode=JPY" \
  --data-urlencode "\$filter=serviceName eq 'Virtual Machines' and armSkuName eq 'Standard_D2s_v3'" \
  | jq '.Items[0:3] | .[] | {sku: .armSkuName, price: .retailPrice, currency: .currencyCode, type: .type}'
```

### 4. Primary Meter Only

Filter for primary meters used for billing:

```bash
curl "https://prices.azure.com/api/retail/prices?api-version=2023-01-01-preview&meterRegion='primary'"
```

### 5. Savings Plan Pricing

Savings plan pricing is only available in preview version:

```bash
curl "https://prices.azure.com/api/retail/prices?api-version=2023-01-01-preview&\$filter=serviceName eq 'Virtual Machines'"
```

### 6. Pagination Handling

API returns max 1,000 items per request. Use `NextPageLink` for subsequent pages:

```python
import requests
import json

url = "https://prices.azure.com/api/retail/prices"
params = {'$filter': "serviceName eq 'Virtual Machines'"}

all_items = []
response = requests.get(url, params=params)
data = response.json()
all_items.extend(data['Items'])

while 'NextPageLink' in data and data['NextPageLink']:
    response = requests.get(data['NextPageLink'])
    data = response.json()
    all_items.extend(data['Items'])

print(f"Total items: {len(all_items)}")
```

## Supported Filters

The following fields support filtering:

- `armRegionName` - Azure region name (e.g., 'westeurope', 'eastus')
- `location` - Data center location (e.g., 'EU West', 'US East')
- `meterId` - Unique meter identifier
- `meterName` - Meter name (e.g., 'F16s Spot')
- `productId` - Product ID
- `skuId` - SKU ID
- `productName` - Product name (e.g., 'Virtual Machines FS Series Windows')
- `skuName` - SKU name (e.g., 'F16s Spot')
- `serviceName` - Service name (e.g., 'Virtual Machines')
- `serviceId` - Service ID
- `serviceFamily` - Service family (e.g., 'Compute', 'Storage', 'Networking')
- `priceType` - Price type ('Consumption', 'Reservation', 'DevTestConsumption')
- `armSkuName` - Azure Resource Manager SKU name (e.g., 'Standard_F16s')

## Service Families

Supported `serviceFamily` values:

- Analytics
- Azure Arc
- Azure Communication Services
- Azure Security
- Azure Stack
- Compute
- Containers
- Data
- Databases
- Developer Tools
- Gaming
- Integration
- Internet of Things
- Management and Governance
- Mixed Reality
- Networking
- Security
- Storage
- Telecommunications
- Web
- Windows Virtual Desktop
- Power Platform
- Quantum Computing

## API Response Structure

```json
{
  "currencyCode": "USD",
  "tierMinimumUnits": 0.0,
  "retailPrice": 0.176346,
  "unitPrice": 0.176346,
  "armRegionName": "westeurope",
  "location": "EU West",
  "effectiveStartDate": "2020-08-01T00:00:00Z",
  "meterId": "000a794b-bdb0-58be-a0cd-0c3a0f222923",
  "meterName": "F16s Spot",
  "productId": "DZH318Z0BQPS",
  "skuId": "DZH318Z0BQPS/00TG",
  "productName": "Virtual Machines FS Series Windows",
  "skuName": "F16s Spot",
  "serviceName": "Virtual Machines",
  "serviceId": "DZH313Z7MMC8",
  "serviceFamily": "Compute",
  "unitOfMeasure": "1 Hour",
  "type": "Consumption",
  "isPrimaryMeterRegion": true,
  "armSkuName": "Standard_F16s"
}
```

### Savings Plan Response Example

```json
{
  "currencyCode": "USD",
  "retailPrice": 2.305,
  "unitPrice": 2.305,
  "serviceName": "Virtual Machines",
  "armSkuName": "Standard_M8ms",
  "savingsPlan": [
    {
      "unitPrice": 0.8065195,
      "retailPrice": 0.8065195,
      "term": "3 Years"
    },
    {
      "unitPrice": 1.5902195,
      "retailPrice": 1.5902195,
      "term": "1 Year"
    }
  ]
}
```

## Output Format

### Basic JSON Output

API responses are in JSON format with the following structure:

```json
{
  "BillingCurrency": "USD",
  "CustomerEntityId": "Default",
  "CustomerEntityType": "Retail",
  "Items": [
    /* Array of pricing items */
  ],
  "NextPageLink": "https://prices.azure.com:443/api/retail/prices?$skip=1000",
  "Count": 1000
}
```

## Important Notes

- **No Authentication Required**: No API keys or tokens needed
- **Rate Limiting**: No explicit limits documented, but use reasonable intervals
- **Case Sensitivity**: API version 2023-01-01-preview onwards, filter values are case-sensitive
  - Correct: `serviceName eq 'Virtual Machines'`
  - Incorrect: `serviceName eq 'virtual machines'`
- **Currency**: Base currency is USD. Other currency prices are for reference
- **Savings Plans**: Only supported in preview version
- **Pagination**: Use `NextPageLink` for large datasets

## Error Handling

- Invalid filter values: HTTP status code 400
- Misspelled service names: Empty result set
- Non-existent regions: Empty result set

## References

- [Official API Documentation](https://learn.microsoft.com/en-us/rest/api/cost-management/retail-prices/azure-retail-prices)
- [Supported Currencies](https://learn.microsoft.com/en-us/azure/cost-management-billing/microsoft-customer-agreement/microsoft-customer-agreement-faq#how-is-azure-priced-under-the-microsoft-customer-agreement)
- [Azure Savings Plans Documentation](https://learn.microsoft.com/en-us/azure/cost-management-billing/savings-plan/)
