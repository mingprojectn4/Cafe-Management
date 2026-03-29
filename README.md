# Cafe Management System

A web-based cafe management system built with Laravel, featuring role-based access for Admin, Barista, and Customers.

## Features

### Admin
- Dashboard with statistics (orders, revenue, products, users)
- Order management - view all orders
- Product management - add, edit, delete products
- Staff management - manage baristas and admins
- Customer list view

### Barista
- View active orders (pending & preparing)
- Update order status (pending → preparing → completed)
- Order history
- Auto-refresh every 30 seconds

### User (Customer)
- Browse menu by category
- Shopping cart functionality
- Place orders (dine-in or takeaway)
- Add special instructions
- View order history
- Cancel pending orders

## Technology Stack

- **Backend:** Laravel 13
- **Database:** SQLite
- **Frontend:** Bootstrap 5, Blade Templates
- **API:** Laravel Sanctum

## Installation

### Prerequisites
- PHP 8.2+
- Composer
- SQLite

### Setup

1. Navigate to the project directory:
```bash
cd "/home/meng/Web Final (Cafe/cafe-app"
```

2. Install dependencies (already done):
```bash
composer install
```

3. Set up environment:
```bash
cp .env.example .env
php artisan key:generate
```

4. Run migrations:
```bash
php artisan migrate:fresh --seed
```

5. Create storage link:
```bash
php artisan storage:link
```

6. Start the development server:
```bash
php artisan serve
```

7. Access the application at: http://localhost:8000

## Demo Credentials

| Role    | Email            | Password |
|---------|------------------|----------|
| Admin   | admin@cafe.com   | password |
| Barista | barista@cafe.com | password |
| User    | user@cafe.com    | password |

## Database Structure

### Users
- id, name, email, password, role (admin/barista/user)

### Products
- id, name, description, price, category, image, available

### Orders
- id, user_id, status, total_price, order_type, notes, completed_at

### Order Items
- id, order_id, product_id, quantity, price, special_instructions

## API Endpoints

### Authentication
- `POST /api/login` - User login
- `POST /api/logout` - User logout
- `GET /api/user` - Get current user

### Products
- `GET /api/products` - Get available products
- `GET /api/products/all` - Get all products (admin)
- `POST /api/products` - Create product (admin)
- `PUT /api/products/{id}` - Update product (admin)
- `DELETE /api/products/{id}` - Delete product (admin)

### Orders
- `GET /api/orders` - Get all orders (admin/barista)
- `GET /api/orders/my-orders` - Get user's orders
- `GET /api/orders/{id}` - Get order details
- `POST /api/orders` - Create new order
- `PUT /api/orders/{id}/status` - Update order status (barista/admin)
- `POST /api/orders/{id}/cancel` - Cancel order

### Staff Management
- `GET /api/staff` - Get staff list
- `GET /api/staff/all` - Get all users
- `POST /api/staff` - Create staff (admin)
- `PUT /api/staff/{id}` - Update staff (admin)
- `DELETE /api/staff/{id}` - Delete staff (admin)

## Project Structure

```
cafe-app/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── AuthController.php
│   │   │   │   ├── ProductController.php
│   │   │   │   ├── OrderController.php
│   │   │   │   └── StaffController.php
│   │   │   ├── AuthController.php
│   │   │   ├── AdminController.php
│   │   │   ├── BaristaController.php
│   │   │   └── UserController.php
│   │   └── Middleware/
│   │       └── AdminMiddleware.php
│   └── Models/
│       ├── User.php
│       ├── Product.php
│       ├── Order.php
│       └── OrderItem.php
├── database/
│   ├── migrations/
│   └── seeders/
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
└── .env
```

## Order Status Flow

```
pending → preparing → completed
   ↓
cancelled
```

## Sample Data

The seeder creates:
- 5 users (1 admin, 1 barista, 3 customers)
- 17 products (6 coffee, 4 tea, 4 pastry, 3 snacks)
- 4 sample orders with different statuses

## License

This project is created for educational purposes.
