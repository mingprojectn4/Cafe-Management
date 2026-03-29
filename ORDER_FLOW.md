# 📋 Order Flow & API Documentation

## Order Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                     ORDER FLOW DIAGRAM                          │
└─────────────────────────────────────────────────────────────────┘

USER (Customer)                    BARISTA (Staff)
     │                                  │
     │  1. Browse Menu                  │
     │  2. Add to Cart                  │
     │  3. Place Order ─────────────┐  │
     │                              │  │
     │                              ▼  │
     │                    ┌──────────────────┐
     │                    │   API: POST      │
     │                    │   /api/orders    │
     │                    └────────┬─────────┘
     │                             │
     │                             ▼
     │                    ┌──────────────────┐
     │                    │  Order Saved to  │
     │                    │   Database       │
     │                    │  Status: pending │
     │                    └────────┬─────────┘
     │                             │
     │◄────────────────────────────┘
     │  4. Order Appears in
     │     "My Orders"
     │
     │                    ┌──────────────────┐
     │                    │  Order Appears   │
     │                    │  on Barista      │
     │                    │  Dashboard       │
     │                    └────────┬─────────┘
     │                             │
     │                    5. Barista Sees Order
     │                       Status: pending
     │                             │
     │                    6. Barista Clicks
     │                       "Start"
     │                             ▼
     │                    ┌──────────────────┐
     │                    │  Status Update   │
     │                    │  preparing       │
     │                    └────────┬─────────┘
     │                             │
     │                    7. Barista Clicks
     │                       "Complete"
     │                             ▼
     │                    ┌──────────────────┐
     │                    │  Status:         │
     │                    │  completed       │
     │                    └────────┬─────────┘
     │                             │
     │◄────────────────────────────┘
     │  8. User Sees Order
     │     Status: completed
     │
     ▼
```

---

## 📍 Where Orders Go

### 1. After User Places Order:

**Database:**
- Order is saved to `orders` table
- Order items saved to `order_items` table
- Status: `pending`

**User Sees:**
- Redirected to "My Orders" page
- Order appears in their order history
- Can see status: `pending`

**Barista Sees:**
- Order appears on Barista Dashboard (`/barista/orders`)
- Shows in "Active Orders" section
- Can see:
  - Customer name
  - Order items
  - Order type (dine-in/takeaway)
  - Special instructions

---

### 2. API Endpoints Used:

#### **Place Order (User)**
```
POST /api/orders
Headers:
  - Accept: application/json
  - X-CSRF-TOKEN: (from meta tag)

Body:
{
  "items": [
    {
      "product_id": 1,
      "quantity": 2,
      "special_instructions": "Extra hot"
    }
  ],
  "order_type": "dine-in",
  "notes": "Table 5"
}

Response:
{
  "success": true,
  "order": {
    "id": 5,
    "user_id": 3,
    "status": "pending",
    "total_price": 9.50,
    "order_type": "dine-in"
  },
  "message": "Order placed successfully"
}
```

#### **Get User's Orders**
```
GET /api/orders/my-orders
Response:
{
  "success": true,
  "orders": [
    {
      "id": 5,
      "status": "pending",
      "total_price": 9.50,
      "orderItems": [...]
    }
  ]
}
```

#### **Get All Orders (Barista)**
```
GET /api/orders
Response:
{
  "success": true,
  "orders": [
    {
      "id": 5,
      "user": { "name": "John Doe" },
      "status": "pending",
      "orderItems": [...]
    }
  ]
}
```

#### **Update Order Status (Barista)**
```
PUT /api/orders/{id}/status
Body:
{
  "status": "preparing"  // or "completed"
}

Response:
{
  "success": true,
  "order": {
    "id": 5,
    "status": "preparing"
  }
}
```

---

## 🔧 How to Test Order Flow

### Step 1: Login as User
```
URL: http://localhost:8001/login
Email: user@cafe.com
Password: password
```

### Step 2: Place an Order
1. Go to: `/user/orders`
2. Click "Add" on any products
3. Click "Cart" button
4. Select order type (dine-in/takeaway)
5. Add notes (optional)
6. Click "Place Order"

### Step 3: Check Order in "My Orders"
1. Click "My Orders" in navigation
2. You should see your order
3. Status should be: `pending`

### Step 4: Login as Barista (New Browser/Incognito)
```
URL: http://localhost:8001/login
Email: barista@cafe.com
Password: password
```

### Step 5: Barista Views Order
1. Go to: `/barista/orders`
2. You should see the order in "Active Orders"
3. Order shows:
   - Customer name
   - Items ordered
   - Order type
   - Status: pending

### Step 6: Barista Updates Status
1. Click "Start" button
2. Status changes to: `preparing`
3. Click "Complete" button
4. Status changes to: `completed`

### Step 7: User Checks Order Status
1. Login as user again
2. Go to "My Orders"
3. Status should now be: `completed`

---

## 🎯 Order Status Flow

```
┌─────────────┐
│  PENDING    │ ← After user places order
└──────┬──────┘
       │ Barista clicks "Start"
       ▼
┌─────────────┐
│ PREPARING   │ ← Barista is making the order
└──────┬──────┘
       │ Barista clicks "Complete"
       ▼
┌─────────────┐
│ COMPLETED   │ ← Order is ready for pickup
└─────────────┘

Note: Can cancel from PENDING status only
```

---

## 📊 Where to See Orders

### User Views:
| Page | What They See |
|------|---------------|
| `/user/orders` | Menu to place new orders |
| `/user/my-orders` | All their orders with status |
| `/user/my-orders/{id}` | Detailed view of one order |

### Barista Views:
| Page | What They See |
|------|---------------|
| `/barista/orders` | Active orders (pending + preparing) |
| `/barista/history` | Completed/cancelled orders |

### Admin Views:
| Page | What They See |
|------|---------------|
| `/admin/dashboard` | Order statistics |
| `/admin/orders` | All orders from all customers |

---

## 🐛 Troubleshooting

### Error: "Unauthenticated"
**Cause:** Not logged in or session expired
**Fix:** 
1. Logout and login again
2. Clear browser cache
3. Make sure cookies are enabled

### Error: Order not appearing for Barista
**Cause:** 
- Order might be completed/cancelled
- Barista page needs refresh
**Fix:**
1. Refresh the barista orders page
2. Check if order status is still pending/preparing

### Error: Can't place order
**Cause:** 
- Cart is empty
- Product unavailable
- Session expired
**Fix:**
1. Add items to cart
2. Make sure you're logged in
3. Try logout and login again

---

## ✅ Testing Checklist

- [ ] User can browse menu
- [ ] User can add items to cart
- [ ] User can place order
- [ ] Order appears in "My Orders"
- [ ] Barista can see new order
- [ ] Barista can update status to "preparing"
- [ ] Barista can update status to "completed"
- [ ] User can see updated status
- [ ] Order history is maintained

---

**Server:** http://localhost:8001
**Documentation:** Check REDESIGN_MODERN.md for UI details
