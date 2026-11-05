<div dir="rtl">

# API آی‌دی‌سیتی (AppleIDCity API)

[English](README.md) | **فارسی**

دسترسی برنامه‌نویسی به تحویل فوری اپل‌آیدی در AppleIDCity. با این API می‌توانید از داخل نرم‌افزار خودتان لیست محصولات را بگیرید، اکانت بخرید و سفارش‌هایتان را ببینید، آن هم با همان کیف پولی که در ربات تلگرام AppleIDCity دارید.

- **یک کیف پول.** حساب‌تان را در ربات تلگرام شارژ می‌کنید و خریدهای API هم از همان موجودی کسر می‌شود.
- **تحویل فوری.** بعد از خرید موفق، مشخصات کامل اکانت همان لحظه در پاسخ برگردانده می‌شود.
- **تلاش مجدد امن.** با کلید یکتایی (Idempotency Key)، ارسال دوباره‌ی یک درخواست هیچ‌وقت دو بار خرید نمی‌کند.
- **قیمت شما.** قیمتی که API برمی‌گرداند قیمت نهایی خودتان است (نرخ عمده، تخفیف محصول و تخفیف شخصی شما اعمال شده).

---

## فهرست

- [شروع سریع](#شروع-سریع)
- [احراز هویت](#احراز-هویت)
- [قواعد کلی](#قواعد-کلی)
- [اندپوینت‌ها](#اندپوینتها)
  - [اطلاعات حساب من](#get-v1me)
  - [لیست محصولات](#get-v1products)
  - [ثبت سفارش (خرید)](#post-v1orders)
  - [لیست سفارش‌ها](#get-v1orders)
  - [دریافت یک سفارش](#get-v1ordersid)
- [Idempotency (جلوگیری از خرید تکراری)](#idempotency-جلوگیری-از-خرید-تکراری)
- [خطاها](#خطاها)
- [محدودیت تعداد درخواست](#محدودیت-تعداد-درخواست)
- [نمونه کد](#نمونه-کد)
- [توصیه‌های کاربردی](#توصیههای-کاربردی)
- [سوالات متداول](#سوالات-متداول)
- [پشتیبانی](#پشتیبانی)

---

## شروع سریع

**۱. توکن بگیرید** از ربات تلگرام (بخش [احراز هویت](#احراز-هویت)).

**۲. کیف پول را شارژ کنید** از داخل ربات.

**۳. موجودی را بررسی کنید:**

<div dir="ltr">

```bash
curl -H "Authorization: Bearer $API_KEY" https://api.appleidcity.com/v1/me
```

</div>

**۴. لیست محصولات را بگیرید** و `id` محصول موردنظر را بردارید:

<div dir="ltr">

```bash
curl -H "Authorization: Bearer $API_KEY" https://api.appleidcity.com/v1/products
```

</div>

**۵. بخرید:**

<div dir="ltr">

```bash
curl -X POST https://api.appleidcity.com/v1/orders \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{"product_id":"PRODUCT_ID"}'
```

</div>

---

## احراز هویت

استفاده از API برای **نمایندگان عادی** و **نمایندگان پیشرفته** فعال است. هر درخواست باید توکن شما را همراه داشته باشد:

<div dir="ltr">

```
Authorization: Bearer aid_xxxxxxxxxxxxxxxxxxxxxxxx
```

</div>

(هدر `X-API-Key: <token>` هم به‌عنوان جایگزین پذیرفته می‌شود.)

### دریافت توکن

در ربات تلگرام AppleIDCity:

1. در منوی اصلی روی **🔌 راهنمای API** بزنید.
2. پیام `apitoken` را بفرستید تا توکن فعلی‌تان نمایش داده شود.
3. اگر هنوز توکن ندارید یا توکن جدید می‌خواهید، پیام `generatenewtoken` را بفرستید. **توکن قبلی همان لحظه از کار می‌افتد.**

> **توکن را محرمانه نگه دارید.** هر کسی که آن را داشته باشد می‌تواند از موجودی کیف پول شما خرید کند. اگر لو رفت، فوراً `generatenewtoken` را بفرستید. هرگز آن را در کد سمت کاربر (مرورگر یا اپ موبایل)، مخزن عمومی یا چت قرار ندهید.

حساب‌های مسدود‌شده نمی‌توانند از API استفاده کنند (`403 user_blocked`).

---

## قواعد کلی

| | |
|---|---|
| **آدرس پایه** | `https://api.appleidcity.com/v1` |
| **پروتکل** | فقط HTTPS |
| **قالب داده** | درخواست و پاسخ به‌صورت JSON (در `POST` هدر `Content-Type: application/json`) |
| **واحد پول** | همه‌ی قیمت‌ها و موجودی‌ها عدد صحیح و به **تومان** هستند |
| **شناسه‌ها (ID)** | رشته‌های مبهم؛ دقیقاً همان‌طور که برگردانده می‌شوند استفاده کنید |
| **تاریخ‌ها** | استاندارد ISO 8601 و به وقت UTC (مثل `2026-11-05T20:14:03.512Z`) |

پاسخ خطا همیشه این ساختار را دارد:

<div dir="ltr">

```json
{
  "error": {
    "code": "insufficient_balance",
    "message": "Wallet balance is not enough"
  }
}
```

</div>

در کد خودتان از `error.code` استفاده کنید (ثابت است). `error.message` برای انسان است و ممکن است تغییر کند.

---

## اندپوینت‌ها

### `GET /v1/me`

اطلاعات حساب و موجودی کیف پول شما را برمی‌گرداند.

<div dir="ltr">

```json
{
  "telegramId": 123456789,
  "username": "your_username",
  "role": "agent",
  "balance": 1500000,
  "discount_percent": 5
}
```

</div>

| فیلد | توضیح |
|---|---|
| `role` | `agent` (نماینده عادی) یا `senior_agent` (نماینده پیشرفته) |
| `balance` | موجودی فعلی کیف پول (تومان) |
| `discount_percent` | تخفیف شخصی شما که از قبل در قیمت محصولات اعمال شده است |

---

### `GET /v1/products`

محصولاتی را که همین الان می‌توانید بخرید برمی‌گرداند (محصولات فعالِ تحویل فوری).

<div dir="ltr">

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

</div>

| فیلد | توضیح |
|---|---|
| `id` | هنگام خرید به‌عنوان `product_id` ارسال می‌شود |
| `guarantee` | مدت گارانتی به روز |
| `type` | `non-personal-icloud` یا `non-personal-appstore` |
| `price` | **قیمت نهایی شما** به تومان (نرخ عمده، تخفیف محصول و تخفیف شما اعمال شده) |
| `stock` / `in_stock` | تعداد اکانت‌های موجود |
| `category` | دسته‌بندی محصول |

محصولات به ترتیب دسته‌بندی مرتب شده‌اند. محصولات ناموجود هم در لیست می‌آیند، با `stock: 0` و `in_stock: false`.

---

### `POST /v1/orders`

**یک** اکانت از محصول مشخص‌شده می‌خرد و مشخصاتش را برمی‌گرداند.

**هدرها**

| هدر | الزامی | توضیح |
|---|---|---|
| `Authorization` | بله | `Bearer <token>` |
| `Content-Type` | بله | `application/json` |
| `Idempotency-Key` | پیشنهاد می‌شود | یک مقدار یکتا برای هر خرید، حداکثر ۱۰۰ کاراکتر. بخش [Idempotency](#idempotency-جلوگیری-از-خرید-تکراری) را ببینید |

**بدنه**

<div dir="ltr">

```json
{ "product_id": "6a88cad19a0a76ac1989b951" }
```

</div>

**پاسخ `201 Created`**

<div dir="ltr">

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

</div>

| فیلد | توضیح |
|---|---|
| `order.status` | برای خرید موفق `delivered` است |
| `balance` | موجودی کیف پول شما **بعد از** خرید |
| `account` | اپل‌آیدی خریداری‌شده: ایمیل، رمز عبور، تاریخ تولد و سوال‌های امنیتی |
| `category.guideline` | راهنمای استفاده‌ی دسته‌بندی (ممکن است خالی باشد) |

اگر همان `Idempotency-Key` دوباره ارسال شود، پاسخ `200 OK` با **همان** اکانت و `"replayed": true` برمی‌گردد (توضیح در ادامه).

**خطاهای ممکن:** `400 invalid_product_id`، `402 insufficient_balance`، `404 product_not_found`، `409 out_of_stock`، `409 request_in_progress`. بخش [خطاها](#خطاها) را ببینید.

> اگر خرید بعد از کسر مبلغ با مشکل مواجه شود (مثلاً خطای تحویل)، مبلغ **به‌طور خودکار برگردانده می‌شود** و API خطا برمی‌گرداند.

---

### `GET /v1/orders`

سفارش‌های تحویل‌شده‌ی شما را از جدید به قدیم لیست می‌کند (شامل سفارش‌هایی که از طریق ربات تلگرام ثبت کرده‌اید).

**پارامترهای کوئری**

| پارامتر | پیش‌فرض | توضیح |
|---|---|---|
| `page` | `1` | شماره‌ی صفحه |
| `limit` | `20` | تعداد در هر صفحه (۱ تا ۱۰۰) |

<div dir="ltr">

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

</div>

این لیست مشخصات ورود اکانت را شامل نمی‌شود؛ برای آن از [`GET /v1/orders/{id}`](#get-v1ordersid) استفاده کنید.

---

### `GET /v1/orders/{id}`

مشخصات کامل یکی از سفارش‌های شما را، همراه با مشخصات اکانت، برمی‌گرداند. ساختار پاسخ مثل پاسخ `POST /v1/orders` است (بدون `replayed`) و `balance` موجودی **فعلی** شماست.

اگر سفارش وجود نداشته باشد یا متعلق به شخص دیگری باشد، خطای `404 order_not_found` برمی‌گردد.

---

## Idempotency (جلوگیری از خرید تکراری)

شبکه ممکن است قطع شود. اگر درخواست خرید timeout شود، نمی‌دانید انجام شده یا نه. ارسال هدر `Idempotency-Key` تلاش مجدد را امن می‌کند.

- برای **هر خرید جدید یک کلید یکتا** بسازید (مثلاً UUID نسخه‌ی ۴).
- اگر لازم شد **همان خرید را دوباره بفرستید** (timeout، قطع اتصال، خطای `5xx`)، **همان کلید** را دوباره بفرستید. اگر خرید اول موفق بوده باشد همان اکانت را می‌گیرید (`200` و `"replayed": true`) و **دو بار پول کسر نمی‌شود**.
- یک کلید را برای خرید دیگری **دوباره استفاده نکنید**؛ به‌جای اکانت جدید، سفارش قبلی را می‌گیرید.
- کلیدها برای هر حساب جداگانه و به مدت **۲۴ ساعت** نگه داشته می‌شوند.
- اگر درخواستی با خطا تمام شود، کلید آزاد می‌شود و می‌توان دوباره از آن استفاده کرد.
- اگر خطای `409 request_in_progress` گرفتید، درخواست اول هنوز در حال انجام است. کمی صبر کنید و با همان کلید دوباره بفرستید.

ارسال این هدر اختیاری است، ولی حتماً توصیه می‌شود. بدون آن، هر درخواست یک خرید جدید حساب می‌شود.

---

## خطاها

| HTTP | `error.code` | معنی |
|---|---|---|
| 400 | `invalid_json` | بدنه‌ی درخواست JSON معتبر نیست |
| 400 | `invalid_product_id` | `product_id` ارسال نشده یا نامعتبر است |
| 400 | `invalid_idempotency_key` | `Idempotency-Key` بیشتر از ۱۰۰ کاراکتر است |
| 401 | `missing_api_key` | توکنی ارسال نشده است |
| 401 | `invalid_api_key` | توکن اشتباه، باطل‌شده یا جایگزین‌شده با توکن جدید است |
| 402 | `insufficient_balance` | موجودی کیف پول از قیمت کمتر است |
| 403 | `user_blocked` | حساب شما مسدود است |
| 403 | `role_not_allowed` | API فقط برای نمایندگان فعال است |
| 404 | `product_not_found` | محصول وجود ندارد یا از طریق API در دسترس نیست |
| 404 | `order_not_found` | سفارش وجود ندارد یا متعلق به شما نیست |
| 404 | `not_found` | مسیر ناشناخته |
| 409 | `out_of_stock` | اکانتی برای این محصول باقی نمانده است |
| 409 | `request_in_progress` | درخواست دیگری با همین `Idempotency-Key` هنوز در حال انجام است |
| 429 | `rate_limited` | تعداد درخواست‌ها زیاد است، سرعت را کم کنید |
| 500 | `pricing_error` | قیمت قابل محاسبه نبود، مبلغی کسر نشد |
| 500 | `order_failed` | سفارش ثبت نشد، مبلغی کسر نشد |
| 500 | `delivery_failed` | تحویل ناموفق بود، مبلغ به کیف پول برگشت |
| 500 | `internal_error` | خطای غیرمنتظره‌ی سرور |

**راهنمای تلاش مجدد:** خطاهای `5xx`، `409 request_in_progress` و خطاهای شبکه را (با همان `Idempotency-Key` و کمی تأخیر) دوباره تلاش کنید. خطاهای `4xx` را کورکورانه تکرار نکنید؛ درخواست را اصلاح کنید.

---

## محدودیت تعداد درخواست

هر توکن حداکثر **۶۰ درخواست در دقیقه** می‌تواند بفرستد. بیشتر از آن خطای `429 rate_limited` برمی‌گردد. هدرهای استاندارد `RateLimit` در پاسخ‌ها وجود دارند. به‌جای گرفتن لیست محصولات قبل از هر خرید، آن را چند ثانیه کش کنید.

---

## نمونه کد

ابتدا توکن را در متغیر محیطی بگذارید (هرگز داخل کد ننویسید):

<div dir="ltr">

```bash
export APPLEIDCITY_API_KEY="aid_xxxxxxxxxxxxxxxxxxxxxxxx"
```

</div>

### cURL

<div dir="ltr">

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

</div>

### JavaScript (Node.js 18 به بالا)

<div dir="ltr">

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

</div>

### Python

<div dir="ltr">

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

</div>

### PHP

<div dir="ltr">

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

</div>

---

## توصیه‌های کاربردی

- **همیشه `Idempotency-Key` بفرستید** در `POST /v1/orders`، و فقط وقتی همان خرید را دوباره می‌فرستید از همان کلید استفاده کنید.
- **اکانت تحویل‌شده را فوراً در سیستم خودتان ذخیره کنید.** می‌توانید با `GET /v1/orders/{id}` دوباره بگیرید، ولی تنها نسخه‌ی شما نباشد.
- **قبل از خرید `stock` را ببینید**، ولی خطای `409 out_of_stock` را هم مدیریت کنید: موجودی ممکن است بین دو درخواست شما تمام شود.
- **خطای `402 insufficient_balance` را مدیریت کنید**: به خودتان هشدار بدهید یا در ربات تلگرام شارژ کنید.
- **قیمت را داده‌ی زنده بدانید.** به تخفیف‌های شما بستگی دارد و ممکن است تغییر کند. از `price` آخرین پاسخ `/v1/products` استفاده کنید و بعد از خرید با `order.price` مقایسه کنید.
- **توکن را در متغیر محیطی یا مدیریت رمز (secret manager) نگه دارید**، نه در کد.
- **توکن را بچرخانید** (`generatenewtoken`) اگر احتمال لو رفتن می‌دهید، یا وقتی یکی از اعضای تیم جدا می‌شود.
- **برای درخواست‌ها timeout بگذارید** (حدود ۳۰ ثانیه).

---

## سوالات متداول

**چطور موجودی API را شارژ کنم؟**
کیف پول‌تان را در ربات تلگرام شارژ کنید. API و ربات یک کیف پول مشترک دارند.

**می‌توانم چند اکانت را در یک درخواست بخرم؟**
فعلاً نه. هر `POST /v1/orders` یک اکانت می‌خرد. برای هر اکانت یک درخواست جدا با `Idempotency-Key` مخصوص خودش بفرستید.

**می‌توانم اپل‌آیدی شخصی را از طریق API بخرم؟**
در حال حاضر نه. API فقط محصولات تحویل فوری را پشتیبانی می‌کند (`non-personal-icloud` و `non-personal-appstore`).

**توکنم را گم کرده‌ام. می‌توانم دوباره ببینمش؟**
پیام `apitoken` را به ربات بفرستید. اگر قابل نمایش نبود، `generatenewtoken` را بفرستید تا توکن جدید بسازد (توکن قبلی باطل می‌شود).

**درخواستم timeout شد. آیا پول کسر شد؟**
با **همان** `Idempotency-Key` دوباره بفرستید. اگر درخواست اول موفق بوده، همان اکانت را می‌گیرید و دوباره پول کسر نمی‌شود. با `GET /v1/orders` هم می‌توانید بررسی کنید.

**چرا `price` با قیمتی که قبلاً دیدم فرق دارد؟**
قیمت‌ها شامل تخفیف‌های شما هستند و با تغییر محصولات یا تخفیف شما ممکن است عوض شوند. همیشه از آخرین مقدار استفاده کنید.

**محیط آزمایشی (sandbox) وجود دارد؟**
خیر. API با کیف پول واقعی شما کار می‌کند.

---

## پشتیبانی

تلگرام: [@AppleIdCity_Supp](https://t.me/AppleIdCity_Supp) · کانال: [@appleidcity](https://t.me/appleidcity)

هنگام تماس با پشتیبانی، **شماره‌ی سفارش** و زمان تقریبی درخواست را بفرستید. هرگز توکن خود را ارسال نکنید.

</div>
