# Inventory Management

The Inventory API provides real-time inventory tracking and snapshot capabilities across all distribution centers. It supports various inventory statuses and provides detailed breakdowns by location and status.

## Endpoints

### Get SKU Inventory Snapshot

**GET** `/inventory`

Retrieve current inventory counts for specified SKUs across all distribution centers.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `cid` | array of strings | Customer identifiers [1..30] | - |
| `limit` | integer | Number of results [1..100] | 30 |
| `offset` | integer | Pagination offset | 0 |

#### Example Request

```http
GET /inventory?cid=pants4523,shirts872340&limit=50
```

#### Example Response

```json
{
  "include_ledger": true,
  "limit": 30,
  "offset": 0,
  "data": [
    {
      "sku_id": "e5d00ff6-b34a-496e-9559-eb90e99463af",
      "customer_identifier": "10000",
      "total_units": 200,
      "total_available_to_sell": 100,
      "as_of": "2018-11-18T00:43:23.095637Z",
      "status_counts": [
        {
          "inventory_location_id": "bf7bb516-ce52-8950-2a8b-9008f0091d93",
          "inventory_location_address": "MasonHub - Rancho Cucamonga | 8595 Milliken Avenue, B102 Rancho Cucamonga, CA 91730",
          "inventory_status": "available",
          "quantity": 100,
          "as_of": "2018-11-18T00:43:22.664965Z"
        }
      ]
    }
  ]
}
```

### Request a Full Snapshot

**POST** `/snapshot_request`

Initiate an asynchronous full inventory snapshot job for large datasets.

#### Request Body Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `snapshot_type` | string | Yes | Currently only `"full"` is supported |
| `snapshot_as_of_date` | string | No | Point-in-time snapshot date (RFC3339) |

#### Example Request

```json
{
  "snapshot_type": "full",
  "snapshot_as_of_date": "2019-08-05T08:15:30-07:00"
}
```

#### Response

The system will process the snapshot asynchronously and send a **SnapshotReady** callback when complete.

### Get Inventory Locations

**GET** `/{accountSlug}/inventory_locations`

Retrieve information about warehouse locations and their UUIDs.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `accountSlug` | string | Yes | Account slug identifying the account |

#### Example Response

```json
{
  "id": "bf7bb516-ce52-8950-2a8b-9008f0091d93",
  "name": "Rancho Cucamonga",
  "city": "Rancho Cucamonga", 
  "locale_code": "CA",
  "address": "8595 Milliken Avenue, B102 Rancho Cucamonga, CA 91730"
}
```

## Inventory Statuses

The system tracks inventory across multiple statuses:

### Primary Statuses
- **available**: Ready for sale and allocation
- **expected**: Incoming inventory from inbound shipments
- **received**: Recently received but not yet available
- **in-receiving**: Currently being processed

### Quality Control Statuses  
- **quality-control**: Under quality inspection
- **damaged**: Damaged inventory requiring disposition
- **refurbishing**: Being refurbished or repaired
- **under-investigation**: Status under investigation

### Lost/Unavailable
- **lost**: Inventory that cannot be located

## Inventory Calculations

### Total Units
Sum of all inventory across all statuses and locations for a SKU.

### Total Available to Sell
Inventory that can be allocated to new orders, typically includes:
- Available inventory
- Expected inventory (for some business rules)

### Kit Inventory
For kit SKUs, inventory calculations automatically consider:
- Pre-built kit quantities 
- Component availability for kit-to-ship
- Mixed inventory scenarios

## Real-time Updates

Inventory changes trigger **skuInventoryChange** callbacks providing:
- Updated totals across all locations
- Status-specific breakdowns  
- Timestamp information
- Location details

### Example Inventory Change Callback

```json
{
  "callback_url": "https://client.com/api/inventory_event",
  "message_type": "skuInventoryChange",
  "message_id": "e58769ec-c4cf-47f9-b80a-378a9b85aeae",
  "data": [
    {
      "sku_id": "e5d00ff6-b34a-496e-9559-eb90e99463af",
      "customer_identifier": "10000", 
      "total_units": 200,
      "total_available_to_sell": 100,
      "as_of": "2018-11-18T00:43:23.095637Z",
      "status_counts": [...]
    }
  ]
}
```

## Best Practices

1. **Polling vs Callbacks**: Use callbacks for real-time updates rather than frequent polling
2. **Batch Queries**: Query multiple SKUs in single requests when possible
3. **Snapshot Usage**: Use full snapshots for initial sync or bulk operations
4. **Status Understanding**: Understand which statuses contribute to available-to-sell quantities