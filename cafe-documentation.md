### 1.1 What Is This Project?
**Cafe Management System** is a web-based POS (Point of Sale) application for cafe operations.
**Features:**
- Role-based access for Admin, Barista, and Customers
- Order management (pending → preparing → completed)
- Product catalog with categories
- Shopping cart functionality
- Real-time order tracking
- Staff management
### 1.2 Technology Stack
| Component | Technology |
|-----------|------------|
| Backend | Laravel 13 (PHP 8.3+) |
| Database | SQLite |
| Frontend | Bootstrap 5, Blade Templates |
| Auth | Laravel Sanctum |
| Styling | Custom Pink Theme (CSS Variables) |
### 1.3 Database Structure
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATABASE SCHEMA                                   │
└─────────────────────────────────────────────────────────────────────────────┘
┌─────────────────┐         ┌─────────────────┐
│     users       │         │    products      │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │         │ id (PK)         │
│ name            │         │ name            │
│ email           │         │ description     │
│ password        │         │ price           │
│ role            │         │ category        │
│ created_at      │         │ image           │
│ updated_at      │         │ available       │
└────────┬────────┘         └────────┬────────┘
         │                           │
         │    1         ┌────────────┘
         │    │         │    N
         │    ▼         │    │
         │ ┌─────────────────────────┐
         │ │        orders           │
         │ ├─────────────────────────┤
N        │ │ id (PK)                 │
         └──│ user_id (FK)           │◄────┐
            │ status                 │     │ 1
            │ total_price            │     │ │
            │ order_type             │     │ ▼
            │ notes                  │     │ ┌─────────────────────────┐
            │ completed_at           │     │ │      order_items        │
            │ created_at            │     │ ├─────────────────────────┤
            │ updated_at            │     │ │ id (PK)                  │
            └───────────────────────┘     │ │ order_id (FK)           │────┘
                        │                 │ │ product_id (FK)─────────┴──►
                        │                 │ │ quantity                 │
                        │                 │ │ price                    │
                        │                 │ │ special_instructions     │
                        │                 │ └─────────────────────────┘
                        ▼
                   (referenced by products.id)
---
## PART 2: CODE EXPLANATION - HOW EVERYTHING WORKS
### 2.1 Authentication Flow
User enters credentials
       ↓
POST /login (AuthController@login)
       ↓
Validates email & password
       ↓
Authenticates with Laravel's auth system
       ↓
Creates session / Returns token
       ↓
Redirects based on role:
  - Admin → /admin/dashboard
  - Barista → /barista/orders
  - User → /user/orders
### 2.2 How Baristas Receive Orders (Complete Flow)
╔══════════════════════════════════════════════════════════════════════╗
║                    ORDER PLACEMENT TO COMPLETION                    ║
╚══════════════════════════════════════════════════════════════════════╝
CUSTOMER SIDE                          CAFE SIDE
─────────────                          ─────────
┌──────────────┐                     ┌──────────────────────┐
│   BROWSER    │                     │     BARISTA POS      │
└──────┬───────┘                     └──────────┬───────────┘
       │                                        │
       │ 1. Browse Products                    │
       │    GET /user/orders                   │
       │◄──────────────────────────────────────│
       │                                        │
       │ 2. Add Items to Cart                   │
       │    (JavaScript updates session)        │
       │                                        │
       │ 3. Click "Place Order"                 │
       │    POST /user/orders/place            │
       │───────────────────────────────────────►│
       │                                        │
       │         ┌─────────────────────────────┐ │
       │         │   UserController@placeOrder   │ │
       │         └──────────────┬──────────────┘ │
       │                        │                 │
       │                        ▼                 │
       │         ┌─────────────────────────────┐ │
       │         │   1. Get cart from session    │ │
       │         │   2. Validate cart items      │ │
       │         │   3. Calculate total price    │ │
       │         │   4. Create Order record      │ │
       │         │      status = 'pending'       │ │
       │         │   5. Create OrderItem records │ │
       │         │   6. Clear cart session      │ │
       │         └─────────────────────────────┘ │
       │                                        │
       │ 4. GET /barista/orders                │
       │    (Auto-refresh every 30 seconds)     │
       │◄──────────────────────────────────────│
       │                                        │
       │                            ┌───────────▼───────────┐
       │                            │ BaristaController     │
       │                            │ @orders method        │
       │                            └───────────┬───────────┘
       │                                        │
       │                                        ▼
       │                            ┌───────────────────────┐
       │                            │ Query:                  │
       │                            │ Order::whereIn(        │
       │                            │   'status',           │
       │                            │   'pending',          │
       │                            │    'preparing'       │
       │                            │ ).get()               │
       │                            └───────────────────────┘
       │                                        │
       │ 5. POST /barista/orders/{id}/status   │
       │    data: {status: 'preparing'}        │
       │◄──────────────────────────────────────│
       │                                        │
       │                            ┌───────────▼───────────┐
       │                            │ Status updated!       │
       │                            │ Auto-refresh shows   │
       │                            │ new status          │
       │                            └─────────────────────┘
