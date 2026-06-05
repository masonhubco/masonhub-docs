# DataFactory

DataFactory is MasonHub's testing and simulation utility that allows you to trigger various OMS actions and callbacks in the sandbox environment. This powerful tool helps you test integration workflows and simulate real-world scenarios.

## Overview

DataFactory provides endpoints to:
- Simulate inventory changes and receipts
- Trigger order fulfillment and shipping
- Generate callback events for testing
- Create realistic test scenarios
- Validate integration workflows

**Important**: DataFactory is only available in the sandbox environment.

## Inventory Simulation

### Create Available Inventory

**POST** `/data_factory/createAvailable`

Simulate adding inventory in 'available' status.

#### Request Body Schema

```json
[
  {
    "sku_id": "e5d00ff6-b34a-496e-9559-eb90e99463af",
    "quantity": 100
  }
]
```

This will:
- Add inventory to the specified SKU
- Trigger **skuInventoryChange** callbacks
- Update total available-to-sell quantities

### Lose Inventory

**POST** `/data_factory/loseInventory`

Simulate inventory loss by decrementing available and incrementing lost status.

#### Request Body Schema

```json
[
  {
    "sku_id": "e5d00ff6-b34a-496e-9559-eb90e99463af", 
    "quantity": 10
  }
]
```

## Order Operations

### Ship Order

**POST** `/data_factory/shipOrder`

Simulate order fulfillment and shipping with optional shorts and multi-shipment support.

#### Request Body Schema

```json
[
  {
    "order_customer_id": "12345",
    "shorts": ["sku1", "sku2"],
    "number_of_shipments": 2
  }
]
```

#### Parameters

| Field | Type | Description |
|-------|------|-------------|
| `order_customer_id` | string | Customer order identifier |
| `shorts` | array | SKU identifiers to short |
| `number_of_shipments` | integer | Number of shipments to create |

#### Behavior

- Changes order status to 'fulfilled'
- Creates shipment records with tracking
- Generates **orderEvent** callbacks
- Handles shorts and multi-shipment scenarios

### Order Shipment Event

**POST** `/data_factory/orderShipmentEvent`

Simulate carrier delivery events for existing shipments.

#### Request Body Schema

```json
[
  {
    "tracking_number": "1z324897234nferg45",
    "event": "delivered"
  }
]
```

#### Available Events

- **tendered**: Package tendered to carrier
- **inTransit**: Package in transit
- **delivered**: Package delivered
- **undeliverable**: Package undeliverable

### Resolve Order Update

**POST** `/data_factory/resolveOrderUpdate`

Force resolution of pending order update requests.

#### Request Body Schema

```json
[
  {
    "order_update_request_id": "0b744e29-b668-4486-85dd-82528b5da0dd",
    "resolution": "success"
  }
]
```

#### Resolution Values

- **success**: Update applied successfully
- **failure**: Update failed

### Resolve Order Cancel

**POST** `/data_factory/resolveOrderCancel`

Force resolution of pending order cancel requests.

#### Request Body Schema

```json
[
  {
    "order_cancel_request_id": "0b744e29-b668-4486-85dd-82528b5da0dd", 
    "resolution": "success"
  }
]
```

## Inbound Shipment Simulation

### Trigger Inbound Shipment Event

**POST** `/data_factory/inboundShipmentEvent`

Simulate inbound shipment receiving events.

#### Request Body Schema

```json
[
  {
    "customer_identifier": "shipment32432",
    "event": "receivingCompleted",
    "randomize_receipt_qty": true,
    "randomize_inventory_statuses": true
  }
]
```

#### Parameters

| Field | Type | Description |
|-------|------|-------------|
| `customer_identifier` | string | Shipment identifier |
| `event` | string | Event type |
| `randomize_receipt_qty` | boolean | Randomize received quantities |
| `randomize_inventory_statuses` | boolean | Random receiving statuses |

#### Available Events

- **onDock**: Shipment arrived at facility
- **receivingStarted**: Receiving process started  
- **receivingCompleted**: Receiving process completed

