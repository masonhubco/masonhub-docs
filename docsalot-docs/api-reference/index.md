# MasonHub Documentation

Welcome to the MasonHub API documentation. This documentation covers the complete MasonHub Order Management System API and related resources.

## API Documentation

- [Complete API Reference](api-reference.md) - Full API specification with all endpoints
- [API Design Philosophy](api-design-philosophy.md) - Core principles behind the API design
- [Security](security.md) - Authentication and security information
- [Catalog](catalog.md) - SKU management endpoints
- [Inventory](inventory.md) - Inventory management and snapshots
- [Inbound Shipments](inbound-shipments.md) - ASN (Advance Ship Notice) management
- [Orders](orders.md) - Order creation and management
- [Shipments](shipments.md) - Outbound shipment tracking
- [Returns](returns.md) - RMA (Return Merchandise Authorization) management
- [Callbacks](callbacks.md) - Webhook management for real-time events
- [Callback Events](callback-events.md) - Event payloads and specifications
- [DataFactory](datafactory.md) - Testing utilities for simulating operations
- [Tokens](tokens.md) - Bearer token management

## Additional Resources

- [The Stack Trace](stack-trace.md) - Understanding MasonHub's logging and audit trail system
- [Release Notes](releases/index.md) - Complete changelog and release history

## Getting Started

The MasonHub API is a comprehensive Order Management System (OMS) that provides:

- **Everything is Really Just Search** - GET methods use search parameters for composability
- **Always an Array** - All methods accept/return arrays for batch operations  
- **Summary vs Detail** - Use `list_type=detail|summary` for response detail level
- **RESTful Design** - GET, POST, PUT, DELETE operations (No PATCH)
- **Robust Error Handling** - Bulk operations are non-blocking with individual success/failure status
- **RFC3339 Date Format** - Consistent date/time formatting throughout

## Authentication

All API endpoints require HTTPS with a valid JWT Bearer token. See the [Security](security.md) section for detailed authentication requirements.

## Base URL

- **Sandbox**: `https://sandbox.masonhub.co/{account_slug}/api/v1/`
- **Production**: `https://app.masonhub.co/{account_slug}/api/v1/`

For support, contact: support@masonhub.co