# Callback Events

This section defines the payloads MasonHub sends to registered callback endpoints. Each event type has a specific structure and contains relevant data for the event.

## Common Payload Structure

All callback events follow this structure:

```json
{
  "callback_url": "https://client.com/api/webhook",
  "message_type": "eventType", 
  "message_id": "unique-event-id",
  "api_version": "1.0",
  "data": [
    // Event-specific data array
  ]
}
```

## SKU Inventory Change Events

Sent when inventory levels change for SKUs across any location or status.

### Event Structure

```json
{
  "callback_url": "https://client.com/api/skuInventoryChange",
  "message_type": "skuInventoryChange",
  "message_id": "e58769ec-c4cf-47f9-b80a-378a9b85aeae",
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

### Triggers

- Inventory receipt from inbound shipments
- Order fulfillment and allocation
- Inventory adjustments and corrections
- Return processing
- Kit assembly/disassembly

## Order Events

Provides real-time order lifecycle updates including status changes, shipments, and shorts.

### Event Structure

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
      ],
      "shorts": [
        {
          "sku_customer_id": "pants3422", 
          "quantity": 1,
          "reason": "damage"
        }
      ]
    }
  ]
}
```

### Order Statuses

- **atWarehouse**: Order received at distribution center
- **inProcess**: Order being picked and packed
- **packed**: Order packed, ready to ship
- **fulfilled**: Order shipped to customer
- **delivered**: Order delivered to customer
- **canceled**: Order canceled

### Multi-Shipment Support

Orders may ship in multiple packages. The `shipments` array contains all shipments for the order.

## Order Update Resolutions

Sent when asynchronous order update requests are resolved (success or failure).

### Event Structure

```json
{
  "callback_url": "https://client.com/api/orderUpdateResolution",
  "message_type": "orderUpdateResolution",
  "message_id": "0896116f-e54b-4756-9d3e-1b0c4a25d821", 
  "data": [
    {
      "id": "0b744e29-b668-4486-85dd-82528b5da0dd",
      "customer_order_id": "12345",
      "status": "success",
      "reason": null,
      "created_at": "2018-08-05T17:32:28Z",
      "updated_at": "2018-08-05T17:32:28Z"
    }
  ]
}
```

### Status Values

- **success**: Update applied successfully
- **failure**: Update failed (reason provided)

## Order Cancel Resolutions  

Sent when asynchronous order cancel requests are resolved.

### Event Structure

```json
{
  "callback_url": "https://client.com/api/orderCancelResolution",
  "message_type": "orderCancelResolution",
  "message_id": "0896116f-e54b-4756-9d3e-1b0c4a25d821",
  "data": [
    {
      "id": "0b744e29-b668-4486-85dd-82528b5da0dd", 
      "customer_order_id": "12345",
      "status": "failure",
      "reason": "Order already packed",
      "created_at": "2018-08-05T17:32:28Z",
      "updated_at": "2018-08-05T17:32:28Z"
    }
  ]
}
```

## Inbound Shipment Events

Sent when inbound shipment status changes during the receiving process.

### Event Structure

```json
{
  "callback_url": "https://client.com/api/inboundShipmentEvent",
  "message_type": "inboundShipmentEvent",
  "message_id": "0896116f-e54b-4756-9d3e-1b0c4a25d821",
  "data": [
    {
      "id": "33455ccc-40f5-431b-81a1-44098f8a92bd",
      "customer_identifier": "shipment32432",
      "status": "receivingComplete",
      "as_of": "2018-12-11T15:53:23Z",
      "details": {
        "inventory_location_address": "MasonHub Rancho | 1 Mason Way Rancho Cucomunga, CA 97130",
        "arrived_at": "2018-12-11T09:15:00Z",
        "started_receiving_at": "2018-12-11T11:01:24Z",
        "finished_receiving_at": "2018-12-11T15:53:23Z",
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
  ]
}
```

### Shipment Statuses

- **onDock**: Shipment arrived at facility
- **receivingStarted**: Receiving process begun
- **receivingCompleted**: All items processed

## RMA Events

Sent when return status changes or items are received.

### Event Structure  

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
      "line_items": [
        {
          "customer_order_id": "54321",
          "customer_sku_id": "shirts872340",
          "quantity": 2,
          "quantity_received": 2,
          "return_reason_code": "tooBig",
          "return_inventory_status": "quality-control",
          "not_on_original_rma": false
        }
      ]
    }
  ]
}
```

### RMA Statuses

- **tendered**: Return tendered to carrier
- **inTransit**: Return in transit to facility  
- **delivered**: Return delivered to facility
- **received**: Return items processed
- **undeliverable**: Return undeliverable

## Snapshot Ready

Sent when full inventory snapshots are ready for download.

### Event Structure

```json
{
  "callback_url": "https://client.com/api/snapshotReady",
  "message_type": "snapshotReady", 
  "message_id": "0896116f-e54b-4756-9d3e-1b0c4a25d821",
  "data": {
    "snapshot_id": "snapshot-uuid-123",
    "snapshot_type": "full",
    "as_of": "2019-08-05T08:15:30Z",
    "download_url": "https://app.masonhub.co/downloads/snapshot-123.csv",
    "expires_at": "2019-08-12T08:15:30Z"
  }
}
```

## Implementation Guidelines

### Webhook Response Requirements

Your webhook endpoints should:

```python
@app.route('/webhook/masonhub', methods=['POST'])
def handle_webhook():
    try:
        payload = request.get_json()
        message_type = payload['message_type']
        message_id = payload['message_id'] 
        data = payload['data']
        
        # Process event based on type
        if message_type == 'skuInventoryChange':
            update_inventory_levels(data)
        elif message_type == 'orderEvent':
            update_order_status(data)
            
        return 'OK', 200
        
    except Exception as e:
        logger.error(f"Webhook error: {e}")
        return 'Error', 500
```

### Best Practices

1. **Idempotency**: Use `message_id` to prevent duplicate processing
2. **Fast Response**: Process events quickly, queue heavy work
3. **Error Handling**: Return appropriate HTTP status codes
4. **Logging**: Log all events for debugging and auditing
5. **Validation**: Verify payload structure and authenticity
6. **Monitoring**: Monitor webhook endpoint health and latency