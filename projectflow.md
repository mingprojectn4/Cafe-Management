# Cafe Management System - Complete Project Flow

**Generated:** March 26, 2026  
**Project:** Laravel Cafe Management System v1.0  
**Framework:** Laravel 13.x, PHP 8.3, Bootstrap 5.3, SQLite

---

## 📁 Project Structure Overview

```
cafe-app/
├── .env                      # Database connection, app config
├── routes/
│   ├── web.php              # Web routes (browser)
│   └── api.php              # API routes (Postman/mobile)
├── app/
│   ├── Http/Controllers/    # Handle requests
│   │   ├── AuthController.php
│   │   ├── AdminController.php
│   │   ├── BaristaController.php
│   │   ├── UserController.php
│   │   └── Api/
│   ├── Models/              # Database tables
│   │   ├── User.php
│   │   ├── Product.php
│   │   ├── Order.php
│   │   └── OrderItem.php
│   └── Http/Middleware/     # Role protection
│       └── AdminMiddleware.php
├── resources/views/         # Frontend (Blade templates)
│   ├── layouts/app.blade.php
│   ├── admin/
│   ├── barista/
│   └── user/
├── database/
│   ├── migrations/          # Table structures
│   └── seeders/             # Sample data
└── config/
    └── database.php         # DB configuration
```

---

## 🔄 Flow 1: User Login (Complete Trace)

### Step 1: User Opens Login Page

**URL:** `http://127.0.0.1:8000/login`

#### File: `routes/web.php`
```php
Route::get('/login', function () {
    return view('auth.login');
})->name('login');
```

**What happens:**
1. Browser requests `/login`
2. Laravel matches route in `web.php`
3. Returns view `auth.login` (Blade template)

---

### Step 2: User Submits Login Form

#### File: `resources/views/auth/login.blade.php`
```html
<form method="POST" action="{{ route('login') }}">
    @csrf
    <input type="email" name="email" value="admin@cafe.com">
    <input type="password" name="password" value="password">
    <button type="submit">Login</button>
</form>
```

**What happens:**
1. User clicks "Login"
2. Form sends POST request to `/login`
3. Includes CSRF token (security)

---

### Step 3: Route Handles POST Request

#### File: `routes/web.php`
```php
Route::post('/login', [AuthController::class, 'login'])
     ->name('login');
```

**What happens:**
1. Laravel matches POST `/login`
2. Calls `AuthController::login()` method

---

### Step 4: Controller Processes Login

#### File: `app/Http/Controllers/AuthController.php`
```php
public function login(Request $request)
{
    // 1. Validate input
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    // 2. Try to authenticate
    if (Auth::attempt($credentials, $request->boolean('remember'))) {
        $request->session()->regenerate();

        // 3. Get authenticated user
        $user = Auth::user();

        // 4. Redirect based on role
        if ($user->isAdmin()) {
            return redirect()->intended(route('admin.dashboard'));
        } elseif ($user->isBarista()) {
            return redirect()->intended(route('barista.orders'));
        } else {
            return redirect()->intended(route('user.orders'));
        }
    }

    // 5. Failed login
    return back()->withErrors([
        'email' => 'Invalid credentials',
    ]);
}
```

**What happens:**
1. Validates email/password format
2. Checks database for matching credentials
3. If valid → creates session
4. Checks user role → redirects to different page
5. If invalid → back to login with error

---

### Step 5: Role Check (User Model)

#### File: `app/Models/User.php`
```php
public function isAdmin(): bool
{
    return $this->role === self::ROLE_ADMIN;  // 'admin'
}

public function isBarista(): bool
{
    return $this->role === self::ROLE_BARISTA;  // 'barista'
}

public function isUser(): bool
{
    return $this->role === self::ROLE_USER;  // 'user'
}
```

**What happens:**
1. `Auth::user()` returns User model
2. Controller calls `$user->isAdmin()`
3. Checks if `role` column = `'admin'`
4. Returns `true` or `false`

---

### Step 6: Redirect to Role-Specific Page

**If Admin:**
```php
return redirect()->route('admin.dashboard');
// Goes to: /admin/dashboard
```

**If Barista:**
```php
return redirect()->route('barista.orders');
// Goes to: /barista/orders
```

