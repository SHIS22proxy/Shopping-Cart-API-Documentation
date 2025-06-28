# Shopping Cart API Documentation

## 1. Introduction

### Overview  
The **Shopping Cart API** is a RESTful service that allows client applications to interact with a virtual shopping cart system. It enables users to add, update, remove items, and view cart contents before placing an order. This API is essential in e-commerce environments to manage the customer's selections and buying process.

### Why It Matters  
A shopping cart is the central component in online retail platforms. Without a reliable, secure, and flexible Shopping Cart API, customers can't effectively manage their purchases, leading to poor user experience and lost sales.

### Who This Guide is For  
This documentation is written for:
- Frontend and backend developers working on e-commerce applications
- QA engineers testing cart-related features
- DevOps teams deploying API services
- Technical leads integrating cart functionality with order and payment systems

---

## 2. Key Terminology

- **Cart**: A temporary container storing items a user intends to purchase.
- **Cart ID**: A unique identifier for a specific cart (often user-bound or session-based).
- **Item ID**: Unique identifier of a product added to the cart.
- **Quantity**: Number of units of a product in the cart.
- **Subtotal**: Sum of all item prices in the cart, excluding taxes or discounts.
- **Session Token**: Auth token representing the user session.
- **Checkout**: The process of finalizing a cart and converting it into an order.

---

## 3. Technical Overview

### 3.1 System Architecture

The Shopping Cart API typically sits between the frontend interface and the backend order/payment system. It uses a session or user-based identifier to store cart contents, either in memory (Redis), database, or cache store.

```mermaid
graph TD
    A[Client (Web/App)] -->|API Request| B[Shopping Cart API]
    B --> C[In-Memory Store (Redis)]
    B --> D[Product Catalog API]
    B --> E[Order Management API]
    B --> F[Authentication Service]
```
### 3.2 Core Technologies
* Language/Framework: Node.js (Express), Python (Django REST), Java (Spring Boot)

* Storage: Redis (for fast session-based cart), or SQL/NoSQL DB

* Authentication: JWT, OAuth 2.0

* Formats: JSON for request/response

## 4. Step-by-Step Guide or Workflow
### 4.1 Authentication
Ensure each request includes a valid token or session header:
```
pgsql

Authorization: Bearer <your-access-token>

```
## 4.2 API Endpoints
### 4.2.1. Create/Get User Cart
GET /api/cart

Response:
```
json

{
  "cart_id": "abc123",
  "items": [],
  "subtotal": 0.00
}
```

### 4.2.2. Add Item to Cart
POST /api/cart/items

Request Body:

```
json
{
  "product_id": "p001",
  "quantity": 2
}
```
Response:
```
json
{
  "message": "Item added to cart.",
  "cart": {
    "items": [
      {
        "product_id": "p001",
        "quantity": 2,
        "price": 15.00
      }
    ],
    "subtotal": 30.00
  }
}
```
### 4.2.3. Update Item Quantity
PUT /api/cart/items/p001

Request Body:
```
json
{
  "quantity": 3
}
```
### 4.2.4. Remove Item from Cart
DELETE /api/cart/items/p001

Response:
```
json
{
  "message": "Item removed",
  "cart": {
    "items": [],
    "subtotal": 0.00
  }
}
```
### 4.2.5. Empty the Cart
DELETE /api/cart

### 4.2.6. Proceed to Checkout
POST /api/cart/checkout

Response:
```
json
{
  "order_id": "ORD56789",
  "status": "checkout_started"
}
```

## 5. Best Practices
* Expire Inactive Carts: Set TTL in Redis to auto-delete old carts.

* Validate Stock Availability: Sync with Product Catalog API before checkout.

* Avoid Overwrites: Use PATCH for partial updates.

* Use Rate Limiting: Protect cart endpoints from abuse.

* Encrypt Sensitive Data: e.g., pricing or promo codes.

* Decouple Checkout: Handle payment/order creation in a separate service.

## 6. Common Issues & Troubleshooting
|Issue	| Description | Resolution |
|-------|-------------|------------|
|404 | Not Found |	Cart or item doesn't exist	Ensure the cart/session is valid |
|409 | Conflict	| Item already exists	Use PUT to update quantity instead of POST |
|401 | Unauthorized	| Invalid or expired token	Re-authenticate the user| 
|422 | Unprocessable |	Invalid product ID or quantity	Validate against Product API |  

## 7. References
Redis for Session Storage

REST API Best Practices

JWT Authentication Guide

Stripe API Checkout Flow

## 8. Appendix
Sample Cart Object

```
json
{
  "cart_id": "abc123",
  "user_id": "u5678",
  "items": [
    {
      "product_id": "p001",
      "name": "USB-C Charger",
      "quantity": 2,
      "unit_price": 25.00
    }
  ],
  "subtotal": 50.00,
  "created_at": "2025-06-27T15:30:00Z"
}
```
Mermaid Sequence: Add Item to Cart

```
sequenceDiagram
    Client ->> Cart API: POST /cart/items
    Cart API ->> Product API: GET /products/{id}
    Product API -->> Cart API: Product info
    Cart API ->> Storage: Add item to cart
    Cart API -->> Client: Cart updated
```

Shell Example: Add to Cart
bash
```
curl -X POST https://api.example.com/api/cart/items \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{"product_id": "p001", "quantity": 2}'
```