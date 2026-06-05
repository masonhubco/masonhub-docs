# The Stack Trace

MasonHub's Order Management System (OMS) logs and maintains version history of all operations it performs. Every request from clients, every queued item processed, and every action taken is recorded and connected back to the preceding action that either called it or caused it. This creates a comprehensive audit trail called the "stack trace."

## Overview

In programming, a stack trace typically shows the execution path of code to help engineers find errors or performance bottlenecks. In MasonHub's OMS, the stack trace serves a broader purpose: it connects every action in the system back to its origin, providing complete transparency and auditability for all operations.

As a 3PL (Third-Party Logistics provider), MasonHub needs to know what happened, when it happened, and how to troubleshoot if needed. The stack trace isn't just a feature of the product—it **is** the product. MasonHub's guarantee is that the system is accurate, reliable, and repeatable, and the stack trace is essential to maintaining and proving these qualities.

## The Stack

The "stack" is conceptually simple but comprehensive in implementation. Every API call and subsequent action is logged and stored with complete context and relationships.

Here are the key points where data is logged and stored:

## Inbound Requests

When a client makes an API request (for example, creating SKUs), MasonHub first records the request and response:

### Example: SKU Creation Request Log

```
id: 43f66b48-24bd-4c17-bc0f-16355861beb5
account_id: [account id for test account]
request_method: POST
request_url: /test/api/v1/skus
request_headers: user-agent: PostmanRuntime/7.4.0, cache-control: no-cache, [additional headers]
request_body: [
  {
    "customer_identifier": "kit00001",
    "brand_name": "peter piper",
    "customer_bar_code": "kit234523",
    "customer_brand_code": "pp",
    "customer_color": "w",
    "brand_code": "3",
    "dim_height": 12,
    "dim_weight": 10,
    "dim_length": 2,
    "dim_volume": 3,
    "image_url": "http://bohemianrhapsody.com/img432523.jpg",
    "price": 15.95,
    "unit_cost": 5.95,
    "product_category": "shirt",
    "product_sub_category": "man blouse",
    "size": "M",
    "vendor_bar_code": "kit675443",
    "vendor_brand_code": "pete's",
    "vendor_color": "WHITE",
    "product_name": "McShirt",
    "product_description": "This is my product description",
    "is_kit": true,
    "kit_skus": [
      {
        "customer_identifier": "10000",
        "quantity": 2
      },
      {
        "customer_identifier": "10001", 
        "quantity": 1
      }
    ]
  }
]
response_headers: content-type: application/json, [additional headers]
response_body: {
  "records_submitted": 1,
  "records_processed": 1,
  "records_failed": 0,
  "records_succeeded": 1,
  "results": [
    {
      "customer_identifier": "kit00001",
      "status": "success",
      "uri": "https://sandbox.masonhub.co/test/api/v1/skus?cid=kit00001"
    }
  ]
}
response_code: 200
created_at: 2018-11-30 01:35:43.3707+00
updated_at: 2018-11-30 01:35:43.370703+00
request_id: NRakdTYTIo-ZnozabOBxQ
```

## Queued Payload Processing

Because all MasonHub API calls are non-blocking, the system splits array requests into individual queued payloads for processing. Each payload is processed individually with timing and results recorded:

### Example: Individual SKU Processing Log