#### Randomization Options

When `randomize_inventory_statuses` is true, items are received into random statuses:
- available
- damaged
- quality-control
- refurbishing
- under-investigation

## RMA Simulation

### RMA Event

**POST** `/data_factory/rmaEvent`

Simulate RMA receiving and processing events.

#### Request Body Schema

```json
[
  {
    "customer_identifier": "rma123",
    "event": "receiveFull",
    "randomize_inventory_statuses": true
  }
]
```

#### Parameters

| Field | Type | Description |
|-------|------|-------------|
| `customer_identifier` | string | RMA identifier |
| `event` | string | RMA event type |
| `randomize_inventory_statuses` | boolean | Random return statuses |

#### Available Events

- **receiveFull**: Receive all expected items
- **receivePartial**: Receive random subset of items
- **tendered**: Return tendered to carrier
- **inTransit**: Return in transit
- **delivered**: Return delivered to facility

## Testing Workflows

### Basic Integration Test

1. **Create SKUs**: Add catalog items
2. **Add Inventory**: Use `createAvailable` 
3. **Create Orders**: Submit test orders
4. **Ship Orders**: Use `shipOrder` to fulfill
5. **Verify Callbacks**: Confirm event delivery

### Multi-Shipment Testing

```json
{
  "order_customer_id": "test-order-123",
  "number_of_shipments": 3
}
```

This creates an order with 3 separate shipments, each with tracking information.

### Shorts Testing

```json
{
  "order_customer_id": "test-order-456", 
  "shorts": ["sku-001", "sku-002"]
}
```

Simulates shortage scenarios for testing backorder handling.

### Return Processing Test

1. **Create RMA**: Submit return authorization
2. **Trigger Receipt**: Use `rmaEvent` with `receiveFull`
3. **Verify Inventory**: Check inventory updates
4. **Confirm Callbacks**: Validate event delivery

## Advanced Scenarios

### Randomized Receiving

Test receiving variances:

```json
{
  "customer_identifier": "shipment-test",
  "event": "receivingCompleted",
  "randomize_receipt_qty": true,
  "randomize_inventory_statuses": true
}
```

This simulates:
- Over/under receipts
- Mixed inventory conditions
- Quality control scenarios

### Complex Order Flows

Test complete order lifecycle:

1. Create order with multiple line items
2. Trigger partial shorts
3. Multi-shipment fulfillment
4. Delivery confirmation
5. Return processing

## Callback Validation

DataFactory operations trigger the same callbacks as production:

### Expected Callbacks

- **skuInventoryChange**: After inventory operations
- **orderEvent**: After order operations
- **inboundShipmentEvent**: After receiving operations
- **rmaEvent**: After return operations
- **orderUpdateResolution**: After update resolutions
- **orderCancelResolution**: After cancel resolutions

### Testing Callback Delivery

1. Register callback URLs in sandbox
2. Trigger DataFactory operations
3. Verify callback payloads
4. Test error handling scenarios

## Best Practices

### Test Data Management

1. **Consistent Identifiers**: Use predictable test IDs
2. **Data Isolation**: Separate test scenarios
3. **Cleanup**: Reset test state between runs
4. **Documentation**: Document test scenarios

### Integration Testing

1. **End-to-End**: Test complete workflows
2. **Error Scenarios**: Test failure cases
3. **Performance**: Test with realistic volumes
4. **Concurrency**: Test parallel operations

### Callback Testing

1. **Idempotency**: Handle duplicate deliveries
2. **Ordering**: Don't assume event sequence
3. **Timeouts**: Handle delivery delays
4. **Authentication**: Verify callback security

## Limitations

- **Sandbox Only**: DataFactory is not available in production
- **Test Data**: Creates simulated data, not real operations
- **Rate Limits**: Standard API rate limits apply
- **State Consistency**: Maintains sandbox state consistency

DataFactory provides comprehensive testing capabilities to validate your MasonHub integration before going live with production data.