**If User:**
```php
return redirect()->route('user.orders');
// Goes to: /user/orders
```

---

## 🗄️ Database Connection Flow

### Step 1: Environment Config

#### File: `.env`
```env
DB_CONNECTION=sqlite
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=cafe
# DB_USERNAME=root
# DB_PASSWORD=
```

**What happens:**
- Uses SQLite (file-based database)
- Database file: `database/database.sqlite`

---

### Step 2: Database Config

#### File: `config/database.php`
```php
'default' => env('DB_CONNECTION', 'sqlite'),

'connections' => [
    'sqlite' => [
        'driver' => 'sqlite',
        'url' => env('DATABASE_URL'),
        'database' => database_path('database.sqlite'),
        'prefix' => '',
        'foreign_key_constraints' => true,
    ],
],
```

**What happens:**
- Reads `.env` settings
- Configures SQLite connection
- Points to `database/database.sqlite` file

---

### Step 3: Model Connects to Database

#### File: `app/Models/User.php`
```php
class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = ['name', 'email', 'password', 'role'];
    protected $hidden = ['password', 'remember_token'];
}
```

**What happens:**
- Extends `Authenticatable` (Laravel's base user class)
- `$fillable` = columns that can be mass-assigned
- `$hidden` = fields never returned in JSON

---

### Step 4: Query Executes

#### Example: Login Query
```php
Auth::attempt(['email' => 'admin@cafe.com', 'password' => 'password'])
```

**SQL Generated:**
```sql
SELECT * FROM users WHERE email = 'admin@cafe.com' LIMIT 1;
```

**What happens:**
1. Laravel Query Builder creates SQL
2. Executes against SQLite database
3. Returns user record
4. Verifies password hash

---

## 📦 Flow 2: Admin Views Products (Complete Trace)

### Step 1: Route Definition

#### File: `routes/web.php`
```php
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/products', [AdminController::class, 'products'])
         ->name('admin.products');
});
```

**What happens:**
- Requires authentication (`auth` middleware)
- Requires admin role (`admin` middleware)
- Maps `/admin/products` to `AdminController::products()`

---

### Step 2: Middleware Checks Role

#### File: `app/Http/Middleware/AdminMiddleware.php`
```php
public function handle($request, Closure $next)
{
    if (!auth()->check() || !auth()->user()->isAdmin()) {
        abort(403, 'Unauthorized access');
    }

    return $next($request);
}
```

**What happens:**
1. Checks if user is logged in (`auth()->check()`)
2. Checks if user is admin (`isAdmin()`)
3. If either fails → 403 error
4. If passes → continues to controller

---

### Step 3: Controller Fetches Data

#### File: `app/Http/Controllers/AdminController.php`
```php
public function products()
{
    if (!Auth::check() || !Auth::user()->isAdmin()) {
        abort(403);
    }

    $products = Product::all();

    return view('admin.products', compact('products'));
}
```

**What happens:**
1. Double-checks admin role (extra security)
2. Queries all products from database
3. Passes data to Blade view

---

### Step 4: Product Model Queries Database

#### File: `app/Models/Product.php`
```php
class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'description',
        'price',
        'category',
        'image',
        'available',
    ];

    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }
}
```

**SQL Generated:**
```sql
SELECT * FROM products;
```

---

### Step 5: View Renders HTML

#### File: `resources/views/admin/products.blade.php`
```blade
@extends('layouts.app')

@section('content')
<div class="main-content">
    <!-- Sidebar -->
    <div class="sidebar">...</div>

    <!-- Main Content -->
    <div class="content">
        <h1>Menu Items</h1>

        @foreach($products as $product)
        <div class="product-card">
            <h3>{{ $product->name }}</h3>
            <p>{{ $product->description }}</p>
            <span>${{ number_format($product->price, 2) }}</span>
        </div>
        @endforeach
    </div>
</div>
@endsection
```

**What happens:**
1. Extends main layout
2. Loops through `$products` array
3. Outputs HTML for each product
4. Laravel compiles Blade to PHP
5. Browser receives final HTML

---

## 🛒 Flow 3: User Places Order (Complete Trace)

### Step 1: User Browses Menu

#### File: `app/Http/Controllers/UserController.php`
```php
public function orders()
{
    $products = Product::where('available', true)
        ->orderBy('category')
        ->get()
        ->groupBy('category');

    return view('user.orders', compact('products'));
}
```

**SQL Generated:**
```sql
SELECT * FROM products 
WHERE available = 1 
ORDER BY category;
```

**Result:**
```php
[
  'coffee' => [Product1, Product2, ...],
  'tea' => [Product3, Product4, ...],
  'pastry' => [Product5, ...],
]
```

---

### Step 2: User Adds to Cart (JavaScript)

#### File: `resources/views/user/orders.blade.php`
```javascript
let cart = [];

function addToCart(id, name, price) {
    const existingItem = cart.find(item => item.id === id);
    if (existingItem) {
        existingItem.quantity++;
    } else {
        cart.push({ id, name, price, quantity: 1 });
    }
    updateCart();
}
```

**What happens:**
1. User clicks product
2. JavaScript adds to cart array (in browser memory)
3. Updates cart display
4. **NOT sent to server yet**

---

### Step 3: User Places Order (Form Submit)

#### File: `resources/views/user/orders.blade.php`
```javascript
document.getElementById('placeOrder').addEventListener('click', async function() {
    const orderData = {
        items: cart.map(item => ({
            product_id: item.id,
            quantity: item.quantity
        })),
        order_type: document.getElementById('orderType').value,
        notes: document.getElementById('orderNotes').value
    };

    const response = await fetch('/user/orders/place', {
        method: 'POST',
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(orderData)
    });

    const result = await response.json();
    
    if (result.success) {
        // Show success modal
    }
});
```

**What happens:**
1. Converts cart to JSON
2. Sends POST request to `/user/orders/place`
3. Includes CSRF token (security)
4. Waits for server response

---

### Step 4: Route Handles Order Placement

#### File: `routes/web.php`
```php
Route::post('/user/orders/place', [UserController::class, 'placeOrder'])
     ->middleware('auth');
```

---

### Step 5: Controller Creates Order

#### File: `app/Http/Controllers/UserController.php`
```php
public function placeOrder(Request $request)
{
    $request->validate([
        'items' => 'required|array|min:1',
        'items.*.product_id' => 'required|exists:products,id',
        'items.*.quantity' => 'required|integer|min:1',
        'order_type' => 'required|in:dine-in,takeaway',
        'notes' => 'nullable|string|max:500',
    ]);

    $user = Auth::user();
    $items = $request->items;

    // Calculate total
    $totalPrice = 0;
    foreach ($items as $item) {
        $product = Product::find($item['product_id']);
        $totalPrice += $product->price * $item['quantity'];
    }

    // Create order
    $order = Order::create([
        'user_id' => $user->id,
        'status' => 'pending',
        'total_price' => $totalPrice,
        'order_type' => $request->order_type,
        'notes' => $request->notes,
    ]);

    // Create order items
    foreach ($items as $item) {
        $product = Product::find($item['product_id']);
        OrderItem::create([
            'order_id' => $order->id,
            'product_id' => $product->id,
            'quantity' => $item['quantity'],
            'price' => $product->price,
        ]);
    }

    return response()->json([
        'success' => true,
        'order' => $order,
    ]);
}
```

**What happens:**
1. Validates all input data
2. Gets authenticated user
3. Loops through items, calculates total
4. Creates Order record
5. Creates OrderItem records for each product
6. Returns JSON response

---

### Step 6: Database Transactions

#### SQL Executed:
```sql
-- 1. Create Order
INSERT INTO orders (user_id, status, total_price, order_type, notes, created_at)
VALUES (3, 'pending', 12.50, 'dine-in', 'Extra hot', NOW());

-- 2. Get Order ID (e.g., 7)

-- 3. Create Order Items
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (7, 1, 2, 3.50);

INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (7, 3, 1, 4.75);
```

---

### Step 7: Success Response

#### JSON Response:
```json
{
  "success": true,
  "order": {
    "id": 7,
    "user_id": 3,
    "status": "pending",
    "total_price": 12.50,
    "order_type": "dine-in",
    "notes": "Extra hot",
    "created_at": "2026-03-26T10:30:00.000000Z"
  }
}
```

#### JavaScript Shows Success:
```javascript
if (result.success) {
    document.getElementById('orderId').textContent = '#' + result.order.id;
    new bootstrap.Modal(document.getElementById('orderModal')).show();
    cart = [];  // Clear cart
    updateCart();
}
```

---

## 🔐 Role Protection Flow

### Middleware Registration

#### File: `bootstrap/app.php` (or `app/Http/Kernel.php` in older Laravel)
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'admin' => \App\Http\Middleware\AdminMiddleware::class,
    ]);
})
```

---

### Admin Route Protection

#### File: `routes/web.php`
```php
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard'])
         ->name('admin.dashboard');
    
    Route::get('/admin/products', [AdminController::class, 'products'])
         ->name('admin.products');
});
```

**Flow:**
```
Request → auth middleware → admin middleware → Controller
            ↓                    ↓
      Check logged in?      Check isAdmin()?
            ↓                    ↓
         Yes → Continue     Yes → Continue
         No → Login         No → 403 Error
