# Cafe Management System - Presentation Slides

**Copy and paste this entire document to your AI slide generator**

---

## SLIDE 1: Title Slide

```
Title: Cafe Management System
Subtitle: A Laravel 13 Web Application
Presented by: [Your Name]
Date: March 26, 2026
Project: Laravel Cafe Management System v1.0
```

**Visual suggestion:** Cafe/coffee shop background image with Laravel logo

---

## SLIDE 2: Project Overview

```
Title: What is This Project?

Content:
• A complete web-based Cafe Management System
• Built with Laravel 13 (PHP 8.3)
• 3 User Roles: Admin, Barista, Customer
• Manages products, orders, staff, and customers
• Responsive UI with Bootstrap 5.3
• RESTful API for future mobile apps

Key Features:
✓ User authentication with role-based access
✓ Product catalog management
✓ Order placement and tracking
✓ Real-time order status updates
✓ Staff and customer management
✓ SQLite database (easily switchable to MySQL)
```

**Visual suggestion:** System architecture diagram or feature icons

---

## SLIDE 3: Technology Stack

```
Title: Technology Stack

Backend:
• Framework: Laravel 13.x
• Language: PHP 8.3+
• Database: SQLite (local development)
• API Authentication: Laravel Sanctum

Frontend:
• CSS Framework: Bootstrap 5.3
• Icons: Bootstrap Icons 1.11.0
• Fonts: Plus Jakarta Sans, Inter
• Templating: Blade (server-side rendering)

Development Tools:
• PHPUnit (testing)
• Laravel Pint (code formatting)
• Faker (sample data)
• Composer (dependency management)
```

**Visual suggestion:** Logos of Laravel, PHP, Bootstrap, SQLite arranged in a grid

---

## SLIDE 4: Project Structure

```
Title: Project Structure

cafe-app/
├── app/
│   ├── Http/Controllers/    # Request handlers
│   │   ├── AdminController.php
│   │   ├── BaristaController.php
│   │   ├── UserController.php
│   │   └── Api/             # API controllers
│   └── Models/              # Database models
│       ├── User.php
│       ├── Product.php
│       ├── Order.php
│       └── OrderItem.php
├── routes/
│   ├── web.php              # Web routes
│   └── api.php              # API routes
├── resources/views/         # Blade templates
│   ├── admin/
│   ├── barista/
│   ├── user/
│   └── layouts/
├── database/
│   ├── migrations/          # Table structures
│   └── seeders/             # Sample data
└── .env                     # Configuration
```

**Visual suggestion:** Folder tree diagram with highlighted key directories

---

## SLIDE 5: Database Schema

```
Title: Database Design

Users Table:
• id, name, email, password, role
• role: admin | barista | user

Products Table:
• id, name, description, price
• category, image, available (boolean)

Orders Table:
• id, user_id (FK), status, total_price
• order_type (dine-in/takeaway), notes
• created_at, completed_at

Order Items Table:
• id, order_id (FK), product_id (FK)
• quantity, price, special_instructions

Relationships:
• User → has many → Orders
• Order → has many → OrderItems
• Product → has many → OrderItems
```

**Visual suggestion:** ERD (Entity Relationship Diagram) showing 4 tables with connections

---

## SLIDE 6: User Roles Overview

```
Title: Three User Roles

┌─────────────┬─────────────┬─────────────┐
│   ADMIN     │  BARISTA    │   USER      │
│             │             │ (Customer)  │
├─────────────┼─────────────┼─────────────┤
│ ✓ Dashboard │ ✓ Order     │ ✓ Browse    │
│ ✓ Products  │   Queue     │   Menu      │
│ ✓ All Orders│ ✓ Update    │ ✓ Place     │
│ ✓ Staff Mgt │   Status    │   Orders    │
│ ✓ Customers │ ✓ History   │ ✓ My Orders │
│             │             │ ✓ Track     │
└─────────────┴─────────────┴─────────────┘

Each role has dedicated pages and permissions
```

**Visual suggestion:** 3-column layout with icons for each role

---

## SLIDE 7: Login System

```
Title: Authentication & Role-Based Routing

Login Flow:
1. User enters credentials at /login
2. Form submits to AuthController::login()
3. Validates against database
4. Creates session
5. Checks user role
6. Redirects to role-specific dashboard

Default Credentials:
┌──────────────┬───────────────────┬──────────┐
│ Role         │ Email             │ Password │
├──────────────┼───────────────────┼──────────┤
│ Admin        │ admin@cafe.com    │ password │
│ Barista      │ barista@cafe.com  │ password │
│ User         │ user@cafe.com     │ password │
└──────────────┴───────────────────┴──────────┘

Security:
• Password hashing (bcrypt)
• CSRF token protection
• Session-based authentication
• Middleware role checks
```

