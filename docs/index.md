---
title: Getting Started
nav_order: 1
---

Click [here](api-reference.md) to jump into the API documentation, otherwise if you need some additional guidance continue reading.

## Get Started

Start [here](application-setup.html) to learn how to register your application, obtain your API credentials, and configure your environment to connect to the Quipt platform.

## [OAuth 1.0a Authorization](oauth-authorization.html)
Understand how to authenticate API requests using OAuth 1.0a, including signing requests and managing access tokens. Start [here](oauth-authorization.html) if you are unfamilar with this authorization protocal.

## Overview

The [Vendor API](vendor-api.md) is a robust and adaptable suite of tools designed to seamlessly integrate with your existing systems, enabling efficient and scalable sales management. By automating manual tasks, streamlining workflows, and providing real-time data insights, the API empowers businesses to:
* Enhance operational efficiency
* Reduce costs and operational overhead
* Drive sales growth and market reach
* Offer an improved customer experience
    
The [Channel API](channel-api.md) empowers organizations to automate their purchasing functions, transforming their procurement landscape for enhanced efficiency, cost savings, and control. By automating manual tasks, streamlining workflows, and providing real-time data insights, the API empowers businesses to:
* Search and compare products from diverse merchants through a centralized platform.
* Access real-time inventory data and pricing information.
* Streamline product selection and requisition processes.
* Create and submit purchase orders electronically to multiple retailers.
* Track order status, delivery updates, and invoices in real-time.
    
The [Notification API](application-api.md#notifications) is a set of endpoints to get application events. Important the user you are using to monitor the events must be subscribed using the [web application](https://app.getquipt.com/#/Settings/MySettings). 
    
The [Carrier API](api-reference.md#tag-Carrier_API) is a set of endpoints to get the available carriers and carrier methods that are used as part of the order workflows. They are used as part of the order and shipment endpoints. 
    
The [Taxonomy API](api-reference.md#tag-Taxonomy_API) is a set of endpoints to get the available carriers and carrier methods that are used as part of the order workflows. They are used as part of the order and shipment endpoints. 

All APIs with the exception of [Carrier API](api-reference.md#tag-Carrier_API) and [Taxonomy API](api-reference.md#tag-Taxonomy_API) require OAuth 1.0a for authorization management. Learn how to [connect to Quipt](application-setup.md).   