```

---

### Barista Route Protection

#### File: `routes/web.php`
```php
Route::middleware(['auth', 'barista'])->group(function () {
    Route::get('/barista/orders', [BaristaController::class, 'orders'])
         ->name('barista.orders');
});
```

#### File: `app/Http/Middleware/BaristaMiddleware.php`
```php
public function handle($request, Closure $next)
{
    if (!auth()->check() || 
        (!auth()->user()->isBarista() && !auth()->user()->isAdmin())) {
        abort(403, 'Unauthorized access');
    }

    return $next($request);
}
```

**What happens:**
- Allows barista OR admin (admin can access everything)

---

## 📊 Database Schema

### Migrations (Table Structure)

#### File: `database/migrations/0001_01_01_000000_create_users_table.php`
```php
public function up(): void
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->string('role')->default('user');  // admin, barista, user
        $table->rememberToken();
        $table->timestamps();
    });
}
```

#### File: `database/migrations/xxxx_create_products_table.php`
```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description')->nullable();
    $table->decimal('price', 8, 2);
    $table->string('category')->nullable();
    $table->string('image')->nullable();
    $table->boolean('available')->default(true);
    $table->timestamps();
});
```

#### File: `database/migrations/xxxx_create_orders_table.php`
```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->string('status')->default('pending');  // pending, preparing, completed, cancelled
    $table->decimal('total_price', 8, 2);
    $table->string('order_type');  // dine-in, takeaway
    $table->text('notes')->nullable();
    $table->timestamp('completed_at')->nullable();
    $table->timestamps();
});
```

#### File: `database/migrations/xxxx_create_order_items_table.php`
```php
Schema::create('order_items', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_id')->constrained()->onDelete('cascade');
    $table->foreignId('product_id')->constrained()->onDelete('cascade');
    $table->integer('quantity');
    $table->decimal('price', 8, 2);
    $table->text('special_instructions')->nullable();
    $table->timestamps();
});
```

---

## 🎯 Complete Request-Response Cycle

```
┌─────────────────────────────────────────────────────────────────┐
│  1. BROWSER                                                     │
│     User clicks "Login"                                         │
│     POST http://127.0.0.1:8000/login                            │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. LARAVEL ROUTER                                              │
│     routes/web.php                                              │
│     Matches: Route::post('/login', [AuthController::login])     │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. MIDDLEWARE                                                  │
│     Check if already logged in                                  │
│     If yes → redirect to dashboard                              │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  4. CONTROLLER                                                  │
│     app/Http/Controllers/AuthController.php                     │
│     login() method                                              │
│     - Validate input                                            │
│     - Auth::attempt() → Check database                          │
│     - Check role (admin/barista/user)                           │
│     - Redirect to appropriate page                              │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  5. MODEL / DATABASE                                            │
│     app/Models/User.php                                         │
│     Query: SELECT * FROM users WHERE email = ?                  │
│     Returns: User object                                        │
│     Database: database/database.sqlite                          │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  6. CONTROLLER (continued)                                      │
│     if ($user->isAdmin())                                       │
│         return redirect('/admin/dashboard')                     │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  7. ROUTE FOR DASHBOARD                                         │
│     routes/web.php                                              │
│     Route::get('/admin/dashboard', [AdminController::dashboard])│
│     Middleware: auth, admin                                     │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  8. MIDDLEWARE CHECK                                            │
│     AdminMiddleware.php                                         │
│     Check: Is user admin?                                       │
│     Yes → Continue                                              │
│     No → 403 Error                                              │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  9. CONTROLLER                                                  │
│     app/Http/Controllers/AdminController.php                    │
│     dashboard() method                                          │
│     - Fetch stats from database                                 │
│     - return view('admin.dashboard')                            │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  10. BLADE VIEW                                                 │
│      resources/views/admin/dashboard.blade.php                  │
│      - Extends layouts/app.blade.php                            │
│      - Renders HTML with stats                                  │
│      - Includes sidebar                                         │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  11. LAYOUT                                                     │
│      resources/views/layouts/app.blade.php                      │
│      - HTML head, CSS, JS                                       │
│      - Yield content                                            │
│      - Final HTML output                                        │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  12. BROWSER                                                    │
│      Receives HTML                                              │
│      Renders page                                               │
│      User sees Admin Dashboard                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Summary: Key Connections

