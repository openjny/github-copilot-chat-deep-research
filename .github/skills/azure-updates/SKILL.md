---
name: azure-updates
description: Retrieve Azure Updates information using the Microsoft Release Communications API. Get the latest announcements about new features, previews, retirements, and general availability for Azure services without authentication. Use when tracking Azure product updates, monitoring service changes, checking retirement announcements, or following new feature releases.
metadata:
  author: openjny
  version: "1.0"
---

# Azure Updates Skill

Retrieve Azure Updates information from the official Microsoft Release Communications API without authentication.

This is the official source for Azure service updates, including new features, previews, retirements, and general availability announcements. Prioritize this source when looking up Azure service updates.

## API Endpoint

```
https://www.microsoft.com/releasecommunications/api/v2/azure
```

## Response Structure

### Top-level Response Fields

| Field            | Type    | Description                                          |
| ---------------- | ------- | ---------------------------------------------------- |
| `@odata.context` | string  | OData metadata URL                                   |
| `@odata.count`   | integer | Total number of items (when `$count=true`)           |
| `value`          | array   | List of update items                                 |
| `facets`         | array   | Available filter options (when `includeFacets=true`) |

### Update Item Fields

| Field                            | Type   | Description                                                                   |
| -------------------------------- | ------ | ----------------------------------------------------------------------------- |
| `id`                             | string | Unique identifier for the update                                              |
| `title`                          | string | Update title                                                                  |
| `description`                    | string | HTML-formatted description                                                    |
| `status`                         | string | Status: "Launched", "In preview", "In development", or null (for retirements) |
| `products`                       | array  | Related Azure products                                                        |
| `productCategories`              | array  | Product categories (e.g., "Compute", "AI + machine learning")                 |
| `tags`                           | array  | Tags (e.g., "Features", "Retirements", "Compliance")                          |
| `generalAvailabilityDate`        | string | GA date in "YYYY-MM" format                                                   |
| `previewAvailabilityDate`        | string | Preview date in "YYYY-MM" format                                              |
| `privatePreviewAvailabilityDate` | string | Private preview date                                                          |
| `availabilities`                 | array  | Availability details (see Availability Ring Values below)                     |
| `created`                        | string | Created timestamp (ISO 8601)                                                  |
| `modified`                       | string | Last modified timestamp (ISO 8601)                                            |
| `locale`                         | string | Locale (typically null)                                                       |

### Available Status Values

- `Launched` - Generally Available
- `In preview` - Public Preview
- `In development` - In Development
- `null` - Typically for retirements

### Availability Ring Values

The `availabilities` array contains objects with `ring`, `year`, and `month` fields:

| Ring Value             | Description        |
| ---------------------- | ------------------ |
| `General Availability` | GA release         |
| `Preview`              | Public Preview     |
| `Private Preview`      | Private Preview    |
| `Retirement`           | Service retirement |

Example:

```json
{
  "availabilities": [
    { "ring": "Preview", "year": 2024, "month": "October" },
    { "ring": "General Availability", "year": 2025, "month": "March" }
  ]
}
```

## Steps

### 1. Basic API Call

Retrieve recent Azure updates:

```bash
# Basic call with count
curl "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=10"

# With jq for better readability
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5" \
  | jq '.value[] | {id, title, status}'
```

### 2. Pagination

Use `top` and `skip` parameters for pagination:

```bash
# Get first 20 items
curl "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=20&skip=0"

# Get next 20 items
curl "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=20&skip=20"

# Get items 41-60
curl "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=20&skip=40"
```

### 3. Sorting

Use `$orderby` to sort results:

```bash
# Sort by modified date (newest first)
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$orderby=modified%20desc" \
  | jq '.value[] | {title, modified}'

# Sort by created date (oldest first)
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$orderby=created%20asc" \
  | jq '.value[] | {title, created}'

# Sort by title alphabetically
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$orderby=title" \
  | jq '.value[] | {title}'
```

### 4. Filtering by Status

```bash
# Get updates in preview
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=status%20eq%20%27In%20preview%27" \
  | jq '.value[] | {title, status}'

# Get launched (GA) updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=status%20eq%20%27Launched%27" \
  | jq '.value[] | {title, status}'

# Get updates in development
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=status%20eq%20%27In%20development%27" \
  | jq '.value[] | {title, status}'
```