**Visual suggestion:** Login flow diagram with arrows

---

## SLIDE 8: Admin Features

```
Title: Admin Role Capabilities

Dashboard (/admin/dashboard):
• Total orders, revenue, pending orders
• Staff and customer counts
• Visual statistics with color-coded cards

Products Management (/admin/products):
• Add/Edit/Delete menu items
• Upload product images
• Set availability status
• Grid view (3-4 items per row)

Orders Management (/admin/orders):
• View all customer orders
• Filter by status and date
• View order details in modals
• Card-based layout

Staff Management (/admin/staff):
• Create admin/barista accounts
• Edit staff information
• Delete staff accounts

Customers Management (/admin/customers):
• View all registered customers
• See customer order history
```

**Visual suggestion:** Screenshots of admin dashboard and products page

---

## SLIDE 9: Barista Features

```
Title: Barista Role Capabilities

Order Queue (/barista/orders):
• View pending and preparing orders
• Real-time order status updates
• Customer information display
• Order items with quantities
• Special instructions visible

Status Management:
• Pending → Preparing (Click "Prepare")
• Preparing → Completed (Click "Complete")
• Color-coded status badges:
  - Yellow: Pending
  - Blue: Preparing
  - Green: Completed
  - Red: Cancelled

Order History (/barista/history):
• View completed orders
• View cancelled orders
• Filter by date range
```

**Visual suggestion:** Barista order queue screenshot with status flow diagram

---

## SLIDE 10: Customer Features

```
Title: Customer (User) Capabilities

Browse Menu (/user/orders):
• View all available products
• Category-based organization
• Category pills for quick navigation
• Product cards with images
• Price and description display

Shopping Cart:
• Slide-out cart panel
• Add/remove items
• Adjust quantities
• Special instructions field
• Order type selection (Dine-in/Takeaway)

Place Order:
• One-click order placement
• Order notes support
• Success modal with Order ID
• Automatic cart clearing

My Orders (/user/my-orders):
• Order history with status
• Order details modal
• Track order progress
• Reorder capability
```

**Visual suggestion:** Menu browsing and cart screenshots

---

## SLIDE 11: Order Flow

```
Title: Complete Order Lifecycle

Step-by-Step Flow:

1. CUSTOMER places order
   Status: PENDING
   ↓
2. BARISTA sees in queue
   Action: Click "Prepare"
   Status: PREPARING
   ↓
3. BARISTA marks ready
   Action: Click "Complete"
   Status: COMPLETED
   ↓
4. Order archived in history

ADMIN can view all statuses anytime
Filter by: pending, preparing, completed, cancelled

Database Updates:
• orders.status field updated
• orders.completed_at timestamp set
• Real-time reflection in all views
```

**Visual suggestion:** Flowchart with 4 steps and arrows

---

## SLIDE 12: Role Protection (Middleware)

```
Title: Security & Access Control

Middleware Protection:

Admin Routes:
Route::middleware(['auth', 'admin'])->group(...)
• Checks: Is logged in? Is admin?
• Denies: 403 error if not admin

Barista Routes:
Route::middleware(['auth', 'barista'])->group(...)
• Checks: Is logged in? Is barista OR admin?
• Allows: Barista and Admin access

User Routes:
Route::middleware(['auth'])->group(...)
• Checks: Is logged in?
• Allows: Any authenticated user

Code Example:
if (!auth()->check() || !auth()->user()->isAdmin()) {
    abort(403, 'Unauthorized access');
}
```

**Visual suggestion:** Shield icon with middleware flow diagram

---

## SLIDE 13: API Endpoints

```
Title: RESTful API (Ready for Mobile Apps)

Public Endpoints:
• GET  /api/products         - Get available products
• POST /api/login            - User login

Protected Endpoints (Sanctum Auth):
• GET  /api/user             - Get current user
• POST /api/logout           - Logout user
• GET  /api/products/all     - All products (admin)
• POST /api/products         - Create product (admin)
• PUT  /api/products/{id}    - Update product (admin)
• DELETE /api/products/{id}  - Delete product (admin)
• GET  /api/orders           - Get orders
• POST /api/orders           - Create order
• PUT  /api/orders/{id}/status - Update status

API Response Format:
{
  "success": true,
  "data": {...},
  "message": "..."
}
```

