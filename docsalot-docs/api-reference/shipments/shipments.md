# Shipments

The Shipments API provides access to outbound shipment information for orders that have been fulfilled and shipped from MasonHub distribution centers.

## Endpoints

### Get Order Shipments

**GET** `/shipments`

Retrieve outbound shipments with comprehensive filtering options.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `id` | array of UUIDs | MasonHub shipment UUIDs [1..30] | - |
| `coid` | array of strings | Customer order IDs | - |
| `sdt` | string | Start date/time (RFC3339) | - |
| `edt` | string | End date/time (RFC3339) | - |
| `offset` | integer | Pagination offset | 0 |
| `limit` | integer | Number of results [1..100] | 30 |

#### Example Request

```http
GET /shipments?coid=12345,67890&sdt=2018-08-01T00:00:00Z
```

#### Example Response

```json
{
  "search_criteria": {
    "sdt": "2018-08-01T00:00:00Z",
    "edt": null
  },
  "limit": 30,
  "offset": 0,
  "list_type": "detail",
  "include_counts": false,
  "data": [
    {
      "id": "db79huy5-ec61-7892-ae7b-57a563b68133",
      "shipment_id": "f0h7uuk",
      "customer_order_id": "12345",
      "shipping_provider": "UPS",
      "shipper_service_level": "ground",
      "tracking_number": "1z324897234nferg45",
      "tracking_url": "http://ups.com/tracking/1z324897234nferg45",
      "shipment_date_time": "2018-08-06T15:11:19Z",
      "shipment_line_items": [
        {
          "sku_customer_id": "shirts872340",
          "quantity": 2
        },
        {
          "sku_customer_id": "pants3422",
          "quantity": 1
        }
      ]
    }
  ]
}
```

## Shipment Data Structure

### Core Shipment Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | MasonHub internal UUID |
| `shipment_id` | string | Human-readable shipment identifier |
| `customer_order_id` | string | Customer's order identifier |
| `shipping_provider` | string | Carrier (UPS, USPS, FedEx, etc.) |
| `shipper_service_level` | string | Service level (ground, priority, express) |
| `tracking_number` | string | Carrier tracking number |
| `tracking_url` | string | Direct tracking URL |
| `shipment_date_time` | string | Ship date/time (RFC3339) |
| `shipment_line_items` | array | Items in this shipment |

### Shipment Line Items

Each shipment contains line items with:

```json
{
  "sku_customer_id": "shirts872340",
  "quantity": 2
}
```

## Multi-Shipment Orders

Orders can ship in multiple packages when:

- Items are too large for single package
- Items ship from different locations
- Inventory availability requires splits
- Business rules dictate splitting

### Example Multi-Shipment Response

```json
{
  "data": [
    {
      "shipment_id": "f0h7uuk",
      "customer_order_id": "12345",
      "shipping_provider": "UPS",
      "tracking_number": "1z324897234nferg45",
      "shipment_line_items": [
        {
          "sku_customer_id": "shirts872340",
          "quantity": 2
        }
      ]
    },
    {
      "shipment_id": "u3d88z4",
      "customer_order_id": "12345",
      "shipping_provider": "UPS", 
      "tracking_number": "1z987654321abcdef",
      "shipment_line_items": [
        {
          "sku_customer_id": "pants3422",
          "quantity": 1
        }
      ]
    }
  ]
}
```

## Integration with Orders API

Shipment information is also available through the Orders API:

### GET `/orders` includes shipments

When retrieving orders, fulfilled orders include shipment data:

```json
{
  "id": "cb65fyn5-534t-7892-hg67-57g173b54133",
  "customer_identifier": "129374",
  "status": "fulfilled",
  "shipments": [
    {
      "shipment_id": "db79huy5-ec61-7892-ae7b-57a563b68133",
      "customer_order_id": "129374",
      "shipping_provider": "UPS",
      "shipper_service_level": "ground",
      "tracking_number": "1z324897234nferg45",
      "tracking_url": "http://ups.com/tracking/1z324897234nferg45",
      "shipment_date_time": "2018-08-06T15:11:19Z",
      "shipment_line_items": [
        {
          "sku_customer_id": "shirts872340",
          "quantity": 2
        }
      ]
    }
  ]
}
```

## Shipment Events and Callbacks

### Order Event Callbacks

When orders ship, **orderEvent** callbacks include shipment details:

```json
{
  "callback_url": "https://client.com/api/orderEvent",
  "message_type": "orderEvent",
  "message_id": "0896116f-e54b-4756-9d3e-1b0c4a25d821",
  "data": [
    {
      "order_id": "8f1316d5-859f-4802-98b0-63d03a9517ac",
      "customer_identifier": "12345",
      "status": "fulfilled",
      "shipments": [
        {
          "shipment_id": "88888",
          "shipping_provider": "UPS",
          "shipper_service_level": "ground", 
          "tracking_number": "1z324897234nferg45",
          "tracking_url": "http://ups.com/tracking/1z324897234nferg45",
          "shipment_date_time": "2018-08-06T15:11:19Z",
          "shipment_line_items": [
            {
              "sku_customer_id": "shirts872340",
              "quantity": 2
            }
          ]
        }
      ]
    }
  ]
}
```

## Tracking Integration

### Carrier Tracking URLs

MasonHub provides direct tracking URLs for major carriers:

- **UPS**: `http://ups.com/tracking/{tracking_number}`
- **USPS**: `http://usps.gov/tracking/{tracking_number}`
- **FedEx**: `https://www.fedex.com/apps/fedextrack/index.html?action=track&tracknumbers={tracking_number}`

### Custom Tracking Implementation

You can construct your own tracking URLs or integrate directly with carrier APIs using the provided tracking numbers.

## Use Cases

### Customer Notifications

Use shipment data to:
- Send shipping confirmation emails
- Provide tracking information
- Update order status on customer portals
- Generate shipping notifications

### Business Intelligence

Track shipping metrics:
- Shipment volumes by carrier
- Average ship times
- Multi-shipment rates
- Carrier performance

### Inventory Planning

Analyze shipment patterns for:
- Demand forecasting
- Inventory allocation
- Shipping cost optimization
- Carrier selection

## Best Practices

1. **Real-time Updates**: Use orderEvent callbacks for immediate shipment notifications
2. **Multi-shipment Handling**: Always handle the possibility of multiple shipments per order
3. **Tracking Integration**: Provide customers with direct tracking links
4. **Error Handling**: Handle cases where tracking information may not be immediately available
5. **Data Retention**: Store shipment data for customer service and reporting needs