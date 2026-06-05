# Callbacks

The Callbacks API manages webhook subscriptions for real-time event notifications from MasonHub. Register callback URLs to receive immediate notifications when events occur in the system.

## Endpoints

### Get Callback URLs

**GET** `/callbacks`

Retrieve registered callback URLs filtered by message type.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `message_type` | string | Filter by message type or "all" | "all" |

#### Available Message Types

- **skuInventoryChange**: Inventory level changes
- **orderEvent**: Order status updates  
- **orderUpdateResolution**: Order update request results
- **orderCancelResolution**: Order cancel request results
- **inboundShipmentEvent**: Inbound shipment status changes
- **rmaEvent**: Return status updates
- **snapshotReady**: Inventory snapshot completion

#### Example Request

```http
GET /callbacks?message_type=skuInventoryChange
```

#### Example Response

```json
[
  {
    "id": "callback-uuid-123",
    "url": "https://api.clienturl.com/api/inventory_event",
    "message_type": "skuInventoryChange",
    "api_version": "1.0",
    "token": "encrypted_verification_token"
  }
]
```

### Create Callback URLs

**POST** `/callbacks`

Register new callback URLs for event notifications.

#### Request Body Schema

Array of callback objects:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | HTTPS endpoint URL |
| `message_type` | string | Yes | Event type to subscribe to |
| `api_version` | string | No | API version (default: "1.0") |
| `token` | string | No | Optional verification token |

#### Example Request

```json
[
  {
    "url": "https://api.clienturl.com/api/skuInventoryChange",
    "message_type": "skuInventoryChange",
    "api_version": "1.0",
    "token": "your_secure_verification_token"
  },
  {
    "url": "https://api.clienturl.com/api/orderEvent", 
    "message_type": "orderEvent",
    "api_version": "1.0"
  }
]
```

### Delete Callback URLs

**DELETE** `/callbacks`

Remove callback subscriptions by ID.

#### Request Body Schema

Array of callback IDs to delete:

```json
[
  "callback-uuid-123",
  "callback-uuid-456"
]
```

## Callback Security

### HTTPS Required

All callback URLs must use HTTPS for security. HTTP URLs will be rejected.

### Verification Tokens

Include optional tokens in callback registrations for verification:

```json
{
  "url": "https://api.clienturl.com/webhook",
  "message_type": "orderEvent",
  "token": "your_encrypted_verification_token"
}
```

The token will be included in callback requests for verification.

### Authentication

Callback requests from MasonHub include:
- User-Agent identification
- Optional verification token (if configured)
- Consistent payload format