| Component | Connects To | How |
|-----------|-------------|-----|
| **Routes** | Controllers | `Route::get('/path', [Controller::class, 'method'])` |
| **Controllers** | Models | `Product::all()`, `Order::create()` |
| **Models** | Database | Eloquent ORM → SQL queries |
| **Controllers** | Views | `return view('name', $data)` |
| **Views** | Layouts | `@extends('layouts.app')` |
| **Middleware** | Routes | `Route::middleware(['auth'])` |
| **Frontend** | Backend | Forms, Fetch API (AJAX) |
| **Database** | `.env` | `DB_CONNECTION=sqlite` |

---

## 🔑 Default Login Credentials

| Role | Email | Password | Redirect |
|------|-------|----------|----------|
| **Admin** | admin@cafe.com | password | /admin/dashboard |
| **Barista** | barista@cafe.com | password | /barista/orders |
| **User** | user@cafe.com | password | /user/orders |

---

## 🎯 All Roles Summary

### ADMIN ☕
**Login:** `admin@cafe.com` / `password`

**Capabilities:**
- Dashboard with statistics
- Manage products (CRUD)
- View all orders
- Manage staff accounts
- View all customers

**Pages:**
- `/admin/dashboard`
- `/admin/orders`
- `/admin/products`
- `/admin/staff`
- `/admin/customers`