**Visual suggestion:** API request/response diagram with JSON example

---

## SLIDE 14: Frontend Design

```
Title: UI/UX Design Features

Color Palette:
• Primary: #E8A4A8 (Light Pink)
• Dark: #C97B7B (Dark Pink)
• Light: #F5C4C8 (Background)
• Accent: #FF8FA3 (Accent Pink)
• Sidebar: #F8D7DA (Soft Pink)
• Text: #FFF0F5 (Light text on sidebar)

Layout:
• Fixed left sidebar (260px)
• White main content area
• No top navbar on admin pages
• Responsive grid system
• Card-based design

Components:
• Rounded corners (16px)
• Soft shadows
• Gradient stat cards
• Modal dialogs
• Slide-out cart panel
• Category pills
• Status badges
```

**Visual suggestion:** Color palette swatches and layout wireframe

---

## SLIDE 15: Code Flow Example

```
Title: How It Works - Login Flow

1. Browser → GET /login
   routes/web.php → returns view('auth.login')

2. User submits form → POST /login
   AuthController::login()

3. Controller validates:
   $credentials = $request->validate([...])

4. Check database:
   Auth::attempt($credentials)
   SQL: SELECT * FROM users WHERE email = ?

5. Check role:
   if ($user->isAdmin()) → redirect to /admin/dashboard
   if ($user->isBarista()) → redirect to /barista/orders
   else → redirect to /user/orders

6. Session created, user authenticated
```

**Visual suggestion:** Numbered flowchart with icons for each step

---

## SLIDE 16: Database Queries

```
Title: Eloquent ORM Examples

Get Available Products:
$products = Product::where('available', true)
    ->orderBy('category')
    ->get();

SQL: SELECT * FROM products 
     WHERE available = 1 
     ORDER BY category

Create Order:
$order = Order::create([
    'user_id' => auth()->id(),
    'status' => 'pending',
    'total_price' => 12.50,
    'order_type' => 'dine-in',
]);

SQL: INSERT INTO orders (user_id, status, total_price, ...)
     VALUES (3, 'pending', 12.50, 'dine-in', ...)

Get Orders by Status:
$orders = Order::where('status', 'pending')->get();

SQL: SELECT * FROM orders WHERE status = 'pending'
```

**Visual suggestion:** Code snippets with SQL output side by side

---

## SLIDE 17: File Upload Handling

```
Title: Product Image Upload

Storage Configuration:
• Images stored in: storage/app/public/products
• Symlink: public/storage → storage/app/public
• Command: php artisan storage:link

Upload Code (AdminController):
if ($request->hasFile('image')) {
    $path = $request->file('image')
        ->store('products', 'public');
    $validated['image'] = $path;
}

Display in Blade:
<img src="{{ asset('storage/' . $product->image) }}"
     alt="{{ $product->name }}">

Supported Formats: JPG, PNG, GIF, WEBP
Max Size: 2MB
```

**Visual suggestion:** Upload flow diagram with folder structure

---

## SLIDE 18: Commands Reference

```
Title: Useful Artisan Commands

Start Development:
php artisan serve
→ http://127.0.0.1:8000

Reset Database:
php artisan migrate:fresh --seed
→ Creates tables + sample data

Clear Cache:
php artisan config:clear
php artisan cache:clear
php artisan view:clear

Create Storage Link:
php artisan storage:link
→ Enables product images

View Routes:
php artisan route:list
→ Shows all 45+ routes

Run Tests:
php artisan test
→ PHPUnit test suite
```

**Visual suggestion:** Terminal screenshot with commands

---

## SLIDE 19: Testing the Application

```
Title: How to Test All Features

1. Start Server:
   php artisan serve

2. Test Admin:
   Login: admin@cafe.com / password
   → Add a product
   → View orders
   → Manage staff

3. Test Barista:
   Login: barista@cafe.com / password
   → Update order status
   → View history

4. Test Customer:
   Login: user@cafe.com / password
   → Browse menu
   → Add to cart
   → Place order
   → View my orders

5. Test API (Postman):
   GET http://127.0.0.1:8000/api/products
   POST http://127.0.0.1:8000/api/login
```

**Visual suggestion:** Checklist with 5 testing scenarios

---

## SLIDE 20: Project Statistics

```
Title: Project Statistics

Code Metrics:
• Total Routes: 45+
• Controllers: 8 (including API)
• Models: 4 (User, Product, Order, OrderItem)
• Views: 15+ Blade templates
• Migrations: 8 database tables
• API Endpoints: 19

Sample Data (Seeded):
• Users: 5 (1 admin, 1 barista, 3 customers)
• Products: 17 (coffee, tea, pastry, snacks)
• Orders: 6 (various statuses)

Files Modified:
• Core controllers: 4
• API controllers: 4
• Views: 11
• Layout: 1 (shared)
• Middleware: 2
```

