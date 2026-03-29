# 🔧 Order Placement Fix

## What Was Fixed:

### 1. Added `statefulApi()` Middleware
**File:** `bootstrap/app.php`

This enables session authentication for API routes, so web frontend can use API endpoints.

```php
$middleware->statefulApi();
```

### 2. Added SANCTUM_STATEFUL_DOMAINS
**File:** `.env`

```
SANCTUM_STATEFUL_DOMAINS=localhost,localhost:8001,127.0.0.1,127.0.0.1:8001
```

### 3. Updated API Routes
**File:** `routes/api.php`

Order routes now use `auth` middleware (session auth) instead of `auth:sanctum`.

---

## ✅ How to Test:

### Step 1: Clear Browser Data
**IMPORTANT:** Clear cookies/cache or use Incognito mode!

### Step 2: Login as User
```
URL: http://localhost:8001/login
Email: user@cafe.com
Password: password
```

### Step 3: Place an Order
1. Click "Order" in navigation
2. Click "Add" on any product (e.g., Latte $4.75)
3. Cart should show: "Cart (1)"
4. Click "Cart" button
5. You should see the item in cart
6. Select order type: "Dine-in"
7. Click "Place Order"

### Step 4: Check Result
**Success:** You should see:
- Order confirmation modal
- Order ID number
- "View My Orders" button

**If you click "View My Orders":**
- You should see your order listed
- Status: pending

---

## 🐛 Still Getting "Unauthenticated"?

### Solution 1: Clear Browser Cookies
1. Press F12
2. Go to Application tab
3. Click "Clear storage"
4. Close and reopen browser
5. Try again

### Solution 2: Use Incognito/Private Mode
1. Ctrl+Shift+N (Chrome) or Ctrl+Shift+P (Firefox)
2. Go to http://localhost:8001/login
3. Login as user@cafe.com / password
4. Try placing order

### Solution 3: Check Session in Database
```bash
cd "/home/meng/Web Final (Cafe/cafe-app"
php artisan tinker
```

Then run:
```php
DB::table('sessions')->count();
```

Should return a number > 0 if you're logged in.

### Solution 4: Verify Auth Status
Open browser console (F12) and run:
```javascript
fetch('/api/user', {
    credentials: 'include'
})
.then(r => r.json())
.then(console.log);
```

If you see user data, you're authenticated.
If you see "Unauthenticated", session is not working.

---

## 🔍 Debug Checklist:

- [ ] Server running on port 8001
- [ ] Logged in as user@cafe.com
- [ ] Cart has items
- [ ] Selected order type
- [ ] Browser cookies enabled
- [ ] No console errors (F12)
- [ ] Used incognito mode (recommended)

---

## 📋 Expected Flow:

```
1. Login (user@cafe.com)
   ↓
2. Browse Menu
   ↓
3. Add to Cart
   ↓
4. Open Cart Panel
   ↓
5. Select Dine-in/Takeaway
   ↓
6. Click "Place Order"
   ↓
7. API Call: POST /api/orders
   ↓
8. Success! → Order Confirmation
   ↓
9. Redirect to "My Orders"
   ↓
10. See order with status: pending
```

---

## 🎯 Quick Test Command:

Open browser console and run:
```javascript
// Check if logged in
fetch('/api/user', { credentials: 'include' })
  .then(r => r.json())
  .then(d => console.log('Auth status:', d));

// Get products
fetch('/api/products')
  .then(r => r.json())
  .then(d => console.log('Products:', d));
```

---

## Server: http://localhost:8001
## Login: user@cafe.com / password

Try placing an order now! 🌸