```
id: 798d67b0-ca41-4310-80c2-d760fa363ff8
account_id: [account id hidden to protect the innocent]
json: {
  "id": null,
  "customer_identifier": "3342336657679876986",
  "customer_bar_code": "324gjhfjkhgf2342",
  "vendor_bar_code": "2342345dfghzsdf342", 
  "pick_style": "FIFO",
  "brand_name": "peter piper",
  "product_name": "McShirt",
  "product_category": "shirt",
  "product_sub_category": "man blouse",
  "product_description": "This is my product description",
  "image_url": "http://bohemianrhapsody.com/img432523.jpg",
  "vendor_brand_code": "pete's",
  "customer_brand_code": "pp",
  "customer_color": "w",
  "vendor_color": "WHITE",
  "size": "M",
  "ormd": false,
  "lot_controlled": false,
  "expirable": false,
  "unit_cost": 5.95,
  "price": 15.95,
  "dim_weight": 10,
  "dim_length": 2,
  "dim_height": 12,
  "dim_volume": 3,
  "created_at": "0001-01-01T00:00:00Z",
  "updated_at": "0001-01-01T00:00:00Z",
  "deleted_at": null,
  "inventory_totals": {
    "sku_id": "00000000-0000-0000-0000-000000000000",
    "total_units": 0,
    "total_available_to_sell": 0,
    "as_of": "0001-01-01T00:00:00Z"
  }
}
created_at: 2018-11-21 21:29:56.922172
processed_at: 2018-11-21 21:29:57.841147
process_result: success
process_messages: [success messages]
```

## Inventory Ledger Entries

Every inventory change creates a ledger entry that references the action that triggered it:

### Example: Inventory Ledger Entry

```
id: ae973696-57f6-4dd7-b1c9-c9c0fd5c2dea
account_id: [account id hidden to protect the innocent]
inventory_location_id: bf7bb516-ce52-8950-2a8b-9008f0091d93
sku_id: e5d00ff6-b34a-496e-9559-eb90e99463af
reference_item_uri: app.masonhub.co/[account_id]/payload/676954fa-d6b2-47ff-b20e-cefac216ac65
actor_uri: data factory create available
entry_type: expected
quantity: 100
occured_at: 2018-11-21 16:13:30.612314+00
created_at: 2018-11-21 16:13:30.652487+00
```

The `reference_item_uri` and `actor_uri` fields create the linkage that allows tracing any inventory change back to its source.

## Outbound Message Tracking

MasonHub also logs all outbound messages (callbacks/webhooks) sent to clients, including the full request and response:

### Example: Callback Message Log

```
id: e58769ec-c4cf-47f9-b80a-378a9b85aeae
account_id: 4f107138-28fc-cafd-a529-97e546a33239
message_type: skuInventoryChange
outbound_url: https://mister-bucket.herokuapp.com/api/bucket/skuInventoryChange
response_headers: via: 1.1 vegur, server: Cowboy, connection: keep-alive, content-type: application/json, [additional headers]
response_body: {
  "id": "39777d0e-1b15-4121-b403-a0a902263c07",
  "created_at": "2018-11-18T00:43:32.340280977Z", 
  "updated_at": "2018-11-18T00:43:32.340284129Z",
  "request_action": "skuInventoryChange",
  "payload": "{\"callback_url\":\"https://mister-bucket.herokuapp.com/api/bucket/skuInventoryChange\",\"message_type\":\"skuInventoryChange\",\"message_id\":\"e58769ec-c4cf-47f9-b80a-378a9b85aeae\",\"data\":[{\"sku_id\":\"e5d00ff6-b34a-496e-9559-eb90e99463af\",\"customer_identifier\":\"10000\",\"total_units\":200,\"total_available_to_sell\":100,\"as_of\":\"2018-11-18T00:43:23.095637Z\",\"status_counts\":[{\"inventory_location_id\":\"bf7bb516-ce52-8950-2a8b-9008f0091d93\",\"inventory_location_address\":\"MasonHub - Rancho Cucamonga | 8595 Milliken Avenue, B102 Rancho Cucamonga, CA 91730\",\"inventory_status\":\"available\",\"quantity\":100,\"as_of\":\"2018-11-18T00:43:22.664965Z\"},{\"inventory_location_id\":\"bf7bb516-ce52-8950-2a8b-9008f0091d93\",\"inventory_location_address\":\"MasonHub - Rancho Cucamonga | 8595 Milliken Avenue, B102 Rancho Cucamonga, CA 91730\",\"inventory_status\":\"received\",\"quantity\":100,\"as_of\":\"2018-11-18T00:43:23.095637Z\"},{\"inventory_location_id\":\"bf7bb516-ce52-8950-2a8b-9008f0091d93\",\"inventory_location_address\":\"MasonHub - Rancho Cucamonga | 8595 Milliken Avenue, B102 Rancho Cucamonga, CA 91730\",\"inventory_status\":\"in-receiving\",\"quantity\":0,\"as_of\":\"2018-11-18T00:43:23.095637Z\"},{\"inventory_location_id\":\"bf7bb516-ce52-8950-2a8b-9008f0091d93\",\"inventory_location_address\":\"MasonHub - Rancho Cucamonga | 8595 Milliken Avenue, B102 Rancho Cucamonga, CA 91730\",\"inventory_status\":\"expected\",\"quantity\":0,\"as_of\":\"2018-11-18T00:43:22.345419Z\"}]}]}"
}
payload: [Full event payload data]
created_at: 2018-11-18 00:43:25.904553+00
message_key: bf7bb516-ce52-8950-2a8b-9008f0091d93_e5d00ff6-b34a-496e-9559-eb90e99463af
response_code: 200
processed_at: 2018-11-18 00:43:32.349294+00
process_result: sent
process_messages: [delivery status]
bearer_token: [redacted]
send_most_recent: t
occured_at: 2018-11-18 00:43:23.095637+00
```

