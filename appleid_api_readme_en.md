# Apple ID City API 📱

This API allows instant retrieval, purchase, and management of Apple IDs.  
Developers can use it in **bots, websites, and applications** without the need to buy or store Apple IDs beforehand.

> Note: A [Persian version](https://github.com/omariscoming/docs-appleidapi) of this documentation is also available.

---

## 1️⃣ Overview 📝

- Instant Apple ID retrieval  
- Purchase and manage Apple IDs  
- Access user information and account balance  
- Secure with a dedicated API token  
- Suitable for both small and large projects  

---

## 2️⃣ Features ✨

- ⚡ **Instant Apple ID retrieval**: No prior storage or purchase required  
- 🔒 **High security**: Each user has a dedicated token  
- 🤖 **Bot and website friendly**: Easy and fast integration  
- 🛠️ **Easy implementation**: Just a few API requests  
- 📈 **Scalable**: Suitable for small and large projects  

---

## 3️⃣ API Token (Authentication) 🔑

All API requests require an **API Token**, obtainable from the [Apple ID City Bot](https://t.me/appleidcitybot).

### Header format
```
Authorization: YOUR_API_TOKEN
Content-Type: application/json
```

### Access rules
- Missing **Authorization header** → `401 Unauthorized`  
- Invalid token → `401 Unauthorized`  
- User type must be `n2` → otherwise `403 Forbidden`  
- Blocked user → `403 Forbidden`  

### Sample error responses
```json
{
  "success": false,
  "error": { "message": "Authorization header missing" }
}
```
```json
{
  "success": false,
  "error": { "message": "Invalid API token" }
}
```
```json
{
  "success": false,
  "error": { "message": "Access denied: insufficient permission level." }
}
```
```json
{
  "success": false,
  "error": { "message": "Internal server error" }
}
```

---

## 4️⃣ Usage 🚀

All endpoints use the following base URL:
```
https://api.appleidcity.com/appleid
```

> All requests require a valid **API Token**.

### Endpoints

| Endpoint           | Description                     | Method |
|------------------|---------------------------------|--------|
| `/buy`           | Purchase an Apple ID            | POST   |
| `/products`      | Get available products          | GET    |
| `/user`          | Get basic user information      | GET    |

---

### 4.1 Purchase Apple ID `/buy`

**Body (JSON):**
| Parameter | Type    | Description                        |
|-----------|---------|------------------------------------|
| icloud    | boolean | Whether the Apple ID is old or not |

**Sample curl request:**
```bash
curl -X POST "https://api.appleidcity.com/appleid/buy" \
-H "Authorization: Bearer YOUR_API_TOKEN" \
-H "Content-Type: application/json" \
-d '{"icloud": true}'
```

**Sample successful response:**
```json
{
  "success": true,
  "message": "Apple ID delivered successfully",
  "data": {
    "credentials": {
      "mail": "example@apple.com",
      "password": "Abc123456",
      "birthdate": "01-01-1990",
      "questions": ["Question1", "Question2", "Question3"]
    },
    "product": {
      "name": "Apple ID US",
      "price": 10,
      "finalPrice": 9,
      "category": "appleid",
      "icloud": true
    },
    "user": {
      "ready": 1,
      "discountPercentage": 10,
      "balance": 91
    }
  }
}
```

---

### 4.2 Get Products `/products`

**Sample curl request:**
```bash
curl -X GET "https://api.appleidcity.com/appleid/products" \
-H "Authorization: Bearer YOUR_API_TOKEN"
```

**Sample successful response:**
```json
{
  "success": true,
  "message": "Product List delivered successfully",
  "data": {
    "products": [
      {
        "name": "Apple ID US",
        "price": 10,
        "finalPrice": 9,
        "category": "appleid",
        "icloud": true
      },
      {
        "name": "Apple ID UK",
        "price": 12,
        "finalPrice": 10.8,
        "category": "appleid",
        "icloud": false
      }
    ]
  }
}
```

---

### 4.3 Get User Info `/user`

**Sample curl request:**
```bash
curl -X GET "https://api.appleidcity.com/appleid/user" \
-H "Authorization: Bearer YOUR_API_TOKEN"
```

**Sample successful response:**
```json
{
  "success": true,
  "message": "User information delivered successfully",
  "data": {
    "balance": 100,
    "totalBalance": 500,
    "ready": 3,
    "custom": 1
  }
}
```

**Formatted user info (bot view):**
```
🔻 User Details
🔹 Name: John Doe
🔹 Username: @johndoe
🔹 User ID: 98765
🔹 Balance: 100
🔹 Discount: %10
🔹 Apple IDs Count: 2
🔹 User Type: n2
🔹 Status: ✅ Active
```

---

## 5️⃣ Important Notes ⚠️

- All `/buy`, `/products`, and `/user` endpoints **require a valid API Token**.  
- Get your token from [Apple ID City Bot](https://t.me/appleidcitybot).  
- `finalPrice` is calculated based on user discount.  
- Error responses include insufficient balance, invalid token, or no product available.  
- All endpoints are prefixed with `/appleid`.  

---

## 6️⃣ Support

For support and questions, contact us via [Apple ID City Bot](https://t.me/appleidcitybot).

