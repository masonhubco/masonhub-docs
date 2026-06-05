# Returns (RMA)

The Returns API manages RMA (Return Merchandise Authorization) processes, allowing customers to return items for processing, refund, or replacement. The system supports various return types and generates return labels automatically.

## Endpoints

### Get Returns

**GET** `/rmas`

Retrieve return information with filtering and pagination.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `id` | array of UUIDs | MasonHub RMA UUIDs [1..30] | - |
| `status` | string | Filter by RMA status | - |
| `sdt` | string | Start date/time (RFC3339) | - |
| `edt` | string | End date/time (RFC3339) | - |
| `offset` | integer | Pagination offset | 0 |
| `limit` | integer | Number of results [1..100] | 30 |
| `list_type` | string | `detail` or `summary` | `detail` |

#### Example Request

```http
GET /rmas?status=open&sdt=2018-12-01T00:00:00Z&limit=25
```

### Create Returns

**POST** `/rmas`

Create one or more return authorizations.

#### Request Body Schema

Array of RMA objects:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `customer_identifier` | string | Yes | Unique RMA identifier |
| `manifest_id` | string | No | Manifest identifier |
| `return_type` | string | Yes | Type of return |
| `generate_return_label` | boolean | No | Generate DHL return label (default: true) |
| `customer_address_*` | string | Conditional | Required if generating return label |
| `customer_instructions` | string | No | Instructions from customer |
| `line_items` | array | Yes | Items being returned |

#### Return Types

- **rma_submitted_by_customer**: Customer-initiated return
- **quality_control**: Quality control return
- **damaged_in_transit**: Damaged during shipping
- **wrong_item_shipped**: Incorrect item shipped

#### Customer Address Fields (Required for Return Labels)

```json
{
  "customer_address_name": "John Smith",
  "customer_address_street_line_one": "100 First Ave",
  "customer_address_street_line_two": "Apt 16",
  "customer_address_postal_code": "10016",
  "customer_address_city": "New York",
  "customer_address_state": "NY",
  "customer_address_country_code": "US"
}
```

#### Line Items Schema

```json
[
  {
    "customer_order_id": "54321",
    "customer_sku_id": "shirts872340",
    "quantity": 2,
    "return_reason_code": "tooBig",
    "return_notes": "Item was too large",
    "return_inventory_status": "quality-control"
  }
]
```

#### Return Reason Codes

- **tooBig**: Item was too large
- **tooSmall**: Item was too small  
- **damaged**: Customer received damaged item
- **wrongItem**: Customer received wrong item
- **other**: Reason not captured in standard codes

#### Return Inventory Statuses

Items can be returned into various statuses:
- **available**: Ready for resale
- **quality-control**: Requires inspection
- **damaged**: Damaged items
- **refurbishing**: Needs refurbishment
- **under-investigation**: Under review

#### Example Request

```json
[
  {
    "customer_identifier": "rma123",
    "return_type": "rma_submitted_by_customer",
    "generate_return_label": true,
    "customer_address_name": "John Smith",
    "customer_address_street_line_one": "100 First Ave",
    "customer_address_city": "New York",
    "customer_address_state": "NY", 
    "customer_address_postal_code": "10016",
    "customer_address_country_code": "US",
    "customer_instructions": "Dry clean before restocking",
    "line_items": [
      {
        "customer_order_id": "54321",
        "customer_sku_id": "shirts872340",
        "quantity": 1,
        "return_reason_code": "tooBig",
        "return_notes": "Didn't fit properly",
        "return_inventory_status": "quality-control"
      }
    ]
  }
]
```

#### Response with Return Label

```json
{
  "records_submitted": 1,
  "records_processed": 1,
  "records_failed": 0,
  "records_succeeded": 1,
  "results": [
    {
      "customer_identifier": "rma123",
      "status": "success",
      "uri": "https://app.masonhub.co/demo_account/api/v1/rmas?cid=rma123",
      "additional_data": {
        "return_label_url": "https://app.masonhub.co/demo_account/api/v1/return-labels/demo1234.pdf",
        "return_tracking_number": "9302020514103629166400"
      }
    }
  ]
}
```

