# AppleIDCity API

**English** | [فارسی](README.fa.md)

Programmatic access to AppleIDCity's instant Apple ID delivery. List products, buy accounts and fetch your orders from your own application, using the same wallet you already use in the AppleIDCity Telegram bot.

- **One wallet.** Top up in the Telegram bot; API purchases are deducted from the same balance.
- **Instant delivery.** A successful purchase returns the full account details in the response.
- **Safe retries.** Idempotency keys make sure a retried request never buys twice.
- **Your price.** Prices returned by the API are your final price (wholesale rate, product discount and your personal discount already applied).

---

## Table of contents

- [Quick start](#quick-start)
- [Authentication](#authentication)
- [Conventions](#conventions)
- [Endpoints](#endpoints)
  - [Get my account](#get-v1me)
  - [List products](#get-v1products)
  - [Create an order](#post-v1orders)
  - [List orders](#get-v1orders)
  - [Get an order](#get-v1ordersid)
- [Idempotency](#idempotency)
- [Errors](#errors)
- [Rate limits](#rate-limits)
- [Code examples](#code-examples)
- [Best practices](#best-practices)
- [FAQ](#faq)
- [Support](#support)

---

## Quick start

1. **Get a token** from the Telegram bot (see [Authentication](#authentication)).
2. **Top up your wallet** in the bot.
3. **Check your balance:**
   ```bash
   curl -H "Authorization: Bearer $API_KEY" https://api.appleidcity.com/v1/me
   ```
4. **List products** and copy the `id` of the one you want:
   ```bash
   curl -H "Authorization: Bearer $API_KEY" https://api.appleidcity.com/v1/products
   ```
5. **Buy it:**
   ```bash
   curl -X POST https://api.appleidcity.com/v1/orders \
     -H "Authorization: Bearer $API_KEY" \
     -H "Content-Type: application/json" \
     -H "Idempotency-Key: $(uuidgen)" \
     -d '{"product_id":"PRODUCT_ID"}'
   ```

---

## Authentication

The API is available to **agents** and **senior agents**. Every request must carry your API token:

```
Authorization: Bearer aid_xxxxxxxxxxxxxxxxxxxxxxxx
```

(The `X-API-Key: <token>` header is accepted as an alternative.)

### Getting your token

In the AppleIDCity Telegram bot:

1. Tap **🔌 API Guide** (راهنمای API) in the main menu.
2. Send the message `apitoken` to receive your current token.
3. If you do not have a token yet, or you want a new one, send `generatenewtoken`. **The previous token stops working immediately.**

> **Keep your token secret.** Anyone who has it can spend your wallet balance. If it leaks, send `generatenewtoken` right away. Never put it in client-side code, public repositories or chats.

Blocked accounts cannot use the API (`403 user_blocked`).

---

## Conventions

| | |
|---|---|
| **Base URL** | `https://api.appleidcity.com/v1` |
| **Protocol** | HTTPS only |
| **Format** | JSON requests and responses (`Content-Type: application/json` on `POST`) |
| **Currency** | All prices and balances are integers in **Toman** |
| **IDs** | Opaque strings, always use them exactly as returned |
| **Dates** | ISO 8601, UTC (e.g. `2026-11-05T20:14:03.512Z`) |

Error responses always use the same shape:

```json
{
  "error": {
    "code": "insufficient_balance",
    "message": "Wallet balance is not enough"
  }
}
```

Use `error.code` in your code (it is stable). `error.message` is for humans and may change.

---

## Endpoints

### `GET /v1/me`

Returns your account and wallet balance.

```json
{
  "telegramId": 123456789,
  "username": "your_username",
  "role": "agent",
  "balance": 1500000,
  "discount_percent": 5
}
```

| Field | Description |
|---|---|
| `role` | `agent` or `senior_agent` |
| `balance` | Current wallet balance (Toman) |
| `discount_percent` | Your personal discount, already included in product prices |

---

### `GET /v1/products`

Returns the products you can buy right now (active, instant-delivery products).

```json
{
  "products": [
    {
      "id": "6a88cad19a0a76ac1989b951",
      "name": "Example product",
      "description": "Product description",
      "guarantee": 30,
      "type": "non-personal-icloud",
      "price": 250000,
      "stock": 12,
      "in_stock": true,
      "category": {
        "id": "6a88c1e29a0a76ac1989b900",
        "name": "Example category",
        "slug": "example-category"
      }
    }
  ]
}
```

| Field | Description |
|---|---|
| `id` | Use this as `product_id` when buying |
| `guarantee` | Warranty period in days |
| `type` | `non-personal-icloud` or `non-personal-appstore` |
| `price` | **Your final price** in Toman (wholesale rate, product discount and your discount applied) |
| `stock` / `in_stock` | Number of accounts currently available |
| `category` | The category the product belongs to |

Products are ordered by category. Out-of-stock products are still listed, with `stock: 0` and `in_stock: false`.

---

### `POST /v1/orders`

Buys **one** account of the given product and returns its details.

**Headers**

| Header | Required | Description |
|---|---|---|
| `Authorization` | yes | `Bearer <token>` |
| `Content-Type` | yes | `application/json` |
| `Idempotency-Key` | recommended | A unique value per purchase, max 100 characters. See [Idempotency](#idempotency) |

**Body**

```json
{ "product_id": "6a88cad19a0a76ac1989b951" }
```

**Response `201 Created`**

```json
{
  "order": {
    "id": "6a95e57ae1e74222f82f5e5d",
    "status": "delivered",
    "price": 250000,
    "paidAt": "2026-11-05T20:14:03.512Z",
    "deliveredAt": "2026-11-05T20:14:03.640Z",
    "createdAt": "2026-11-05T20:14:03.498Z"
  },
  "balance": 1250000,
  "product": {
    "id": "6a88cad19a0a76ac1989b951",
    "name": "Example product",
    "guarantee": 30,
    "type": "non-personal-icloud"
  },
  "category": {
    "id": "6a88c1e29a0a76ac1989b900",
    "name": "Example category",
    "slug": "example-category",
    "description": "Category description",
    "guideline": "Usage guideline for this category"
  },
  "account": {
    "email": "example@icloud.com",
    "password": "********",
    "birthDate": "2000-01-01",
    "securityQuestions": [
      { "question": "Question 1", "answer": "Answer 1" },
      { "question": "Question 2", "answer": "Answer 2" },
      { "question": "Question 3", "answer": "Answer 3" }
    ]
  }
}
```

| Field | Description |
|---|---|
| `order.status` | `delivered` for a successful purchase |
| `balance` | Your wallet balance **after** the purchase |
| `account` | The purchased Apple ID: email, password, birth date and security questions |
| `category.guideline` | The usage guideline of the category (may be empty) |

If the same `Idempotency-Key` is sent again, you get `200 OK` with the **same** account and `"replayed": true` (see below).

**Possible errors:** `400 invalid_product_id`, `402 insufficient_balance`, `404 product_not_found`, `409 out_of_stock`, `409 request_in_progress`. See [Errors](#errors).

> If a purchase fails after your wallet was charged (for example a delivery problem), the amount is **automatically refunded** and the API returns an error.

---

### `GET /v1/orders`

Lists your delivered orders, newest first (including orders you placed through the Telegram bot).

**Query parameters**

| Parameter | Default | Description |
|---|---|---|
| `page` | `1` | Page number |
| `limit` | `20` | Items per page (1 to 100) |

```json
{
  "page": 1,
  "limit": 20,
  "total": 134,
  "orders": [
    {
      "id": "6a95e57ae1e74222f82f5e5d",
      "product": { "id": "6a88cad19a0a76ac1989b951", "name": "Example product" },
      "price": 250000,
      "status": "delivered",
      "createdAt": "2026-11-05T20:14:03.498Z",
      "deliveredAt": "2026-11-05T20:14:03.640Z"
    }
  ]
}
```

The list does not include account credentials. Use [`GET /v1/orders/{id}`](#get-v1ordersid) for those.

---

### `GET /v1/orders/{id}`

Returns the full details of one of your orders, including the account credentials. The response has the same shape as the `POST /v1/orders` response (without `replayed`), and `balance` is your **current** balance.

Returns `404 order_not_found` if the order does not exist or belongs to someone else.

---

## Idempotency

Networks fail. If a purchase request times out you do not know whether it went through. Sending an `Idempotency-Key` header makes retrying safe.

- Generate a **new unique key (for example a UUID v4) for every new purchase.**
- If you need to **retry the same purchase** (timeout, connection reset, `5xx`), send the **same key again**. If the first attempt succeeded you receive the same account (`200`, `"replayed": true`) and you are **not** charged twice.
- Do **not** reuse a key for a different purchase: you would get the old order back instead of a new account.
- Keys are scoped to your account and kept for **24 hours**.
- If a request fails with an error, the key is released and the same key can be used again.
- If you get `409 request_in_progress`, the original request is still running. Wait a moment and retry with the same key.

The header is optional, but we strongly recommend always sending it. Without it, every request is treated as a new purchase.

---

## Errors

| HTTP | `error.code` | Meaning |
|---|---|---|
| 400 | `invalid_json` | The request body is not valid JSON |
| 400 | `invalid_product_id` | `product_id` is missing or malformed |
| 400 | `invalid_idempotency_key` | `Idempotency-Key` is longer than 100 characters |
| 401 | `missing_api_key` | No token was sent |
| 401 | `invalid_api_key` | The token is wrong, revoked or replaced by a newer one |
| 402 | `insufficient_balance` | Your wallet balance is lower than the price |
| 403 | `user_blocked` | Your account is blocked |
| 403 | `role_not_allowed` | The API is only available to agents |
| 404 | `product_not_found` | The product does not exist or is not available through the API |
| 404 | `order_not_found` | The order does not exist or is not yours |
| 404 | `not_found` | Unknown route |
| 409 | `out_of_stock` | No accounts are left for this product |
| 409 | `request_in_progress` | Another request with the same `Idempotency-Key` is still running |
| 429 | `rate_limited` | Too many requests, slow down |
| 500 | `pricing_error` | The price could not be calculated, you were not charged |
| 500 | `order_failed` | The order could not be created, you were not charged |
| 500 | `delivery_failed` | Delivery failed, your balance was refunded |
| 500 | `internal_error` | Unexpected server error |

**Retry guidance:** retry `5xx`, `409 request_in_progress` and network errors (with the same `Idempotency-Key` and a short delay). Do not blindly retry `4xx` errors: fix the request instead.

---

## Rate limits

Each token can make up to **60 requests per minute**. Beyond that the API returns `429 rate_limited`. Standard `RateLimit` response headers are included. Cache the product list for a few seconds instead of fetching it before every purchase.

---

## Code examples

Set your token as an environment variable first (never hard-code it):

```bash
export APPLEIDCITY_API_KEY="aid_xxxxxxxxxxxxxxxxxxxxxxxx"
```

### cURL

```bash
BASE=https://api.appleidcity.com/v1
AUTH="Authorization: Bearer $APPLEIDCITY_API_KEY"

# balance
curl -s -H "$AUTH" $BASE/me

# products
curl -s -H "$AUTH" $BASE/products

# buy (a fresh key for every new purchase)
curl -s -X POST $BASE/orders \
  -H "$AUTH" -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{"product_id":"PRODUCT_ID"}'

# fetch an order again
curl -s -H "$AUTH" $BASE/orders/ORDER_ID
```

### JavaScript (Node.js 18+)

```js
import { randomUUID } from 'node:crypto'

const BASE = 'https://api.appleidcity.com/v1'
const API_KEY = process.env.APPLEIDCITY_API_KEY
const sleep = (ms) => new Promise((r) => setTimeout(r, ms))

export async function buyAccount(productId) {
  const idempotencyKey = randomUUID() // one key per purchase, reused on retries

  for (let attempt = 1; attempt <= 3; attempt++) {
    let res
    try {
      res = await fetch(`${BASE}/orders`, {
        method: 'POST',
        headers: {
          Authorization: `Bearer ${API_KEY}`,
          'Content-Type': 'application/json',
          'Idempotency-Key': idempotencyKey,
        },
        body: JSON.stringify({ product_id: productId }),
        signal: AbortSignal.timeout(30_000),
      })
    } catch {
      await sleep(1000 * attempt) // network error or timeout: safe to retry with the same key
      continue
    }

    const data = await res.json()
    if (res.ok) return data

    const code = data.error?.code
    if (res.status >= 500 || code === 'request_in_progress') {
      await sleep(1000 * attempt)
      continue
    }
    throw new Error(`${code}: ${data.error?.message}`)
  }

  throw new Error('Purchase not confirmed. Check GET /orders before retrying with a new key.')
}
```

### Python

```python
import os
import time
import uuid

import requests

BASE = "https://api.appleidcity.com/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APPLEIDCITY_API_KEY']}"}


def buy_account(product_id, retries=3):
    key = str(uuid.uuid4())  # one key per purchase, reused on retries

    for attempt in range(1, retries + 1):
        try:
            r = requests.post(
                f"{BASE}/orders",
                json={"product_id": product_id},
                headers={**HEADERS, "Idempotency-Key": key},
                timeout=30,
            )
        except requests.RequestException:
            time.sleep(attempt)  # network error or timeout: safe to retry with the same key
            continue

        data = r.json()
        if r.ok:
            return data

        code = data.get("error", {}).get("code")
        if r.status_code >= 500 or code == "request_in_progress":
            time.sleep(attempt)
            continue

        raise RuntimeError(f"{code}: {data['error']['message']}")

    raise RuntimeError("Purchase not confirmed. Check GET /orders before retrying with a new key.")
```

### PHP

```php
<?php
$base   = 'https://api.appleidcity.com/v1';
$apiKey = getenv('APPLEIDCITY_API_KEY');

function api(string $method, string $path, ?array $body = null, array $extraHeaders = []): array
{
    global $base, $apiKey;

    $ch = curl_init($base . $path);
    curl_setopt_array($ch, [
        CURLOPT_CUSTOMREQUEST  => $method,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_TIMEOUT        => 30,
        CURLOPT_HTTPHEADER     => array_merge([
            "Authorization: Bearer $apiKey",
            'Content-Type: application/json',
        ], $extraHeaders),
    ]);
    if ($body !== null) {
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($body));
    }

    $raw    = curl_exec($ch);
    $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    return [$status, json_decode($raw, true)];
}

// one key per purchase; reuse the same key if you retry
$key = bin2hex(random_bytes(16));

[$status, $data] = api('POST', '/orders', ['product_id' => 'PRODUCT_ID'], ["Idempotency-Key: $key"]);

if ($status === 201 || $status === 200) {
    echo $data['account']['email'];
} else {
    echo $data['error']['code'];
}
```

---

## Best practices

- **Always send an `Idempotency-Key`** on `POST /v1/orders`, and reuse it only when retrying the same purchase.
- **Save the delivered account immediately** in your own system. You can fetch it again with `GET /v1/orders/{id}`, but do not depend on that as your only copy.
- **Check `stock` before buying**, but still handle `409 out_of_stock`: stock can run out between your two requests.
- **Handle `402 insufficient_balance`** by alerting yourself or topping up in the Telegram bot.
- **Treat prices as live data.** They depend on your discount and can change. Use the `price` from the latest `/v1/products` response and compare with the `order.price` after buying.
- **Store the token in an environment variable or secret manager**, never in source code.
- **Rotate the token** (`generatenewtoken`) if you suspect a leak, or when a team member leaves.
- **Use timeouts** (about 30 seconds) on your HTTP client.

---

## FAQ

**How do I top up the API balance?**
Top up your wallet in the Telegram bot. The API and the bot share one wallet.

**Can I buy several accounts in one request?**
Not yet. Each `POST /v1/orders` buys one account. Send one request per account, each with its own `Idempotency-Key`.

**Can I buy personal Apple IDs through the API?**
Not at the moment. The API supports instant-delivery products only (`non-personal-icloud` and `non-personal-appstore`).

**I lost my token. Can I see it again?**
Send `apitoken` to the bot. If it cannot be displayed, send `generatenewtoken` to create a new one (the old one is revoked).

**My request timed out. Was I charged?**
Retry with the **same** `Idempotency-Key`. If the first request succeeded you get the same account back and are not charged again. You can also check `GET /v1/orders`.

**Why does `price` differ from the price I saw before?**
Prices include your discounts and may change when products or your discount change. Always use the latest value.

**Is there a sandbox or test mode?**
No. The API works with your real wallet.

---

## Support

Telegram: [@AppleIdCity_Supp](https://t.me/AppleIdCity_Supp) · Channel: [@appleidcity](https://t.me/appleidcity)

When you contact support, include the **order ID** and the approximate time of the request. Never send your token.
