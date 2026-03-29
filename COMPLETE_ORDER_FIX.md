# 🔧 Complete Order System Fix

## Root Cause Analysis

### Problem:
"Unauthenticated" error when placing order because:
1. API routes use `auth` middleware
2. But API doesn't have access to web session by default
3. `statefulApi()` was added but may need additional config

## ✅ Complete Fix Applied

### 1. Database Status ✓
```
Users: 5 (admin, barista, 3 customers)
Products: 17 (coffee, tea, pastry, snacks)
Orders: 4 (sample orders)
Sessions: Working
```

### 2. Configuration Fixed ✓

**File: `bootstrap/app.php`**
```php
$middleware->statefulApi();  // Enables session for API
```

**File: `.env`**
```
SANCTUM_STATEFUL_DOMAINS=localhost,localhost:8001,127.0.0.1
SESSION_DRIVER=database
```

### 3. Routes Configuration ✓

**File: `routes/api.php`**
```php
// Orders use 'auth' middleware (web session)
Route::middleware(['auth'])->group(function () {
    Route::post('/orders', [OrderController::class, 'store']);
    // ... other order routes
});
```

---

## 🎯 END-TO-END TEST GUIDE

### Test 1: User Places Order

**Step 1: Open Incognito Window**
```
Ctrl + Shift + N (Chrome)
```

**Step 2: Login as User**
```
URL: http://localhost:8001/login
Email: user@cafe.com
Password: password
```

**Step 3: Navigate to Order Page**
- Click "Order" in navigation
- URL should be: `/user/orders`

**Step 4: Add Items to Cart**
```javascript
// Open browser console (F12) and run:
console.log('Testing cart...');

// Check if products load
fetch('/api/products', { credentials: 'include' })
  .then(r => r.json())
  .then(d => console.log('Products loaded:', d.products.length));

// Check auth
fetch('/api/user', { credentials: 'include' })
  .then(r => r.json())
  .then(d => console.log('User:', d.user?.name));
```

**Expected Console Output:**
```
Testing cart...
Products loaded: 17
User: Jane Customer
```

**Step 5: Place Order**
1. Click "Add" on any product
2. Cart count should increase
3. Click "Cart" button
4. Select "Dine-in"
5. Click "Place Order"

**Expected Result:**
- Order confirmation modal appears
- Order ID is shown
- No "Unauthenticated" error

---

### Test 2: Barista Receives Order

**Step 1: Open NEW Incognito Window**
```
Ctrl + Shift + N (Chrome)
```

**Step 2: Login as Barista**
```
URL: http://localhost:8001/login
Email: barista@cafe.com
Password: password
```

**Step 3: Check Orders**
- Should see orders on dashboard
- Look for the order you just placed
- Should show:
  - Customer name
  - Order items
  - Status: pending

**Step 4: Update Order Status**
1. Click "Start" button
2. Status changes to: preparing
3. Click "Complete" button
4. Status changes to: completed

---

### Test 3: User Sees Status Update

**Step 1: Go back to User's browser**

**Step 2: Go to "My Orders"**
- Click "My Orders" in navigation
- Find your order
- Status should be: completed

---

## 🔍 Debug Commands

### Check if Authenticated (Browser Console)
```javascript
fetch('/api/user', {
    credentials: 'include'
})
.then(r => r.json())
.then(d => console.log('Auth:', d));
```

### Check Orders API (Browser Console)
```javascript
// Place test order
fetch('/api/orders', {
    method: 'POST',
    credentials: 'include',
    headers: {
        'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        items: [{ product_id: 1, quantity: 1 }],
        order_type: 'dine-in'
    })
})
.then(r => r.json())
.then(d => console.log('Order result:', d));
```

### Check Database (Command Line)
```bash
cd "/home/meng/Web Final (Cafe/cafe-app"
php artisan tinker
```

```php
// Check latest order
App\Models\Order::latest()->first();

// Check order with items
$order = App\Models\Order::with('orderItems')->latest()->first();
echo "Order #{$order->id} by {$order->user->name}";
echo "Items: " . $order->orderItems->count();
```

---

## 📋 Checklist

Before testing, verify:
- [ ] Server running on port 8001
- [ ] Using incognito/private mode
- [ ] Browser cookies enabled
- [ ] No console errors (F12)
- [ ] CSRF token in page source
- [ ] Session table has records

---

## 🎯 Quick Test Sequence

```
1. Open Incognito → Login as user@cafe.com
2. Go to /user/orders
3. Open Console (F12)
4. Run: fetch('/api/user', {credentials:'include'}).then(r=>r.json()).then(console.log)
5. Should see: {success: true, user: {...}}
6. If yes → Place order normally
7. If no → Session not working, clear cookies
```

---

## 🚨 Common Issues

### Issue 1: "Unauthenticated"
**Cause:** Session not passed to API
**Fix:** 
1. Make sure `statefulApi()` is in bootstrap/app.php
2. Use incognito mode
3. Clear browser cookies

### Issue 2: CSRF Token Mismatch
**Cause:** Token not sent with request
**Fix:**
1. Check meta tag exists: `<meta name="csrf-token" content="...">`
2. Check JavaScript includes it in headers

### Issue 3: Order Not Showing for Barista
**Cause:** Barista looking at wrong page
**Fix:**
1. Barista should go to `/barista/orders`
2. Refresh page if order doesn't appear
3. Check order status is still 'pending' or 'preparing'

---

## ✅ Expected Flow

```
USER (Incognito 1)
  ↓ Login: user@cafe.com
  ↓ Go to /user/orders
  ↓ Add items to cart
  ↓ Place order
  ↓ See confirmation
  ↓ Check "My Orders" - sees order (pending)
  
BARISTA (Incognito 2)
  ↓ Login: barista@cafe.com
  ↓ Go to /barista/orders
  ↓ Sees order in "Active Orders"
  ↓ Clicks "Start" → status: preparing
  ↓ Clicks "Complete" → status: completed
  
USER (Back to Incognito 1)
  ↓ Refresh "My Orders"
  ↓ Sees status: completed ✓
```

---

**Server:** http://localhost:8001
**Status:** Ready for testing