### Update Returns

**PUT** `/rmas`

Update one or more returns with full replacement of data.

#### Request Body Schema

Same schema as Create Returns.

### Delete Returns

**DELETE** `/rmas`

Delete returns by customer identifier. Only returns in 'open' status can be deleted.

#### Request Body Schema

```json
[
  {
    "customer_identifier": "rma123"
  },
  {
    "customer_identifier": "rma456"
  }
]
```

## RMA Statuses

### Status Lifecycle

1. **open**: RMA created, awaiting return shipment
2. **tendered**: Return package tendered to carrier
3. **inTransit**: Return package in transit
4. **delivered**: Return package delivered to facility
5. **received**: Return items received and processed
6. **undeliverable**: Return package undeliverable

## Return Label Generation

### DHL Return Labels

MasonHub automatically generates DHL return labels when:
- `generate_return_label` is true (default)
- Complete customer address is provided
- Return is successfully created

### Label Security

- Return label URLs require API authentication
- Labels contain customer personal information
- URLs are time-limited for security

### Accessing Return Labels

```http
GET /return-labels/demo1234.pdf
Authorization: Bearer your_jwt_token
```

## RMA Events and Callbacks

### RMA Event Callbacks

Subscribe to **rmaEvent** callbacks for return status updates:

```json
{
  "callback_url": "https://client.com/api/rmaEvent",
  "message_type": "rmaEvent",
  "message_id": "0896116f-e54b-4756-9d3e-1b0c4a25d821",
  "data": [
    {
      "id": "vf79fyn5-ec61-7892-ae7b-57a173b68133",
      "customer_identifier": "rma123",
      "package_id": "1z324897234nferg45",
      "return_type": "rma_submitted_by_customer",
      "status": "received",
      "received_at": "2018-08-06T11:18:46Z",
      "notes": "All items received in good condition",
      "customer_instructions": "Dry Clean",
      "line_items": [
        {
          "customer_order_id": "54321",
          "customer_sku_id": "shirts872340",
          "quantity": 2,
          "quantity_received": 2,
          "return_reason_code": "tooBig",
          "return_notes": "Didn't fit",
          "return_inventory_status": "quality-control",
          "not_on_original_rma": false
        }
      ]
    }
  ]
}
```

### Unexpected Items

When items not on the original RMA are received:
- `not_on_original_rma` is set to `true`
- Items are added to the RMA if they exist in the catalog
- Appropriate inventory adjustments are made

## Business Rules and Validation

### Order Dependencies (Optional)

- RMAs can be created without existing orders
- Facilitates go-live scenarios with historical returns
- When order references are used, validation ensures:
  - Order exists in the system
  - SKU was on the referenced order
  - Quantity doesn't exceed original order quantity

### Cross-Order Returns

- Single RMA can reference multiple orders
- Order references exist at line item level
- Flexible return grouping for customer convenience

## Inventory Impact

### Return Processing

When returns are received:
- Inventory is added to specified status
- **skuInventoryChange** callbacks are triggered
- Total inventory counts are updated
- Status-specific breakdowns are maintained

### Partial Receipts

The system handles:
- Full receipts (all items returned)
- Partial receipts (some items returned)
- Over receipts (more items than expected)
- Quality variances (different conditions than expected)

## Best Practices

1. **Complete Addresses**: Provide complete customer addresses for return label generation
2. **Clear Instructions**: Include detailed customer instructions for handling
3. **Proper Reason Codes**: Use appropriate return reason codes for analytics
4. **Status Monitoring**: Use rmaEvent callbacks to track return progress
5. **Quality Control**: Route questionable returns through quality control status
6. **Documentation**: Maintain detailed notes for customer service and inventory teams