### 5. Filtering by Tags

Use `tags/any()` for filtering by tags:

```bash
# Get retirement announcements
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=tags/any(t:t%20eq%20%27Retirements%27)" \
  | jq '.value[] | {title, tags}'

# Get feature announcements
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=tags/any(t:t%20eq%20%27Features%27)" \
  | jq '.value[] | {title, tags}'

# Get security-related updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=tags/any(t:t%20eq%20%27Security%27)" \
  | jq '.value[] | {title, tags}'

# Get Microsoft Build announcements
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=tags/any(t:t%20eq%20%27Microsoft%20Build%27)" \
  | jq '.value[] | {title, tags}'
```

### 6. Filtering by Product

Use `products/any()` for filtering by Azure products:

```bash
# Get AKS updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=products/any(p:p%20eq%20%27Azure%20Kubernetes%20Service%20(AKS)%27)" \
  | jq '.value[] | {title, products}'

# Get Azure Functions updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=products/any(p:p%20eq%20%27Azure%20Functions%27)" \
  | jq '.value[] | {title, products}'

# Get Azure Cosmos DB updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=products/any(p:p%20eq%20%27Azure%20Cosmos%20DB%27)" \
  | jq '.value[] | {title, products}'

# Get Azure OpenAI Service updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=products/any(p:p%20eq%20%27Azure%20OpenAI%20Service%27)" \
  | jq '.value[] | {title, products}'
```

### 7. Filtering by Product Category

Use `productCategories/any()` for filtering by category:

```bash
# Get AI + machine learning updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=productCategories/any(c:c%20eq%20%27AI%20%2B%20machine%20learning%27)" \
  | jq '.value[] | {title, productCategories}'

# Get Compute updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=productCategories/any(c:c%20eq%20%27Compute%27)" \
  | jq '.value[] | {title, productCategories}'

# Get Containers updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=productCategories/any(c:c%20eq%20%27Containers%27)" \
  | jq '.value[] | {title, productCategories}'

# Get Security updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=productCategories/any(c:c%20eq%20%27Security%27)" \
  | jq '.value[] | {title, productCategories}'
```

### 8. Filtering by Availability Ring

Use `availabilities/any()` for filtering by availability ring (recommended for retirements):

```bash
# Get retirement announcements (recommended method)
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=availabilities/any(a:a/ring%20eq%20%27Retirement%27)" \
  | jq '.value[] | {title, availabilities}'

# Get GA updates by ring
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=availabilities/any(a:a/ring%20eq%20%27General%20Availability%27)" \
  | jq '.value[] | {title, availabilities}'

# Get Preview updates by ring
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=availabilities/any(a:a/ring%20eq%20%27Preview%27)" \
  | jq '.value[] | {title, availabilities}'
```

### 9. Filtering by Date

```bash
# Get updates GA'd in January 2025
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=generalAvailabilityDate%20eq%20%272025-01%27" \
  | jq '.value[] | {title, generalAvailabilityDate}'

# Get updates preview'd in December 2024
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=previewAvailabilityDate%20eq%20%272024-12%27" \
  | jq '.value[] | {title, previewAvailabilityDate}'
```

### 10. Search

Use `$search` for text search:

```bash
# Search for "kubernetes" in updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$search=kubernetes" \
  | jq '.value[] | {title}'

# Search for "AI" in updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$search=AI" \
  | jq '.value[] | {title}'

# Search for "retirement" in updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$search=retirement" \
  | jq '.value[] | {title}'
```

### 11. Getting Facets (Filter Options)

Use `includeFacets=true` to get available filter values with counts:

```bash
# Get all facets
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&includeFacets=true&top=1" \
  | jq '.facets'

# Get product categories with counts
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&includeFacets=true&top=1" \
  | jq '.facets[] | select(.name == "ProductCategory") | .values[] | {value, count}'

# Get status options with counts
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&includeFacets=true&top=1" \
  | jq '.facets[] | select(.name == "Status") | .values'

# Get tags with counts
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&includeFacets=true&top=1" \
  | jq '.facets[] | select(.name == "Tags") | .values[] | {value, count}'
```

