# API Design Philosophy

## Everything is Really Just Search

GET methods use search parameters instead of resource URLs for composability and flexibility. URLs like `...?id=123121` allow for more flexible querying patterns.

## Always an Array

All HTTP methods (GET, POST, PUT, DELETE) accept or return arrays to enable batch operations. This design allows for efficient bulk processing of resources.

## Summary vs Detail

GET endpoints support `list_type=detail` or `list_type=summary` to control the level of information returned:

- **Detail**: Full resource information including all nested objects and relationships
- **Summary**: Condensed resource information with key fields only

## GET, POST, PUT, DELETE, No Patch

The API follows a simplified REST approach:

- **GET**: Retrieve resources using search parameters
- **POST**: Create new resources  
- **PUT**: Update existing resources (full replacement of records)
- **DELETE**: Remove resources
- **No PATCH**: Full object replacement is required for updates

## Error Handling

Bulk operations return HTTP 200 with individual success/failure status for each object in the batch:

```json
{
  "records_submitted": 3,
  "records_processed": 3, 
  "records_failed": 1,
  "records_succeeded": 2,
  "results": [
    {
      "customer_identifier": "item1",
      "status": "success"
    },
    {
      "customer_identifier": "item2", 
      "status": "failure",
      "reason": "Validation error"
    },
    {
      "customer_identifier": "item3",
      "status": "success"
    }
  ]
}
```

## Date Time Format

All date/time values conform to the RFC3339 standard format:

```
2018-08-05T17:32:28Z
2018-12-11T15:53:23.095637Z
```

This ensures consistent date handling across all API endpoints and integrations.