## Universal Resource Identifiers (URI)

All logged items are stored with URIs (Universal Resource Identifiers) that allow systems to tie arbitrary items together and trace actions back to their source. This provides the foundation for:

- **Complete audit trails**: Every action can be traced to its origin
- **Debugging capabilities**: Issues can be tracked through the entire system
- **Transparency**: Full visibility into system operations
- **Recoverability**: Nothing gets lost in translation
- **Infinite replay**: Calculations and processes can be replayed from any point

## Benefits of the Stack Trace System

### For MasonHub Engineering

- **Debugging**: Quickly identify root causes of issues
- **Performance optimization**: Track bottlenecks through the system
- **Quality assurance**: Verify system behavior and data integrity
- **System monitoring**: Real-time visibility into operations

### For MasonHub Clients

- **Transparency**: Complete visibility into order and inventory processing
- **Audit capability**: Full records for compliance and reporting
- **Troubleshooting**: Self-service debugging capabilities (future feature)
- **Confidence**: Proof that the system is accurate and reliable

### For Operations Teams

- **Order tracking**: Complete visibility into order processing
- **Inventory accuracy**: Full audit trail for all inventory changes  
- **Issue resolution**: Fast problem identification and resolution
- **Process improvement**: Data-driven operational enhancements

## Testing Infrastructure

MasonHub maintains sophisticated testing infrastructure to validate the stack trace system:

### Mr. Bucket

A smoke test service that receives all callback requests for inspection. Named because it "receives all requests for later inspection," Mr. Bucket responds with the original payload to verify system functionality.

### Mrs. Shovel  

A load testing service that creates thousands of test records to validate system performance under stress and identify race conditions.

## Future Capabilities

The stack trace foundation enables future features including:

- **Client debugging tools**: Self-service troubleshooting capabilities
- **Advanced analytics**: Deep insights into operational patterns  
- **Predictive capabilities**: Forecasting based on historical patterns
- **Automated optimization**: System self-tuning based on performance data

## Technical Implementation

The stack trace system is built into every level of the MasonHub OMS:

- **API Gateway**: Logs all inbound requests and responses
- **Queue Processing**: Tracks payload processing and results
- **Business Logic**: Records all state changes and decisions
- **External Communications**: Logs all outbound messages and responses
- **Data Storage**: Maintains complete historical records

This comprehensive logging and tracing system ensures that MasonHub can deliver on its promise of accuracy, reliability, and transparency in supply chain management.

**For questions about the stack trace system, contact Chris or Andy at MasonHub.**