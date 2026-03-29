# Troubleshooting Guide

## Issue: 404 After Login

If you're getting 404 error after logging in, follow these steps:

### Solution 1: Clear Browser Cache
1. Open your browser's developer tools (F12)
2. Right-click the refresh button → "Empty Cache and Hard Reload"
3. Or use: Ctrl+Shift+Delete (Windows) / Cmd+Shift+Delete (Mac)
4. Clear cookies and cached files
5. Close and reopen browser
6. Try logging in again

### Solution 2: Clear Laravel Sessions
Run these commands in the project directory:

```bash
cd "/home/meng/Web Final (Cafe/cafe-app"
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan session:flush
```

### Solution 3: Restart the Server
```bash
# Stop existing server
pkill -f "php artisan serve"

# Start new server
php artisan serve --host=0.0.0.0 --port=8000
```

### Solution 4: Check Database
Verify users exist with correct roles:

```bash
cd "/home/meng/Web Final (Cafe/cafe-app"
php artisan tinker
```

Then run:
```php
App\Models\User::all()->each(fn($u) => print($u->role . ': ' . $u->email . PHP_EOL));
```

Expected output:
```
admin: admin@cafe.com
barista: barista@cafe.com
user: user@cafe.com
user: mike@cafe.com
user: sarah@cafe.com
```

If no users exist, re-run seeder:
```bash
php artisan db:seed
```

### Solution 5: Test Routes
Check if routes are registered:

```bash
php artisan route:list | grep admin
php artisan route:list | grep barista
php artisan route:list | grep user
```

### Solution 6: Check Logs
View error logs:

```bash
tail -f storage/logs/laravel.log
```

## Quick Test

Open browser and go to:
```
http://localhost:8000/login
```

Login with:
- Email: `admin@cafe.com`
- Password: `password`

You should be redirected to: `http://localhost:8000/admin/dashboard`

If you see 404, check the browser console (F12) for errors.

## Common Errors

### Error: "Route not defined"
**Fix:** Run `php artisan route:clear`

### Error: "View not found"
**Fix:** Run `php artisan view:clear`

### Error: "Class not found"
**Fix:** Run `composer dump-autoload`

### Error: "Permission denied"
**Fix:** 
```bash
chmod -R 775 storage/
chmod -R 775 bootstrap/cache/
```

## Manual Testing

Test each role:

1. **Admin:**
   - Login: admin@cafe.com / password
   - Should see: Admin Dashboard with statistics
   - URL: /admin/dashboard

2. **Barista:**
   - Login: barista@cafe.com / password
   - Should see: Orders management page
   - URL: /barista/orders

3. **User:**
   - Login: user@cafe.com / password
   - Should see: Menu/Order page
   - URL: /user/orders

## Still Not Working?

Try accessing routes directly:

1. Login at: http://localhost:8000/login
2. After login, manually go to:
   - Admin: http://localhost:8000/admin/dashboard
   - Barista: http://localhost:8000/barista/orders
   - User: http://localhost:8000/user/orders

If direct access works but redirect doesn't, it's a session/cookie issue.
Clear browser cookies for localhost:8000.
