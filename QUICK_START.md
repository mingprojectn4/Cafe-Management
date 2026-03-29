# Quick Start Guide

## ✅ System Status
- Server: Running on http://localhost:8000
- Database: Seeded with sample data
- Routes: All registered correctly

## 🔐 Login Credentials

| Role | Email | Password | Redirects To |
|------|-------|----------|--------------|
| Admin | admin@cafe.com | password | /admin/dashboard |
| Barista | barista@cafe.com | password | /barista/orders |
| User | user@cafe.com | password | /user/orders |

## 📝 How to Test

### Step 1: Open Browser
Go to: http://localhost:8000

### Step 2: Clear Browser Data (IMPORTANT!)
Press F12 → Application tab → Clear storage → Clear site data
OR: Ctrl+Shift+Delete → Clear cookies and cache

### Step 3: Login
1. Enter email: `admin@cafe.com`
2. Enter password: `password`
3. Click "Sign In"

### Step 4: Verify Redirect
- Admin should see: Dashboard at `/admin/dashboard`
- Barista should see: Orders at `/barista/orders`
- User should see: Menu at `/user/orders`

## 🐛 If You Get 404 After Login

### Option 1: Clear Laravel Cache
```bash
cd "/home/meng/Web Final (Cafe/cafe-app"
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
```

### Option 2: Restart Server
```bash
pkill -f "php artisan serve"
php artisan serve --host=0.0.0.0 --port=8000
```

### Option 3: Use Incognito/Private Window
Open browser incognito mode and go to http://localhost:8000

### Option 4: Test with Direct URLs
After logging in, manually navigate to:
- Admin: http://localhost:8000/admin/dashboard
- Barista: http://localhost:8000/barista/orders  
- User: http://localhost:8000/user/orders

## ✨ Features to Test

### As Admin:
1. View dashboard statistics
2. Click "Products" → Add/Edit/Delete products
3. Click "Staff" → Manage staff accounts
4. Click "Orders" → View all orders
5. Click "Customers" → View customer list

### As Barista:
1. View active orders
2. Click "Start" to change status to "preparing"
3. Click "Complete" to finish order
4. View order history

### As User:
1. Browse menu by category
2. Click "Add" to add items to cart
3. Click "Cart" to view cart
4. Select order type (Dine-in/Takeaway)
5. Click "Place Order"
6. View order in "My Orders"

## 🔍 Debug Info

Test route: http://localhost:8000/test

This shows if routes are working correctly.

## 📞 Still Having Issues?

Check TROUBLESHOOTING.md for detailed solutions.
