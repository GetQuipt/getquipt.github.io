---
layout: docs
title: Vendor API
nav_order: 2
---

The Vendor API is intended for partners that sell inventory through Quipt. The sections below cover each integration area available to automate the entire fulfillment life cycle — from listing inventory through to payment reconciliation.

A [Postman collection](https://getquipt.github.io/images/PostmanQuiptMerchantAPI.zip) is available to explore the endpoints interactively.

> Use cases throughout this document are for demonstration purposes only and are not intended to cover every possible error state.

---

## Overview

A fully integrated merchant implementation touches five subsystems, each building on the previous:

| Subsystem | Purpose |
|---|---|
| [Inventory](#inventory) | List products and keep availability synchronized |
| [Orders](#orders) | Receive, acknowledge, ship, and invoice orders |
| [Shipments](#shipments) | Manage Quipt-generated labels, packing slips, and shipment status |
| [Requests](#requests) | Handle post-order events: returns, intercepts, lost shipments, credits |
| [Accounting](#accounting) | View, pay, and reconcile receivable transactions |

Not every subsystem needs to be implemented. A minimal integration might only use Inventory and Orders. Add Shipments if Quipt generates your labels, and Requests and Accounting as your operational maturity grows.

---

## Inventory

> List and manage your inventory.

Inventory is the most flexible subsystem with respect to required data. Every partner has a different degree of automation available, so the API is designed to accommodate varying levels of completeness.

- **Minimal integration** — If your system can only export a SKU and quantity, that is sufficient to keep availability synchronized. Complete the product listing manually in the Quipt web application, then use the API solely to update available quantity going forward.
- **Full catalog integration** — Partners with richer product data can automate the entire listing process, including product attributes, pricing, and condition details, enabling a more fluid onboarding experience for new SKUs.

The key principle: _start with what you have_. A partial automation is better than no automation, and the API can be extended incrementally as more data becomes available.

**Use Case 1 — Create or update inventory items**

Your system pushes new or updated SKUs to Quipt. If the item already exists it is updated; if it does not exist it is created. This is the primary mechanism for keeping your Quipt catalog in sync with your internal product database.

![Create or update inventory items](https://getquipt.github.io/images/merchant-inventory-list.png)

**Use Case 2 — Monitor listed items and update cost if changed**

Your system periodically polls listed inventory and compares cost against your internal pricing. If the cost has changed, an update is pushed to Quipt. This is useful for partners whose pricing fluctuates frequently and who need Quipt to reflect current cost at all times.

![Monitor listed inventory and update cost](https://getquipt.github.io/images/merchant-inventory-monitor.png)

For full details on available endpoints see the [Inventory](api-reference.md#operations-tag-Vendor_API_-_Inventory) reference.

---

## Orders

> An item has sold. How do we get the order details, acknowledge, ship, and invoice?

The order subsystem covers the complete lifecycle of a purchase order from the moment it is cut to you through to invoicing. The expected sequence is:

1. **Poll for new orders** — Retrieve orders that have been assigned to your merchant account.
2. **Acknowledge** — Confirm receipt of each order with either an acceptance or a rejection.
3. **Ship** — Once accepted, fulfill and ship the item in a timely manner and post the tracking details.
4. **Invoice** — Submit the invoice to close out the financial transaction.

**Acknowledgement rules to be aware of:**
- An acceptance confirms you will fulfill the order and triggers the retailer's expectation of shipment.
- A rejection cancels the order immediately and **cannot be undone**. Only reject an order if you are certain it cannot be fulfilled.
- If an order was initially accepted but later cannot be shipped (e.g. item found to be out of stock after acceptance), submit a new rejection acknowledgement as soon as possible.

**Invoicing options** — There are three ways to invoice a completed order. Choose the one that fits your workflow:

| Method | Description |
|---|---|
| Invoice service | Post invoice details independently after shipment |
| Ship service | Include invoice details in the same request as the shipment |
| Auto Invoice | Enable in Settings to have Quipt invoice automatically on shipment |

**Use Case 1 — Get new orders cut to merchant**

Poll for orders in a new or open state and store them locally for processing. Storing orders locally allows downstream processes (shipment, invoicing) to operate without redundant API calls.

![Get new orders cut to merchant](https://getquipt.github.io/images/merchant-orders-list.png)

**Use Case 2 — Order shipped by merchant**

After fulfillment, post the shipment details (carrier, tracking number, ship date) to Quipt. The retailer is notified and the order moves to a shipped state.

![Order shipped by merchant](https://getquipt.github.io/images/merchant-orders-ship.png)

**Use Case 3 — Order canceled by merchant**

If an accepted order cannot be fulfilled, submit a rejection acknowledgement to cancel it. This should be done as early as possible to allow the retailer to source the item elsewhere.

![Order canceled by merchant](https://getquipt.github.io/images/merchant-orders-cancel.png)

**Use Case 4 — Order invoiced by the merchant**

After shipment, post the invoice details using the invoice service. Note that invoicing can alternatively be handled via the ship service or Auto Invoice — this use case demonstrates the standalone invoice service approach.

![Order invoiced by merchant](https://getquipt.github.io/images/merchant-orders-invoice.png)

For full details on available endpoints see the [Orders](api-reference.md#operations-tag-Vendor_API_-_Orders) reference.

---

## Shipments

> Does Quipt generate your shipment labels?

The Shipments subsystem is only relevant if you ship via Quipt (i.e. Quipt generates your labels and packing slips). If you use your own shipping infrastructure, this subsystem can be skipped.

When shipping via Quipt, the API provides:
- **Label and packing slip retrieval** — Download generated documents attached to your orders.
- **Status acknowledgements** — Mark shipments as printed, on dock, or picked up to give Quipt and the retailer visibility into your fulfillment progress.
- **Per-shipment invoicing** — Invoice at the shipment level rather than the order level, which is useful for orders that ship in multiple packages.

> **Prerequisite:** Both use cases below assume Orders Use Case 1 has been implemented and open purchase orders are stored in a local data store. The shipment API references order identifiers — without locally stored orders, you would need to look up order details on every call.

**Use Case 1 — Get new shipments**

Poll for shipments associated with open purchase orders. Download labels and packing slips for printing, then acknowledge receipt to update shipment status.

![Get new shipments](https://getquipt.github.io/images/merchant-shipments-list.png)

**Use Case 2 — Invoice completed shipments**

After a shipment has been picked up or handed to the carrier, post invoice details at the shipment level. This is the preferred invoicing approach when orders ship in multiple packages, as each package can be invoiced independently.

![Invoice completed shipments](https://getquipt.github.io/images/merchant-shipments-invoice.png)

For full details on available endpoints see the [Shipments](api-reference.md#operations-tag-Vendor_API_-_Shipments) reference.

---

## Requests

> Something happened to an order. How do we track and manage it?

Requests are post-order events that require action from one or both parties. Unlike orders — where the workflow is largely linear — requests are collaborative and non-linear: the required steps depend on the request type, and both the merchant and retailer may have actions to take at different points.

A request can be one of five types:

| Type | Description |
|---|---|
| **Return** | An item is being sent back by the customer |
| **Intercept** | A shipment in transit needs to be recalled or redirected |
| **Lost Shipment** | A shipped item cannot be located and is presumed lost |
| **Partial Credit** | A partial refund is applied without a physical return |
| **Missing Accessory** | An item was received incomplete (e.g. missing a cable or manual) |

### Returns

Returns are the most common request type. There are two points of initiation:

**Retailer-initiated return** — The customer contacts the retailer to request a return. The retailer approves and submits the request to Quipt. From that point, the item is expected to be shipped back to the merchant. Upon receipt and inspection, the merchant issues a credit to the retailer.

![Retailer-initiated return flow](https://getquipt.github.io/images/merchant-requests-merchant-flow-retailer-initiated.png)

**Merchant-initiated (vendor-initiated) return** — The item is returned to the merchant without prior notification, such as a refused or undeliverable shipment. In this case the merchant initiates the return request in Quipt to begin the credit process.

![Merchant-initiated return flow](https://getquipt.github.io/images/merchant-requests-merchant-flow-merchant-initiated.png)

### Common Use Cases

**Use Case 1 — Get new requests assigned to the merchant**

Poll for open requests in a new state. Store them locally and route them to the appropriate internal workflow based on request type. Centralizing request retrieval this way avoids polling separately per request type and simplifies queue management.

![Get new requests assigned to merchant](https://getquipt.github.io/images/merchant-requests-list.png)

**Use Case 2 — Merchant-requested archive of a request**

Once a request has been resolved and no further action is required, archive it to remove it from the active queue. Archiving does not delete the request — it remains accessible for audit and reconciliation purposes.

![Merchant archive of a request](https://getquipt.github.io/images/merchant-requests-archive.png)

For full details on available endpoints see the [Requests](api-reference.md#operations-tag-Vendor_API_-_Requests) reference.

---

## Accounting

> A transaction is completed. How do we manage it until we are paid and settled?

The Accounting subsystem aggregates completed order and request activity into a ledger of receivable transactions. It provides the tools to view outstanding balances, apply payments, and reconcile the ledger against your internal accounts receivable records.

As orders are fulfilled and requests are resolved, the financial details are automatically posted to the ledger. The API allows you to query those entries and manage their lifecycle without requiring manual reconciliation in the Quipt web interface.

**Use Case 1 — Get open receivable transactions**

Retrieve all transactions in an open (unpaid) state. Use this to drive your internal AR process — match entries against expected revenue, flag discrepancies, and prepare payment batches.

![Get open receivable transactions](https://getquipt.github.io/images/merchant-accounting-list.png)

**Use Case 2 — Apply a payment to receivable transactions**

Post payment details against one or more open transactions to mark them as settled. This is typically run after a payment batch has been processed, allowing your AR records and the Quipt ledger to stay in sync.

![Apply payment to receivable transactions](https://getquipt.github.io/images/merchant-accounting-apply.png)

For full details on available endpoints see the [Accounting](api-reference.md#operations-tag-Vendor_API_-_Accounting) reference.

---

## Next Steps

With an understanding of each subsystem, the recommended integration path for a new merchant is:

1. **[Set up your application](application-setup.md)** and obtain your OAuth credentials.
2. **[Authorize a user](oauth-authorization.md)** to generate the token pair needed to sign API requests.
3. Implement **Inventory** to get your catalog listed.
4. Implement **Orders** to begin receiving and fulfilling purchase orders.
5. Add **Shipments** if you ship via Quipt.
6. Add **Requests** to handle post-order exceptions.
7. Add **Accounting** to automate payment reconciliation.
