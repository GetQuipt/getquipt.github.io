---
layout: docs
title: Channel API
nav_order: 3
---

The Channel API is intended for partners that list and sell inventory through Quipt. The sections below cover each integration area available to automate the entire fulfillment life cycle — from surfacing merchant inventory on your storefront through to managing post-order requests.

A [Postman collection](https://getquipt.github.io/images/PostmanQuiptRetailerAPI.zip) is available to explore the endpoints interactively.

> Use cases throughout this document are for demonstration purposes only and are not intended to cover every possible error state.

---

## Overview

A fully integrated retailer implementation spans three subsystems:

| Subsystem | Purpose |
|---|---|
| [Inventory](#inventory) | Discover merchant inventory, synchronize availability, and surface listings on your storefront |
| [Orders](#orders) | Place orders, track shipment status, communicate with the merchant, and request cancellations |
| [Requests](#requests) | Initiate and track post-order returns, communicate updates, and manage cancellations |

The Retailer API is complementary to the [Vendor API](vendor-api.md). Where the Vendor API describes the fulfillment workflow from the seller's perspective, the Retailer API describes it from the buyer's perspective. Understanding both is useful when troubleshooting cross-party workflows such as order acknowledgements and return credits.

---

## Inventory

> How to manage virtual inventory.

As a retailer, you do not own the inventory — you surface it. The Inventory subsystem provides the mechanisms to discover new items listed by merchants, keep availability and pricing current, and respond to changes as merchants update their catalog.

There are two approaches to staying synchronized:

- **Polling** — Periodically query for new items or availability changes and process them in batches. This is the simpler approach and works well when near-real-time accuracy is not critical.
- **Notifications** — Subscribe to inventory change events so your system is alerted as updates occur, rather than discovering them on the next poll cycle. This reduces latency and API call volume for high-frequency catalogs. See the [Notifications](api-reference.md#operations-tag-Notification_API) reference for setup details.

Acknowledging items is an important part of the inventory workflow. An acknowledgement signals to Quipt that your system has received and processed the item or update, which allows it to be cleared from the queue and prevents it from being returned in subsequent calls.

**Use Case 1 — Get new items and acknowledge**

Poll for items that have been newly listed by merchants and are not yet known to your system. For each item retrieved, post an acknowledgement to confirm receipt. Store the item details locally to power your storefront without requiring a live API call on every page load.

![Get new items and acknowledge](https://getquipt.github.io/images/retailer-inventory-new.png)

**Use Case 2 — Get availability and acknowledge**

Poll for availability changes on items already in your catalog. Availability changes include quantity updates and price changes. Acknowledge each change after processing to clear it from the queue. This is the primary mechanism for keeping your storefront's stock levels and pricing accurate.

![Get availability and acknowledge](https://getquipt.github.io/images/retailer-inventory-update.png)

**Use Case 3 — Using Notifications: get recently updated items, acknowledge, and archive**

Rather than polling on a fixed schedule, use the Notifications subsystem to receive alerts when items are updated. Retrieve the flagged items, process the changes, acknowledge receipt, and archive the notification to remove it from the active queue. This pattern is recommended for catalogs where pricing or availability changes frequently, as it reduces unnecessary polling and ensures updates are reflected on your storefront promptly.

![Get recently updated items via notifications, acknowledge, and archive](https://getquipt.github.io/images/retailer-inventory-notification-update.png)

For full details on available endpoints see the [Inventory](api-reference.md#operations-tag-Channel_API_-_Virtual_Inventory) reference.

---

## Orders

> How to place an order and track status changes.

The Orders subsystem covers the full lifecycle of a purchase order from the retailer's perspective — placing the order, tracking its progress through the merchant's fulfillment workflow, communicating with the merchant, and canceling if necessary.

The typical order flow is:

1. **Create the order** — Submit a purchase order to the merchant for a specific item and quantity.
2. **Await acknowledgement** — The merchant will accept or reject the order. Monitor order status until an acknowledgement is received.
3. **Track shipment** — Once accepted and fulfilled, retrieve shipment details including carrier and tracking number.
4. **Communicate** — If questions arise during fulfillment, post messages on the order to create an auditable communication thread visible to both parties.
5. **Cancel if needed** — If the order needs to be canceled after submission, submit a cancellation request. Note that once a vendor has shipped the order, cancellation may no longer be possible.

> **Important:** A merchant rejection cancels the order immediately and cannot be undone. If an order is rejected that you believe should have been accepted, contact the merchant directly or raise a support request rather than resubmitting — duplicate orders can cause fulfillment issues.

**Use Case 1 — Create order**

Submit a new purchase order to Quipt for routing to the appropriate merchant. Include the item identifier, quantity, shipping address, and any retailer-specific reference numbers needed to reconcile the order in your system. Store the returned Quipt order identifier locally for use in subsequent status and shipment queries.

![Create order](https://getquipt.github.io/images/retailer-order-create.png)

**Use Case 2 — Get shipments**

Poll for shipment details on open orders. When the merchant ships the item, Quipt will have carrier and tracking information available. Retrieve this to update your order management system and notify your customer. Continue polling until all line items on the order have been shipped or the order is otherwise resolved.

![Get shipments](https://getquipt.github.io/images/retailer-order-status.png)

**Use Case 3 — Post message**

Post a message on an order to communicate with the merchant. Messages are attached to the order record and visible to both parties, creating a traceable thread for questions about fulfillment, special handling instructions, or issue resolution. This is preferable to out-of-band communication as it keeps the full context with the order.

![Post message on order](https://getquipt.github.io/images/retailer-order-message.png)

**Use Case 4 — Request cancellation**

Submit a cancellation request for an order that has not yet been shipped. The merchant will receive the request and confirm or deny it. Monitor the order status after submitting — a cancellation request does not guarantee cancellation, and the merchant may have already begun fulfillment.

![Request order cancellation](https://getquipt.github.io/images/retailer-order-cancel.png)

For full details on available endpoints see the [Orders](api-reference.md#operations-tag-Channel_API_-_Orders) reference.

---

## Requests

> How to create a new return and monitor for further action.

The Requests subsystem handles post-order exceptions initiated by the retailer. As a retailer, the primary request type you will work with is a **return** — initiated when a customer wishes to send an item back to the merchant.

The return workflow is collaborative. After the retailer submits a return request, the merchant must acknowledge it, and then both parties track the progress through inspection and credit. Because the workflow is non-linear and requires action from both sides, monitoring request status and communicating within the request record is important to keep the process moving.

The key stages of a retailer-initiated return are:

1. **Create the return request** — Submit the request with the order reference, reason for return, and any supporting details.
2. **Monitor status** — Poll the request for status changes. The merchant will acknowledge the request and, after receiving and inspecting the returned item, post a credit.
3. **Communicate** — Post messages on the request if additional context or clarification is needed.
4. **Cancel if needed** — If the return is no longer necessary (e.g. the customer changed their mind before shipping the item back), submit a cancellation request.

> Once the item has been shipped back to the merchant, canceling the return request is generally not possible. Ensure the customer genuinely intends to return the item before submitting the request.

**Use Case 1 — Create new return**

Submit a return request against a fulfilled order. Include the order identifier, the reason for the return, and any condition notes that will help the merchant assess the item on receipt. The request is routed to the merchant for acknowledgement.

![Create new return request](https://getquipt.github.io/images/retailer-request-return-create.png)

**Use Case 2 — Monitor requests**

Poll for status changes on open return requests. Status will progress as the merchant acknowledges the request, receives the item, and processes the credit. Store request state locally to avoid reprocessing resolved items and to drive any customer-facing status updates in your system.

![Monitor request status](https://getquipt.github.io/images/retailer-request-return-status.png)

**Use Case 3 — Post message**

Post a message on a request to communicate with the merchant. This is the same mechanism as order messaging and serves the same purpose — keeping communication in context and auditable. Use it to provide return shipping details, clarify the return reason, or follow up on a pending credit.

![Post message on request](https://getquipt.github.io/images/retailer-request-return-message.png)

**Use Case 4 — Request cancellation**

Submit a cancellation request for a return that is no longer needed. As with order cancellations, this is a request rather than an immediate action — the merchant will confirm or deny. Monitor the request status after submitting to confirm the outcome.

![Request cancellation of return](https://getquipt.github.io/images/retailer-request-return-cancel.png)

For full details on available endpoints see the [Requests](api-reference.md#operations-tag-Channel_API_-_Requests) reference.

---

## Next Steps

With an understanding of each subsystem, the recommended integration path for a new retailer is:

1. **[Set up your application](application-setup.md)** and obtain your OAuth credentials.
2. **[Authorize a user](oauth-authorization.md)** to generate the token pair needed to sign API requests.
3. Implement **Inventory** to begin surfacing merchant listings on your storefront.
4. Implement **Orders** to enable purchasing and order tracking.
5. Add **Requests** to support customer returns.

For context on how your orders and returns appear from the merchant's side, see the [Vendor API](vendor-api.md) overview.
