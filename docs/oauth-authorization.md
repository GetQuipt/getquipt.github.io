---
title: OAuth 1.0a Authorization
parent: Getting Started
nav_order: 2
---

Quipt uses [OAuth 1.0a](http://tools.ietf.org/html/rfc5849) to authorize third-party applications to access the API on behalf of a Quipt user. OAuth 1.0a is a delegated authorization protocol — your application never handles the user's Quipt credentials directly. Instead, it exchanges signed requests for tokens that represent the user's permission to act on their behalf.

## Web Based Authorization

### Endpoints

| Purpose | Method | URL |
|---|---|---|
| Request temporary token | `POST` | `https://app.getquipt.com/oauth/initiate` |
| User authorization | `REDIRECT` | `https://app.getquipt.com/#/oauth/authorizing?oauth_token={token}` |
| Exchange for access token | `POST` | `https://app.getquipt.com/oauth/token` |

### Step 1 — Obtain a Temporary Token

Make a signed `POST` request to the initiate endpoint. Quipt will return a temporary `token` and `secret` that identify this authorization attempt.

```
POST https://app.getquipt.com/oauth/initiate
```

**Required OAuth parameters** (passed in the `Authorization` header):

| Parameter | Description |
|---|---|
| `oauth_consumer_key` | Your application's consumer key |
| `oauth_signature_method` | Signature method (e.g. `HMAC-SHA1`) |
| `oauth_timestamp` | Current UTC Unix timestamp |
| `oauth_nonce` | Unique random string for this request |
| `oauth_version` | `1.0` |
| `oauth_callback` | URL Quipt will redirect the user to after authorization |
| `oauth_signature` | Request signature (see [RFC 5849 §3.4](http://tools.ietf.org/html/rfc5849#section-3.4)) |

**Response:**

```
oauth_token=TEMPORARY_TOKEN&oauth_token_secret=TEMPORARY_SECRET&oauth_callback_confirmed=true
```

Store the `oauth_token_secret` securely — you will need it to sign the access token request in Step 3.

---

### Step 2 — Redirect the User to Quipt

Redirect the user's browser to the Quipt authorization page, appending the temporary token from Step 1:

```
https://app.getquipt.com/#/oauth/authorizing?oauth_token=TEMPORARY_TOKEN
```

Quipt will prompt the user to log in and approve access for your application. If the user approves, Quipt redirects them back to the `oauth_callback` URL you provided in Step 1, with the following query parameters appended:

```
https://your-callback-url?oauth_token=TEMPORARY_TOKEN&oauth_verifier=VERIFIER_CODE
```

Capture and store the `oauth_verifier` value — it is required in Step 3.

> If the user denies access, Quipt redirects to your callback with `oauth_problem=user_refused`. Your application should handle this case gracefully.

---

### Step 3 — Exchange for an Access Token

Make a signed `POST` request to the token endpoint, using the temporary token and verifier obtained in Step 2. Sign the request using both your consumer secret and the temporary token secret from Step 1.

```
POST https://app.getquipt.com/oauth/token
```

**Required OAuth parameters** (passed in the `Authorization` header):

| Parameter | Description |
|---|---|
| `oauth_consumer_key` | Your application's consumer key |
| `oauth_token` | The temporary token from Step 1 |
| `oauth_verifier` | The verifier code from the Step 2 callback |
| `oauth_signature_method` | Signature method (e.g. `HMAC-SHA1`) |
| `oauth_timestamp` | Current UTC Unix timestamp |
| `oauth_nonce` | Unique random string for this request |
| `oauth_version` | `1.0` |
| `oauth_signature` | Request signature |

**Response:**

```
oauth_token=ACCESS_TOKEN&oauth_token_secret=ACCESS_SECRET
```

---

### Step 4 — Store and Use the Access Token

Persist the `oauth_token` (access token) and `oauth_token_secret` (access secret) securely. These credentials represent the user's authorization and do not expire unless revoked. Include them in the `Authorization` header of every subsequent API request, signed with your consumer secret and the access secret.

> Treat the access token and secret with the same care as a password. Do not log them, expose them to client-side code, or transmit them outside of HTTPS.

### Signing Requests

All requests must be signed per [RFC 5849 §3.4](http://tools.ietf.org/html/rfc5849#section-3.4). The signature is constructed from:

1. The HTTP method (uppercased)
2. The base URL (scheme + host + path, normalized)
3. A normalized string of all OAuth and request parameters, percent-encoded and sorted

These three components are joined with `&` to form the **signature base string**, which is then signed using HMAC-SHA1 (or your chosen method) with the key:

```
percent_encode(consumer_secret) + "&" + percent_encode(token_secret)
```

For temporary token requests (Step 1), `token_secret` is an empty string.

The resulting signature is included as `oauth_signature` in the `Authorization` header alongside the other OAuth parameters.

Most OAuth 1.0a client libraries handle signature construction automatically. Refer to your library's documentation or [RFC 5849](http://tools.ietf.org/html/rfc5849) for a complete specification.