### 2.3 Shopping Cart System
```php
// CART STRUCTURE (Stored in Session)
session('cart') = [
    0 => [
        'product_id' => 1,
        'name' => 'Espresso',
        'price' => 3.50,
        'quantity' => 2,
        'instructions' => 'Extra shot'
    ],
    1 => [
        'product_id' => 5,
        'name' => 'Croissant',
        'price' => 2.50,
        'quantity' => 1,
        'instructions' => null
    ]
];
// ADD TO CART
public function addToCart(Request $request)
{
    $cart = session()->get('cart', []);
    $productId = $request->product_id;
    
    if (isset($cart[$productId])) {
        $cart[$productId]['quantity'] += $request->quantity ?? 1;
    } else {
        $product = Product::find($productId);
        $cart[$productId] = [
            'product_id' => $product->id,
            'name' => $product->name,
            'price' => $product->price,
            'quantity' => $request->quantity ?? 1,
            'instructions' => $request->instructions,
        ];
    }
    
    session()->put('cart', $cart);
    return response()->json(['success' => true, 'cart_count' => count($cart)]);
}
// CLEAR CART AFTER ORDER
session()->forget('cart');
2.4 Order Placement Code
// app/Http/Controllers/UserController.php
public function placeOrder(Request $request)
{
    // STEP 1: Validate
    $validated = $request->validate([
        'order_type' => 'required|in:dine_in,takeaway',
        'notes' => 'nullable|string|max:500',
    ]);
    // STEP 2: Get cart
    $cart = session()->get('cart', []);
    if (empty($cart)) {
        return back()->with('error', 'Your cart is empty!');
    }
    // STEP 3: Calculate total
    $totalPrice = 0;
    foreach ($cart as $item) {
        $product = Product::find($item['product_id']);
        $totalPrice += $product->price * $item['quantity'];
    }
    // STEP 4: Create Order
    $order = Order::create([
        'user_id' => auth()->id(),
        'status' => 'pending',
        'total_price' => $totalPrice,
        'order_type' => $validated['order_type'],
        'notes' => $validated['notes'] ?? null,
    ]);
    // STEP 5: Create Order Items
    foreach ($cart as $item) {
        $product = Product::find($item['product_id']);
        OrderItem::create([
            'order_id' => $order->id,
            'product_id' => $item['product_id'],
            'quantity' => $item['quantity'],
            'price' => $product->price,
            'special_instructions' => $item['instructions'] ?? null,
        ]);
    }
    // STEP 6: Clear cart
    session()->forget('cart');
    // STEP 7: Redirect
    return redirect()
        ->route('user.order-detail', $order->id)
        ->with('success', 'Order placed successfully!');
}
2.5 Order Status Flow
pending → preparing → completed
    ↓
cancelled (only from pending)
// Status transitions (enforced in controller)
$allowedTransitions = [
    'pending' => ['preparing', 'cancelled'],
    'preparing' => ['completed', 'cancelled'],
    'completed' => [],  // Cannot change
    'cancelled' => [],  // Cannot change
];
2.6 Auto-Refresh for Barista
// resources/views/barista/orders.blade.php
@push('scripts')
<script>
    // Refresh every 30 seconds
    const REFRESH_INTERVAL = 30000;
    
    function refreshOrders() {
        if (!document.hidden) {
            location.reload();
        }
    }
    
    let refreshTimer = setInterval(refreshOrders, REFRESH_INTERVAL);
    
    // Show countdown
    let countdown = REFRESH_INTERVAL / 1000;
    setInterval(() => {
        countdown--;
        document.getElementById('refresh-countdown').textContent = countdown;
        if (countdown <= 0) countdown = REFRESH_INTERVAL / 1000;
    }, 1000);
</script>
@endpush
2.7 API Routes Explained
// routes/api.php
// PUBLIC (No auth)
Route::post('/login', [AuthController::class, 'login']);
Route::get('/products', [ProductController::class, 'index']);
// SANCTUM PROTECTED (Token auth - for mobile apps)
Route::middleware(['auth:sanctum'])->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::get('/products/all', [ProductController::class, 'all']);
    Route::post('/products', [ProductController::class, 'store']);
    Route::put('/products/{id}', [ProductController::class, 'update']);
    Route::delete('/products/{id}', [ProductController::class, 'destroy']);
    Route::get('/staff', [StaffController::class, 'index']);
});
// SESSION PROTECTED (Web auth - for browser)
Route::middleware(['auth'])->group(function () {
    Route::get('/orders', [OrderController::class, 'index']);
    Route::post('/orders', [OrderController::class, 'store']);
    Route::put('/orders/{id}/status', [OrderController::class, 'updateStatus']);
    Route::post('/orders/{id}/cancel', [OrderController::class, 'cancel']);
});
2.8 Eloquent Relationships
// User Model
class User extends Model
{
    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }
}
// Order Model
class Order extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
    
    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }
}
// OrderItem Model
class OrderItem extends Model
{
    public function order(): BelongsTo
    {
        return $this->belongsTo(Order::class);
    }
    
    public function product(): BelongsTo
    {
        return $this->belongsTo(Product::class);
    }
}
// Product Model
class Product extends Model
{
    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }
}
---
PART 3: TEACHER Q&A - POTENTIAL QUESTIONS
Basic/Fundamental Questions
Q1: What is Laravel?
> Laravel is a PHP framework for web development that follows the MVC (Model-View-Controller) pattern. It provides built-in features like routing, authentication, and database management.
Q2: What is a POS (Point of Sale) system?
> A POS system is software that handles sales transactions, tracks inventory, and manages customer orders. For a cafe, it includes menu management, order processing, and payment handling.
Q3: Why did you choose Laravel for this project?
> Laravel offers built-in authentication (Sanctum), ORM (Eloquent), routing, and MVC structure. It reduces development time and provides security features out-of-the-box.
Q4: What database did you use and why?
> SQLite - it's serverless, requires no setup, and is perfect for small-to-medium applications like this student project.
Q5: What is the difference between web routes and API routes?
> Web routes render Blade templates (HTML) for the browser. API routes return JSON data for external applications or mobile apps.
---
Architecture & Structure Questions
Q6: Explain your project structure
> - app/ - Business logic (Controllers, Models)
> - routes/ - URL definitions
> - resources/views/ - Blade templates
> - database/ - Migrations and seeders
Q7: What is MVC pattern? How does it apply here?
> - Model = Product, Order, User (database interaction)
> - View = Blade templates (HTML output)
> - Controller = AdminController, BaristaController (request handling)
Q8: What is the relationship between Orders and OrderItems?
> One Order has many OrderItems (1:N). Each OrderItem belongs to one Order and references one Product.
---
Authentication & Security Questions
Q9: How does user authentication work?
> Laravel Sanctum provides API tokens for SPA authentication. Users log in via /api/login, receive a token, and include it in subsequent requests.
Q10: How did you implement role-based access (Admin/Barista/User)?
> The User model has a role field. Middleware checks the role before allowing access to routes. Each role sees different navigation links and pages.
Q11: What security measures are in place?
> - CSRF protection on forms
> - Password hashing (bcrypt)
> - Route middleware for auth checks
> - Input validation on all forms
---
Feature Implementation Questions
Q12: Explain the order flow from start to finish
> 1. Customer browses menu
> 2. Adds items to cart
> 3. Places order (status: pending)
> 4. Barista sees order, updates to "preparing"
> 5. Barista marks as "completed"
> 6. Customer can view order history
Q13: How does the shopping cart work?
> Cart items are stored in session. When order is placed, items are saved to order_items table with quantity and price, and cart is cleared.
Q14: Can orders be cancelled?
> Yes - only orders with "pending" status can be cancelled by the customer.
Q15: How does auto-refresh work for baristas?
> The barista orders page uses JavaScript setInterval to reload the page every 30 seconds to show new orders.
---
Database & Technical Questions
Q16: What are migrations?
> Migrations are version control for your database. They define table structures (fields, types, constraints) and can be run with php artisan migrate.
Q17: What is Eloquent ORM?
> Eloquent is Laravel's ORM that lets you interact with database tables using PHP classes instead of raw SQL. Example: Product::all() instead of SELECT * FROM products.
Q18: What are seeders used for?
> Seeders populate the database with sample data (users, products, orders) for testing. Run with php artisan db:seed.
---
Advanced/Challenging Questions
Q19: How would you add real-time notifications?
> Using Laravel Echo + Pusher - when an order status changes, broadcast an event. Baristas would receive instant notifications without refreshing.
Q20: How would you handle payment integration?
> Integrate Stripe or PayPal SDK. Add payment fields to checkout, create a payment intent, confirm payment, then create the order only if payment succeeds.
Q21: How would you scale this for multiple cafe branches?
> Add a cafe_id to all tables, implement multi-tenancy, use a centralized database with separate schemas, or use different database connections per branch.
Q22: What if two baristas update the same order?
> Use database transactions and optimistic locking - check if the record was modified before saving, or use Laravel's atomic updates.
Q23: How would you add inventory management?
> Add stock field to products table. Decrement stock when order is placed. Add low-stock alerts. Prevent orders when stock is 0.
Q24: What are the limitations of this system?
> - No real-time updates (polling only)
> - No payment processing
> - No reporting/analytics
> - SQLite not suitable for production scale
> - No mobile-responsive POS interface
---
Future Enhancement Questions
Q25: What features would you add next?
> - Payment integration (Stripe)
> - Real-time WebSocket updates
> - Table reservation system
> - Inventory tracking
> - Sales reports & analytics
> - Mobile app version
---
PART 4: ADVANCED TOPICS
4.1 Database Migrations
// database/migrations/2024_01_01_000001_create_users_table.php
public function up(): void
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->string('role')->default('user'); // admin, barista, user
        $table->rememberToken();
        $table->timestamps();
    });
}
// database/migrations/2024_01_01_000002_create_products_table.php
public function up(): void
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description')->nullable();
        $table->decimal('price', 8, 2);
        $table->string('category');
        $table->string('image')->nullable();
        $table->boolean('available')->default(true);
        $table->timestamps();
    });
}
// database/migrations/2024_01_01_000003_create_orders_table.php
public function up(): void
{
    Schema::create('orders', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->onDelete('cascade');
        $table->string('status')->default('pending');
        $table->decimal('total_price', 8, 2);
        $table->string('order_type'); // dine_in, takeaway
        $table->text('notes')->nullable();
        $table->timestamp('completed_at')->nullable();
        $table->timestamps();
        $table->index('status');
        $table->index('user_id');
    });
}
// database/migrations/2024_01_01_000004_create_order_items_table.php
public function up(): void
{
    Schema::create('order_items', function (Blueprint $table) {
        $table->id();
        $table->foreignId('order_id')->constrained()->onDelete('cascade');
        $table->foreignId('product_id')->constrained()->onDelete('restrict');
        $table->integer('quantity');
        $table->decimal('price', 8, 2);
        $table->text('special_instructions')->nullable();
        $table->timestamps();
    });
}
4.2 Seeders
// database/seeders/DatabaseSeeder.php
public function run(): void
{
    $this->call([
        UserSeeder::class,
        ProductSeeder::class,
        OrderSeeder::class,
    ]);
}
// database/seeders/UserSeeder.php
public function run(): void
{
    // Admin
    User::create([
        'name' => 'Admin User',
        'email' => 'admin@cafe.com',
        'password' => Hash::make('password'),
        'role' => 'admin',
    ]);
    // Barista
    User::create([
        'name' => 'Barista Staff',
        'email' => 'barista@cafe.com',
        'password' => Hash::make('password'),
        'role' => 'barista',
    ]);
    // Customer
    User::create([
        'name' => 'John Doe',
        'email' => 'user@cafe.com',
        'password' => Hash::make('password'),
        'role' => 'user',
    ]);
}
// database/seeders/ProductSeeder.php
public function run(): void
{
    $products = [
        ['name' => 'Espresso', 'price' => 3.50, 'category' => 'coffee'],
        ['name' => 'Latte', 'price' => 4.50, 'category' => 'coffee'],
        ['name' => 'Cappuccino', 'price' => 4.50, 'category' => 'coffee'],
        ['name' => 'Americano', 'price' => 4.00, 'category' => 'coffee'],
        ['name' => 'Mocha', 'price' => 5.00, 'category' => 'coffee'],
        ['name' => 'Cold Brew', 'price' => 4.50, 'category' => 'coffee'],
        ['name' => 'Green Tea', 'price' => 3.00, 'category' => 'tea'],
        ['name' => 'Earl Grey', 'price' => 3.50, 'category' => 'tea'],
        ['name' => 'Chai Latte', 'price' => 4.50, 'category' => 'tea'],
        ['name' => 'Matcha Latte', 'price' => 5.00, 'category' => 'tea'],
        ['name' => 'Croissant', 'price' => 3.50, 'category' => 'pastry'],
        ['name' => 'Blueberry Muffin', 'price' => 3.00, 'category' => 'pastry'],
        ['name' => 'Cinnamon Roll', 'price' => 4.00, 'category' => 'pastry'],
        ['name' => 'Chocolate Cake', 'price' => 5.00, 'category' => 'pastry'],
        ['name' => 'Avocado Toast', 'price' => 8.00, 'category' => 'snacks'],
        ['name' => 'Caesar Salad', 'price' => 9.00, 'category' => 'snacks'],
        ['name' => 'Sandwich Club', 'price' => 10.00, 'category' => 'snacks'],
    ];
    foreach ($products as $product) {
        Product::create($product);
    }
}
4.3 Form Validation
// Validation in Controller
$validated = $request->validate([
    'order_type' => 'required|in:dine_in,takeaway',
    'notes' => 'nullable|string|max:500',
    'items' => 'required|array|min:1',
    'items.*.product_id' => 'required|exists:products,id',
    'items.*.quantity' => 'required|integer|min:1',
]);
// Custom messages
$request->validate([
    'email' => 'required|email',
    'price' => 'required|numeric|min:0',
], [
    'email.required' => 'Please enter your email address.',
    'email.email' => 'Please enter a valid email address.',
    'price.min' => 'Price cannot be negative.',
]);
4.4 Middleware
// app/Http/Middleware/AdminMiddleware.php
public function handle(Request $request, Closure $next): Response
{
    if (!auth()->check()) {
        return redirect()->route('login')
            ->with('error', 'Please log in to access this page.');
    }
    if (!auth()->user()->isAdmin()) {
        return redirect()->route('dashboard')
            ->with('error', 'You do not have permission to access that page.');
    }
    return $next($request);
}
// Usage in routes
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard']);
});
4.5 Flash Messages
// Controller - Success
return redirect()->route('user.order-detail', $order->id)
    ->with('success', 'Order placed successfully!');
// Controller - Error
return back()->with('error', 'Your cart is empty!');
// Blade - Display
@if(session('success'))
    <div class="alert alert-success">{{ session('success') }}</div>
@endif
@if(session('error'))
    <div class="alert alert-danger">{{ session('error') }}</div>
@endif
4.6 Artisan Commands
# Development
php artisan serve                          # Start server
php artisan serve --port=8080            # Specific port
# Database
php artisan migrate                       # Run migrations
php artisan migrate:fresh                # Reset & migrate
php artisan migrate:fresh --seed         # Fresh + seed
php artisan db:seed                       # Run seeders
php artisan db:seed --class=ProductSeeder # Specific seeder
# Routes
php artisan route:list                   # List routes
php artisan route:list --path=api        # API routes only
# Cache
php artisan config:cache                 # Cache config
php artisan route:cache                  # Cache routes
php artisan cache:clear                  # Clear cache
php artisan view:clear                   # Clear views
# Tinker
php artisan tinker                       # Interactive shell
# Testing
php artisan test                         # Run tests
# Key
php artisan key:generate                # Generate app key
php artisan key:generate --show         # Show key
# Storage
php artisan storage:link                # Create storage link
4.7 Security Best Practices
// Password Hashing (Automatic)
User::create([
    'password' => 'secret123'  // Will be auto-hashed
]);
// Mass Assignment Protection
class Product extends Model
{
    protected $fillable = [
        'name', 'description', 'price', 'category', 'image', 'available'
    ];
}
// CSRF Protection (Automatic in Blade)
<form method="POST">
    @csrf  {{-- Required for all POST/PUT/DELETE --}}
</form>
// SQL Injection Prevention (Eloquent is safe)
$products = Product::where('category', $category)->get();
// XSS Prevention (Blade escapes by default)
{{ $user->name }}  {{-- Safe - escaped --}}
{!! $user->bio !!}  {{-- Dangerous - raw HTML --}}
4.8 Bootstrap Components Used
{{-- Cards --}}
<div class="card">
    <div class="card-header">Header</div>
    <div class="card-body">
        <h5 class="card-title">Title</h5>
        <p class="card-text">Content</p>
        <a href="#" class="btn btn-primary">Button</a>
    </div>
</div>
{{-- Badges --}}
<span class="badge bg-warning">Pending</span>
<span class="badge bg-success">Completed</span>
<span class="badge bg-danger">Cancelled</span>
{{-- Tables --}}
<table class="table table-striped table-hover">
    <thead class="table-dark">
        <tr><th>ID</th><th>Name</th></tr>
    </thead>
    <tbody>
        @foreach($items as $item)
            <tr><td>{{ $item->id }}</td><td>{{ $item->name }}</td></tr>
        @endforeach
    </tbody>
</table>
{{-- Forms --}}
<input type="text" class="form-control" name="name">
<select class="form-select" name="category">
    <option value="coffee">Coffee</option>
</select>
<textarea class="form-control" name="notes" rows="3"></textarea>
{{-- Grid --}}
<div class="row">
    <div class="col-md-4">Column 1</div>
    <div class="col-md-4">Column 2</div>
    <div class="col-md-4">Column 3</div>
</div>
{{-- Alerts --}}
<div class="alert alert-success">Success message</div>
<div class="alert alert-danger">Error message</div>
<div class="alert alert-warning">Warning message</div>
---
PART 5: DEMO CREDENTIALS
Role
Admin
Barista
User
---
## PART 6: FUTURE ENHANCEMENTS
### Short-term
- Real-time notifications (WebSocket)
- Payment integration (Stripe/PayPal)
- Mobile-responsive design
- Email notifications
### Long-term
- Table reservation system
- Inventory management
- Sales analytics & reporting
- Multi-branch support
- Mobile app version
- Loyalty/reward program
---
PART 7: TROUBLESHOOTING
Common Issues
Error: "CSRF token mismatch"
> Add @csrf in your forms
Error: "419 Page Expired"
> Session expired - refresh and try again
Error: "Table doesn't exist"
> Run php artisan migrate
Error: "Class not found"
> Run composer dump-autoload
Error: "Target class does not exist"
> Run php artisan cache:clear
Useful Commands
php artisan route:list                    # Check routes
php artisan migrate:status              # Check migrations
php artisan tinker                       # Interactive testing
tail -f storage/logs/laravel.log        # View logs
php artisan config:clear                 # Clear config cache
---
PART 8: PROJECT STRUCTURE
cafe-app/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── AuthController.php
│   │   │   │   ├── OrderController.php
│   │   │   │   ├── ProductController.php
│   │   │   │   └── StaffController.php
│   │   │   ├── AdminController.php
│   │   │   ├── AuthController.php
│   │   │   ├── BaristaController.php
│   │   │   └── UserController.php
│   │   └── Middleware/
│   │       └── AdminMiddleware.php
│   └── Models/
│       ├── User.php
│       ├── Product.php
│       ├── Order.php
│       └── OrderItem.php
├── bootstrap/
│   └── app.php
├── config/
├── database/
│   ├── migrations/
│   └── seeders/
├── public/
├── resources/
│   └── views/
│       ├── layouts/
│       │   └── app.blade.php
│       ├── auth/
│       │   └── login.blade.php
│       ├── admin/
│       │   ├── dashboard.blade.php
│       │   ├── orders.blade.php
│       │   ├── products.blade.php
│       │   ├── staff.blade.php
│       │   └── customers.blade.php
│       ├── barista/
│       │   ├── orders.blade.php
│       │   └── history.blade.php
│       └── user/
│           ├── orders.blade.php
│           ├── my-orders.blade.php
│           └── order-detail.blade.php
├── routes/
│   ├── web.php
│   └── api.php
├── storage/
├── tests/
├── .env
├── composer.json
└── README.md
---
## PART 9: PRESENTATION OUTLINE
### Timeline (~25-30 minutes)
1. **Introduction** (2-3 min)
   - Project title & purpose
   - What is POS?
   - Why build this system?
2. **Technology Stack** (1-2 min)
   - Laravel 13, SQLite, Bootstrap 5
   - Architecture overview
3. **Features by Role** (5-7 min)
   - Admin: Dashboard, Products, Staff
   - Barista: Order queue, Status updates
   - Customer: Menu, Cart, Orders
4. **Code Walkthrough** (5-8 min)
   - Authentication flow
   - Order placement process
   - Barista order receiving
   - Database relationships
5. **Demo** (5-8 min)
   - Login as each role
   - Place order
   - Update status
   - View history
6. **API Structure** (2 min)
   - RESTful endpoints
   - JSON responses
7. **Q&A** (5 min)
   - Address teacher questions
   - Discuss future improvements
