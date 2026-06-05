# Release Notes

This section contains the complete changelog and release history for the MasonHub API and Order Management System. Each release includes detailed information about new features, API changes, bug fixes, and important updates.

## Latest Releases

- [SKU Barcodes - March 18, 2019](sku-barcodes.md)
- [DataFactory: Random Inventory Statuses - March 12, 2019](datafactory-random-inventory-statuses.md)
- [Updated Return Reason Codes - March 12, 2019](updated-return-reason-codes.md)
- [RMA Changes (Return Labels) - March 6, 2019](rma-changes-return-labels.md)
- [Data Factory Multi Shipment - March 6, 2019](data-factory-multi-shipment.md)
- [SKU API Changes - February 12, 2019](sku-changes.md)

## 2018 Releases

- [Order Shipment GET - December 29, 2018](shipment-get.md)
- [RMA Data Factory - December 28, 2018](rma-data-factory.md)
- [Inbound Shipments API Complete - December 27, 2018](inbound-shipment-api-complete.md)
- [Inbound Shipments Delete and Events - December 26, 2018](inbound-shipments-get-delete-and-event.md)
- [RMA Creation and List - December 25, 2018](rmas-create-and-list.md)
- [Expected Inventory Calculation - December 18, 2018](inbound-shipments-expected-inventory.md)
- [Order Shorts - December 18, 2018](order-shorts.md)
- [Inbound Shipment Creation - December 17, 2018](inbound-shipments-create-and-update.md)
- [Order Cancelations - December 14, 2018](order-cancel.md)
- [Order Update Callbacks - December 13, 2018](order-update-callbacks.md)
- [Order Updates - December 7, 2018](order-updates.md)
- [Order Events, Kitting and Other Improvements - December 3, 2018](order-events-kitting.md)
- [Order GET and Callback Events - November 28, 2018](order-get.md)
- [Universal Resource Identifiers - November 21, 2018](universal-resource-identifier.md)
- [Order Creation and Inventory Totals - November 19, 2018](order-creation-and-inventory-changes.md)

## Release Categories

### Major Features
- **Kitting Support**: Kit SKU functionality with pre-built and kit-to-ship options
- **Return Management**: Complete RMA system with return label generation
- **Inbound Shipments**: ASN management with receiving workflows
- **Order Management**: Full order lifecycle with asynchronous operations
- **Callback System**: Real-time webhooks for all major events

### API Enhancements
- **Multi-shipment Orders**: Orders can ship in multiple packages
- **Order Updates**: Asynchronous order modification system
- **Order Cancellation**: Comprehensive cancel request workflow
- **SKU Barcode Expansion**: Multiple barcode support per SKU
- **DataFactory Testing**: Complete testing simulation utilities

### Infrastructure Improvements
- **Stack Trace System**: Complete audit trail for all operations
- **Universal Resource Identifiers**: System-wide resource linking
- **Inventory Calculations**: Real-time inventory totals and projections
- **Callback Holds**: Temporary suspension of webhook delivery

## Breaking Changes

### March 6, 2019 - RMA Changes
- `customer_order_id` no longer required for RMA creation
- `generate_return_label` now defaults to `true`
- Customer address required when generating return labels

### February 12, 2019 - SKU API Changes  
- `vendor_style` field now required (1-40 characters)
- `dim_width` field now required
- New optional fields: `vendor_size`, `country_of_origin`, `commodity_code`

## Upcoming Features

The MasonHub team continuously works on new features and improvements. Check back regularly for updates on:

- Enhanced reporting capabilities
- Advanced inventory optimization
- International shipping support
- Additional carrier integrations
- Mobile application support

## Support

For questions about any release or to report issues:

**Email**: support@masonhub.co  
**Documentation**: [https://docs.masonhub.co](https://docs.masonhub.co)

## Changelog Format

Each release note includes:

- **Release Date**: When the feature was deployed
- **Feature Description**: What was added or changed
- **API Impact**: Any breaking changes or new endpoints
- **Migration Guide**: How to update existing integrations
- **Examples**: Code samples showing new functionality
- **Contact Information**: Who to reach for questions

This ensures you have all the information needed to understand and implement new features as they become available.