**Visual suggestion:** Infographic with numbers and icons

---

## SLIDE 21: Key Learnings

```
Title: What I Learned

Laravel Framework:
• MVC architecture
• Eloquent ORM
• Blade templating
• Middleware & authentication
• Routing & controllers
• Database migrations
• Seeders & factories

Web Development:
• Role-based access control
• CRUD operations
• File upload handling
• Form validation
• Session management
• RESTful API design
• Responsive UI design

Soft Skills:
• Problem-solving
• Debugging
• Documentation
• Project organization
```

**Visual suggestion:** Lightbulb or brain icon with skill categories

---

## SLIDE 22: Future Enhancements

```
Title: Possible Future Features

Payment Integration:
• Stripe/PayPal payment gateway
• Online payment before order
• Payment history

Real-time Features:
• WebSocket for live order updates
• Push notifications
• Kitchen display system

Advanced Features:
• Inventory management
• Low stock alerts
• Sales analytics & reports
• Email notifications
• SMS order confirmations
• Customer loyalty program
• Multi-location support

Mobile App:
• React Native app using existing API
• QR code ordering
• Order ahead feature
```

**Visual suggestion:** Roadmap timeline with future features

---

## SLIDE 23: Challenges & Solutions

```
Title: Challenges Faced & Solutions

Challenge 1: Role-based redirection
Solution: Check role in controller, redirect accordingly

Challenge 2: Order status updates
Solution: Barista controller with status change methods

Challenge 3: Cart management
Solution: JavaScript array + AJAX submission

Challenge 4: Product image uploads
Solution: Laravel storage system + symlink

Challenge 5: Consistent design across roles
Solution: Shared layout file with CSS variables

Challenge 6: API authentication
Solution: Laravel Sanctum token-based auth

Challenge 7: Pagination styling
Solution: Custom CSS for Bootstrap pagination
```

**Visual suggestion:** Problem → Solution arrow diagrams

---

## SLIDE 24: Conclusion

```
Title: Conclusion

Summary:
✓ Complete cafe management system
✓ 3 distinct user roles with proper access control
✓ Full CRUD for products, orders, staff
✓ Real-time order status tracking
✓ RESTful API ready for mobile integration
✓ Clean, responsive UI design
✓ Secure authentication & authorization

Technologies Used:
• Laravel 13 + PHP 8.3
• Bootstrap 5.3
• SQLite Database
• Laravel Sanctum (API auth)

Project Status: COMPLETE ✅

Thank You!
Questions?
```

**Visual suggestion:** "Thank You" with project screenshot collage

---

## SLIDE 25: Q&A

```
Title: Questions & Answers

Common Questions:

Q: Why SQLite instead of MySQL?
A: Lightweight for development, easily switchable via .env file

Q: How does role protection work?
A: Middleware checks user role before allowing access

Q: Can you add payment gateway?
A: Yes, architecture supports Stripe/PayPal integration

Q: Is the API secure?
A: Yes, uses Laravel Sanctum token authentication

Q: How do you handle file uploads?
A: Laravel storage system saves to storage/app/public

Q: Can this scale to multiple cafes?
A: Yes, with database changes and multi-tenancy support

Contact:
[Your Email]
[Your GitHub/Portfolio]
```

**Visual suggestion:** Q&A icon with contact information

---

# END OF SLIDES

**Total Slides:** 25
**Estimated Presentation Time:** 15-20 minutes
**Recommended Pace:** 30-45 seconds per slide

---

## Instructions for AI Slide Generator:

```
1. Create one slide per section above
2. Use the "Title" as slide title
3. Use "Content" bullet points as slide body
4. Follow "Visual suggestion" for images/diagrams
5. Maintain consistent color scheme (pink theme from project)
6. Use modern, clean design
7. Include code snippets where shown
8. Add transitions between slides
9. Export as PowerPoint or Google Slides format
```

**Color Scheme for Slides:**
- Primary: #E8A4A8 (Light Pink)
- Secondary: #C97B7B (Dark Pink)
- Background: White or very light pink (#FDECEF)
- Text: Dark gray (#4A3737)
- Accents: #FF8FA3

---

**Document Created:** March 26, 2026
**For:** Cafe Management System Presentation
**Framework:** Laravel 13.x
