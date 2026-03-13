---
layout: docs
title: Application Setup
parent: Getting Started
nav_order: 1
---

Before connecting to Quipt users, you must register your application in Quipt to obtain the credentials required for the OAuth 1.0a authorization process.

## Step 1 — Create the Application

1. Navigate to [https://app.getquipt.com/#/Settings/Advanced](https://app.getquipt.com/#/Settings/Advanced).
2. Click the **+** button to the right of **Applications**.
3. Complete the application form:
   1. **Application Name** — Enter a name that identifies your application.
   2. **Description** — Briefly describe what your application does.
   3. **Global Access** — Select whether the application should be globally accessible to all Quipt users, or restricted to users within your organization.
   4. **Terms & Conditions** — If the application is intended for external use, enter your terms and conditions. Users will be required to accept these before authorizing.
   5. **Permissions** — Select the permissions your application will need to complete its intended actions. Only request permissions that are necessary.
4. Click **Save**.

On save, Quipt will generate a **Consumer Key** and **Secret Token**. Store these securely — they are the root credentials used to sign all OAuth requests for this application.

> Treat your Consumer Key and Secret Token like a password. Do not commit them to source control, expose them to client-side code, or transmit them outside of HTTPS.

## Step 2 — Authorize a User

Once you have a Consumer Key and Secret, you must authorize at least one user within your organization before making API calls on their behalf. This generates the user-level token and secret required to complete the OAuth signature.

1. Navigate to [https://app.getquipt.com/#/Applications](https://app.getquipt.com/#/Applications).
2. Locate your application in the list and click **Authorize**.
3. Review and accept the terms and conditions, and grant the requested permissions to the application.
4. Click **Save**.

On save, Quipt will generate a **Token** and **Secret** specific to the authorizing user. Together with your Consumer Key and Consumer Secret, these four values are used to sign every API request made on that user's behalf.

| Credential | Source | Used for |
|---|---|---|
| `consumer_key` | Application registration (Step 1) | Identifies your application |
| `consumer_secret` | Application registration (Step 1) | Signs requests |
| `oauth_token` | User authorization (Step 2) | Identifies the authorized user |
| `oauth_token_secret` | User authorization (Step 2) | Signs requests on the user's behalf |

## Next Steps

With your credentials in hand, you are ready to begin making signed API requests. See [OAuth 1.0a Authorization](oauth-authorization.md) for a full walkthrough of the OAuth 1.0a flow and request signing process.
