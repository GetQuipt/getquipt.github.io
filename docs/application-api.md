---
layout: default
title: Application API
---

The Application API provides access to the aggregate data sets and cross-cutting services that underpin each subsystem. Rather than being tied to a specific workflow like orders or inventory, these services are available across all integration types and can be used by both vendors and channels.

The Application API currently covers four areas:

| Subsystem | Purpose |
|---|---|
| [Notifications](#notifications) | Subscribe to and monitor in-app events across all workflows |
| [Taxonomy](api-reference.md#operations-tag-Taxonomy_API) | Access the category and attribute structure used to classify inventory |
| [Carriers](api-reference.md#operations-tag-Carrier_API) | Retrieve available carriers and shipping methods |
| [Catalog](api-reference.md#operations-tag-Catalog_API) | Access the shared product catalog for reference data |

---

## Notifications

Users can subscribe to a defined set of events that occur throughout Quipt. When a subscribed event fires, a notification is sent to the user's account. The Notification API provides the endpoints to retrieve, manage, and archive those notifications programmatically.

### When to use Notifications

Notifications are the recommended alternative to polling for time-sensitive changes. Rather than repeatedly querying an endpoint to detect whether something has changed, your system can instead check the notification queue — or better, process notifications as they arrive — and only make the downstream detail call when there is actually something to act on.

Common use cases include:

- **Order workflow tracking** — Subscribe to `PurchaseOrder` or `SalesOrder` events to detect when an order is acknowledged, shipped, or requires action, without polling the orders endpoint continuously.
- **Inventory price and quantity monitoring** — Subscribe to `VirtualInventory` or `PhysicalInventory` events to detect cost or availability changes and synchronize your internal systems in near-real time.
- **Request lifecycle tracking** — Subscribe to `ChannelRequest` or `VendorRequest` events to monitor returns, intercepts, and lost shipments as they progress through each stage.
- **Promotional events** — Subscribe to `Promotion` events to react to promotions starting, ending, or changing status.

### Notification lifecycle

A notification moves through the following states:

| Status Code | State | Description |
|---|---|---|
| `1` | Unread | Newly received, not yet processed |
| `2` | Read | Retrieved and marked as read |
| `4` | Archived | Processed and removed from the active queue |

The recommended pattern for a polling-based integration is: retrieve unread notifications → process each one → mark as read or archive. Archiving is preferred once a notification has been fully handled, as it keeps the queue clean and prevents reprocessing.

---

### Use Cases

**Use Case 1 — Manage notifications in a custom client application**

Poll the notification queue for unread messages, display or process them within your application, and mark each as read or archive it once handled. This is the baseline pattern for any notification-driven integration.

![Get user notifications for management in a custom client](https://getquipt.github.io/images/notification-list.png)

**Use Case 2 — Subscribe to price change events and synchronize internal pricing**

Subscribe the user to the `VirtualInventory` Price Change event (event code `44`). Poll for unread `VirtualInventory` notifications filtered to that event. For each notification received, call `GET /inventory/virtual/id/{id}` using the `RecordId` to retrieve the updated item details, then apply the new pricing to your internal systems. Archive the notification once the update has been applied.

![Subscribe to price change, monitor notifications, look up item, and update pricing](https://getquipt.github.io/images/notification-monitor.png)

---

### Event Reference

The tables below list all available event types and codes that can be subscribed to and received as notifications.

#### Batch

| Event Code | Event Name |
|---|---|
| 10 | Status Changed |
| 11 | Completed |
| 12 | Failed |

#### Report

| Event Code | Event Name |
|---|---|
| 110 | Created |
| 111 | Status Change |
| 112 | Removed |
| 113 | Shared |
| 114 | Report Queued |
| 115 | Queued Report Status Change |
| 116 | Report Generated |
| 117 | Report Downloaded |

#### Blog

| Event Code | Event Name |
|---|---|
| 170 | Created |
| 171 | General Message Created |
| 172 | Application Update Message Created |
| 173 | Scheduled Downtime Message Created |
| 174 | Unscheduled Downtime Message Created |

#### PhysicalInventory

| Event Code | Event Name |
|---|---|
| 22 | Cost Change |
| 23 | Discontinued |
| 24 | Published |
| 25 | Out Of Stock |
| 26 | Add Stock |
| 27 | Quantity Change |
| 28 | Quantity Allocated Change |
| 29 | Outsourced |
| 30 | Status Change |

#### VirtualInventory

| Event Code | Event Name |
|---|---|
| 40 | Activation Change |
| 41 | Allocation Change |
| 42 | Promotion Initialized |
| 43 | Promotion Removed |
| 44 | Price Change |
| 45 | Catalog Status Change |
| 46 | Inventory Status Change |
| 47 | ARP Change |
| 48 | Promotion Change |

#### Feeds

| Event Code | Event Name |
|---|---|
| 40 | Activation Change |
| 41 | Allocation Change |
| 42 | Promotion Initialized |
| 43 | Promotion Removed |
| 44 | Price Change |
| 45 | Catalog Status Change |
| 46 | Inventory Status Change |
| 47 | ARP Change |
| 48 | Promo Change |

#### PurchaseOrder

| Event Code | Event Name |
|---|---|
| 50 | Status Change |
| 51 | Placed |
| 52 | Canceled |
| 53 | Price Change |
| 54 | Quantity Ordered Change |
| 55 | Quantity Back Ordered |
| 56 | Quantity Canceled |
| 57 | Ship To Change |
| 58 | Shipped |
| 59 | Action Required |
| 330 | Action Required – Invalid SKU |
| 331 | Action Required – Duplicate Order Detected |
| 332 | Action Required – Out of Stock |
| 333 | Action Required – Invalid Pricing |
| 334 | Action Required – Invalid Ship-To Address |
| 335 | Action Required – Invalid Carrier Method Selected |
| 336 | Action Required – Execution of Custom Code Failed |
| 337 | Action Required – Invalid Payment |
| 338 | Action Required – On Hold |
| 339 | Action Required – Late Back Order |
| 340 | Action Required – Late Shipment |
| 341 | Action Required – Late Shipment Pickup |
| 342 | Action Required – Warehouse Cancellation |
| 343 | Action Required – Shipment Exception |
| 344 | Action Required – Payment Capture Failed |
| 345 | Action Required – Payment Refund Failed |
| 346 | Action Required – Invalid Tracking Number |
| 347 | Action Required – Cancellation Requested |

#### SalesOrder

| Event Code | Event Name |
|---|---|
| 50 | Status Change |
| 51 | Placed |
| 52 | Canceled |
| 53 | Price Change |
| 54 | Quantity Ordered Change |
| 55 | Quantity Back Ordered |
| 56 | Quantity Canceled |
| 57 | Ship To Change |
| 58 | Shipped |
| 59 | Action Required |
| 330 | Action Required – Invalid SKU |
| 331 | Action Required – Duplicate Order Detected |
| 332 | Action Required – Out of Stock |
| 333 | Action Required – Invalid Pricing |
| 334 | Action Required – Invalid Ship-To Address |
| 335 | Action Required – Invalid Carrier Method Selected |
| 336 | Action Required – Execution of Custom Code Failed |
| 337 | Action Required – Invalid Payment |
| 338 | Action Required – On Hold |
| 339 | Action Required – Late Back Order |
| 340 | Action Required – Late Shipment |
| 341 | Action Required – Late Shipment Pickup |
| 342 | Action Required – Warehouse Cancellation |
| 343 | Action Required – Shipment Exception |
| 344 | Action Required – Payment Capture Failed |
| 345 | Action Required – Payment Refund Failed |
| 346 | Action Required – Invalid Tracking Number |
| 347 | Action Required – Cancellation Requested |

#### ChannelRequest

| Event Code | Event Name |
|---|---|
| 90 | Created |
| 91 | Tracking Change |
| 92 | Action Status Change |
| 93 | Status Change |
| 94 | Comment Change |
| 95 | Closed |
| 96 | Canceled |
| 97 | Action Required |
| 200 | Intercept Attempt |
| 201 | Intercept Failed |
| 202 | Intercept In-Route |
| 203 | Intercept Delivered |
| 210 | Lost Shipment Trace Requested |
| 211 | Lost Shipment Trace Failed |
| 212 | Lost Shipment In-Route |
| 213 | Lost Shipment Delivered |
| 214 | Lost Shipment Claim Opened |
| 215 | Lost Shipment Claim Paid |
| 216 | Lost Shipment Claim Denied |
| 220 | Missing Accessory Credit Offered |
| 221 | Missing Accessory Credit Countered |
| 222 | Missing Accessory Return Created |
| 223 | Missing Accessory Label Requested |
| 225 | Missing Accessory Label Generated |
| 226 | Missing Accessory Label Generation Error |
| 227 | Missing Accessory Credit Accepted |
| 228 | Missing Accessory Pickup Requested |
| 229 | Missing Accessory Shipped |
| 240 | Partial Credit Accepted |
| 241 | Partial Credit Counter |
| 242 | Partial Credit Impasse |
| 250 | Return Credited |
| 251 | Return Credit Offered |
| 252 | Return Exchanged |
| 253 | Return Received |
| 254 | Return Label Requested |
| 256 | Return Label Generated |
| 257 | Return Label Generation Error |
| 258 | Return Pickup Requested |
| 259 | Return Accepted |
| 260 | Warranty Check Failed |

#### VendorRequest

| Event Code | Event Name |
|---|---|
| 90 | Created |
| 91 | Tracking Change |
| 92 | Action Status Change |
| 93 | Status Change |
| 94 | Comment Change |
| 95 | Closed |
| 96 | Canceled |
| 97 | Action Required |
| 200 | Intercept Attempt |
| 201 | Intercept Failed |
| 202 | Intercept In-Route |
| 203 | Intercept Delivered |
| 210 | Lost Shipment Trace Requested |
| 211 | Lost Shipment Trace Failed |
| 212 | Lost Shipment In-Route |
| 213 | Lost Shipment Delivered |
| 214 | Lost Shipment Claim Opened |
| 215 | Lost Shipment Claim Paid |
| 216 | Lost Shipment Claim Denied |
| 220 | Missing Accessory Credit Offered |
| 221 | Missing Accessory Credit Countered |
| 222 | Missing Accessory Return Created |
| 223 | Missing Accessory Label Requested |
| 225 | Missing Accessory Label Generated |
| 226 | Missing Accessory Label Generation Error |
| 227 | Missing Accessory Credit Accepted |
| 228 | Missing Accessory Pickup Requested |
| 229 | Missing Accessory Shipped |
| 240 | Partial Credit Accepted |
| 241 | Partial Credit Counter |
| 242 | Partial Credit Impasse |
| 250 | Return Credited |
| 251 | Return Credit Offered |
| 252 | Return Exchanged |
| 253 | Return Received |
| 254 | Return Label Requested |
| 256 | Return Label Generated |
| 257 | Return Label Generation Error |
| 258 | Return Pickup Requested |
| 259 | Return Accepted |
| 260 | Warranty Check Failed |

#### Promotion

| Event Code | Event Name |
|---|---|
| 130 | Starting New |
| 131 | Ending |
| 132 | Ended |
| 133 | Invited |
| 134 | Accepted |
| 135 | Rejected |
| 136 | Cancellation Requested |
| 137 | Canceled |
| 138 | Status Change |

---

## Related

- [Channel API — Inventory](channel-api.md#inventory) — Describes how Notifications integrate into the channel inventory sync workflow.
- [Vendor API](vendor-api.md) — Notifications can be used to drive event-based processing across the vendor fulfillment lifecycle.