### 12. Combining Filters

Combine multiple filters with `and`:

```bash
# Get AKS updates that are in preview
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=products/any(p:p%20eq%20%27Azure%20Kubernetes%20Service%20(AKS)%27)%20and%20status%20eq%20%27In%20preview%27" \
  | jq '.value[] | {title, status}'

# Get security updates from Containers category
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=productCategories/any(c:c%20eq%20%27Containers%27)%20and%20tags/any(t:t%20eq%20%27Security%27)" \
  | jq '.value[] | {title, tags, productCategories}'

# Get launched features in AI category
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=5&\$filter=productCategories/any(c:c%20eq%20%27AI%20%2B%20machine%20learning%27)%20and%20status%20eq%20%27Launched%27%20and%20tags/any(t:t%20eq%20%27Features%27)" \
  | jq '.value[] | {title, status}'
```

### 13. Getting Latest Updates (Common Use Case)

```bash
# Get 10 most recently modified updates
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=10&\$orderby=modified%20desc" \
  | jq '.value[] | {title, status, modified}'

# Get latest GA announcements
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=10&\$filter=status%20eq%20%27Launched%27&\$orderby=modified%20desc" \
  | jq '.value[] | {title, generalAvailabilityDate, modified}'

# Get latest preview announcements
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=10&\$filter=status%20eq%20%27In%20preview%27&\$orderby=modified%20desc" \
  | jq '.value[] | {title, previewAvailabilityDate, modified}'

# Get upcoming retirements (using availability ring)
curl -s "https://www.microsoft.com/releasecommunications/api/v2/azure?\$count=true&top=10&\$filter=availabilities/any(a:a/ring%20eq%20%27Retirement%27)&\$orderby=modified%20desc" \
  | jq '.value[] | {title, availabilities, modified}'
```

## Output Format

### Simplified Update Item

```json
{
  "id": "466954",
  "title": "Public Preview: Collect Azure Container Storage metrics...",
  "status": "In preview",
  "products": ["Azure Container Storage", "Azure Kubernetes Service (AKS)"],
  "productCategories": ["Containers", "Compute"],
  "tags": ["Features"],
  "generalAvailabilityDate": null,
  "previewAvailabilityDate": "2024-12",
  "modified": "2025-01-09T16:00:28.6638383Z"
}
```

### Facet Response

```json
{
  "name": "Status",
  "values": [
    { "value": "In development", "count": 137 },
    { "value": "In preview", "count": 2929 },
    { "value": "Launched", "count": 4686 }
  ]
}
```

## Query Parameters Summary

| Parameter       | Type    | Description               | Example                        |
| --------------- | ------- | ------------------------- | ------------------------------ |
| `$count`        | boolean | Include total count       | `$count=true`                  |
| `includeFacets` | boolean | Include filter facets     | `includeFacets=true`           |
| `top`           | integer | Number of items to return | `top=20`                       |
| `skip`          | integer | Number of items to skip   | `skip=20`                      |
| `$orderby`      | string  | Sort field and direction  | `$orderby=modified desc`       |
| `$filter`       | string  | OData filter expression   | `$filter=status eq 'Launched'` |
| `$search`       | string  | Text search query         | `$search=kubernetes`           |

## Filter Expression Examples

| Filter Type          | Expression                                             |
| -------------------- | ------------------------------------------------------ |
| By status            | `status eq 'Launched'`                                 |
| By tag               | `tags/any(t:t eq 'Features')`                          |
| By availability ring | `availabilities/any(a:a/ring eq 'Retirement')`         |
| By product           | `products/any(p:p eq 'Azure Functions')`               |
| By category          | `productCategories/any(c:c eq 'Compute')`              |
| By date              | `generalAvailabilityDate eq '2025-01'`                 |
| Combined             | `status eq 'Launched' and tags/any(t:t eq 'Features')` |

## Notes

- The API is publicly accessible without authentication
- All dates are in UTC and ISO 8601 format
- HTML content in `description` field needs to be parsed/stripped for plain text
- Special characters in filter values must be URL-encoded
- Maximum `top` value may be limited by the API
