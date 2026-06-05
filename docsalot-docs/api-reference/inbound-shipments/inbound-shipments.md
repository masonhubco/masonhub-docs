# Inbound Shipments / ASN

The Inbound Shipments API, also known as ASN (Advance Ship Notice), manages incoming inventory shipments to MasonHub distribution centers. This system allows you to notify MasonHub about expected deliveries and track their receiving status.

## Endpoints

### Get Inbound Shipments (ASN)

**GET** `/inbound_shipments`

Retrieve inbound shipment information with comprehensive filtering options.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `id` | array of UUIDs | MasonHub UUIDs [1..30] | - |
| `cid` | array of strings | Customer identifiers [1..30] | - |
| `cpid` | array of strings | Customer purchase order IDs | - |
| `status` | string | Filter by shipment status | - |
| `sdt` | string | Start date/time (RFC3339) | - |
| `edt` | string | End date/time (RFC3339) | - |
| `offset` | integer | Pagination offset | 0 |
| `limit` | integer | Number of results [1..100] | 30 |
| `list_type` | string | `detail` or `summary` | `detail` |

#### Example Request

```http
GET /inbound_shipments?status=open&sdt=2018-12-01T00:00:00Z&limit=25
```

### Create Inbound Shipments (ASN)

**POST** `/inbound_shipments`

Create one or more inbound shipments. Maximum 10 shipments per request.

#### Request Body Schema

Array of inbound shipment objects:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `customer_identifier` | string | Yes | Unique shipment identifier |
| `customer_purchase_order_id` | string | Yes | Purchase order reference |
| `inventory_location_id` | string | Yes | MasonHub location UUID |
| `expected_arrival_date` | string | Yes | Expected delivery date (RFC3339) |
| `shipper_name` | string | Yes | Name of shipping company |
| `carrier_information` | object | No | Detailed carrier info |
| `tracking_number` | string | No | Carrier tracking number |
| `shipper_city` | string | No | Shipper city |
| `shipper_locale` | string | No | Shipper state/locale |
| `shipper_country` | string | No | Shipper country |
| `special_instructions` | string | No | Handling instructions |
| `comments` | string | No | Additional comments |
| `line_items` | array | Yes | Items expected in shipment |

#### Carrier Information Object

```json
{
  "name": "Frankie's Red Hot Carrier",
  "type": "TL",
  "contact_name": "John Jacob", 
  "driver_name": "Donny James Jr",
  "driver_phone": "555-343-4213"
}
```

#### Line Items Array

```json
[
  {
    "customer_sku_id": "10000",
    "quantity": 80
  },
  {
    "customer_sku_id": "10001", 
    "quantity": 20
  }
]
```

#### Example Request

```json
[
  {
    "customer_identifier": "shipment32432",
    "customer_purchase_order_id": "po234532", 
    "inventory_location_id": "bf7bb516-ce52-8950-2a8b-9008f0091d93",
    "carrier_information": {
      "name": "Frankie's Red Hot Carrier",
      "type": "TL",
      "contact_name": "John Jacob",
      "driver_name": "Donny James Jr", 
      "driver_phone": "555-343-4213"
    },
    "tracking_number": "356732154",
    "shipper_name": "Freddie's Pirate Army",
    "shipper_city": "Kalamazoo",
    "shipper_locale": "MI",
    "shipper_country": "US",
    "expected_arrival_date": "2018-12-11T10:00:00Z",
    "special_instructions": "Fold using provided template during put away",
    "comments": "Shipments often not labeled but carrier is reliable",
    "line_items": [
      {
        "customer_sku_id": "10000",
        "quantity": 80
      },
      {
        "customer_sku_id": "10001",
        "quantity": 20
      }
    ]
  }
]
```

### Update Inbound Shipments (ASN)

**PUT** `/inbound_shipments`

Update one or more inbound shipments. Only shipments in 'open' status can be updated.

#### Request Body Schema

Same schema as Create Inbound Shipments. Full replacement of shipment data is required.

### Delete Inbound Shipments (ASN)

**DELETE** `/inbound_shipments`

Soft delete inbound shipments by customer identifier. Only open shipments can be deleted.

#### Request Body Schema

```json
[
  {
    "customer_identifier": "shipment234523"
  }
]
```

## Shipment Statuses

### Status Lifecycle

1. **open**: Shipment created, awaiting arrival
2. **onDock**: Shipment has arrived at facility
3. **receivingStarted**: Receiving process has begun
4. **receivingComplete**: All items have been received

### Status Transitions

- Creating a shipment → **open**
- Arrival at facility → **onDock** 
- Start processing → **receivingStarted**
- Finish processing → **receivingComplete**

## Receiving Process

### Expected Inventory

When inbound shipments are created, the system automatically:
- Updates expected inventory totals
- Triggers **skuInventoryChange** callbacks
- Maintains accurate inventory projections

### Receiving Events

The system generates **inboundShipmentEvent** callbacks for:

#### OnDock Event
```json
{
  "customer_identifier": "shipment32432",
  "status": "onDock", 
  "as_of": "2018-12-11T09:15:00Z"
}
```

#### Receiving Started Event  
```json
{
  "customer_identifier": "shipment32432",
  "status": "receivingStarted",
  "as_of": "2018-12-11T11:01:24Z"
}
```

#### Receiving Completed Event
```json
{
  "customer_identifier": "shipment32432", 
  "status": "receivingComplete",
  "as_of": "2018-12-11T15:53:23Z",
  "details": {
    "line_items": [
      {
        "customer_sku_id": "shirt-534212",
        "quantity": 100,
        "total_quantity_received": 90,
        "not_on_original_shipment": false
      }
    ]
  }
}
```

## Key Features

### Inventory Integration
- Automatic expected inventory updates
- Real-time inventory change notifications
- Handles partial and over receipts

### Exception Handling
- Tracks items not on original shipment
- Records receiving discrepancies
- Maintains audit trail

### Validation
- Ensures all line item SKUs exist in catalog
- Validates shipment status for updates/deletes
- Enforces business rules

## Best Practices

1. **Location IDs**: Always verify inventory location UUIDs before creating shipments
2. **SKU Validation**: Ensure all line item SKUs exist in your catalog before creating shipments
3. **Status Monitoring**: Use callbacks to monitor receiving progress rather than polling
4. **Batch Updates**: Update multiple shipments in single requests when possible
5. **Error Handling**: Handle partial success scenarios in bulk operations