---

### BARISTA ☕
**Login:** `barista@cafe.com` / `password`

**Capabilities:**
- View order queue
- Update order status (pending → preparing → completed)
- View order history

**Pages:**
- `/barista/orders`
- `/barista/history`

---

### USER (Customer) ☕
**Login:** `user@cafe.com` / `password`

**Capabilities:**
- Browse menu
- Place orders
- Add special instructions
- View order history
- Track order status

**Pages:**
- `/user/orders`
- `/user/my-orders`
- `/user/order/{id}`

---

## 🚀 Commands Reference

```bash
# Start development server
php artisan serve

# Reset database with sample data
php artisan migrate:fresh --seed

# Clear config cache
php artisan config:clear

# Create storage symlink for images
php artisan storage:link

# View all routes
php artisan route:list

# Run tests
php artisan test
```

---

## 📌 API Endpoints Reference

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/login` | Public | Login user |
| GET | `/api/products` | Public | Get available products |
| GET | `/api/products/all` | Sanctum | Get all products (admin) |
| POST | `/api/products` | Sanctum | Create product (admin) |
| PUT | `/api/products/{id}` | Sanctum | Update product (admin) |
| DELETE | `/api/products/{id}` | Sanctum | Delete product (admin) |
| POST | `/api/logout` | Sanctum | Logout user |
| GET | `/api/user` | Sanctum | Get current user |
| GET | `/api/orders` | Session | Get orders |
| POST | `/api/orders` | Session | Create order |
| GET | `/api/orders/my-orders` | Session | Get user's orders |
| GET | `/api/orders/{id}` | Session | Get order details |
| PUT | `/api/orders/{id}/status` | Session | Update order status |
| GET | `/api/orders/dashboard/stats` | Session | Dashboard statistics |
| POST | `/api/orders/{id}/cancel` | Session | Cancel order |

---

**Document Generated:** March 26, 2026  
**Project Status:** Complete ✅  
**Framework:** Laravel 13.x + Bootstrap 5.3 + SQLite
