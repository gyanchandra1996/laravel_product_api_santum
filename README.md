Got it — you want the **same Product API using Sanctum but specifically in Laravel 12**. Good news: Laravel 12 setup is cleaner, but a few things have changed slightly (especially middleware handling).

Let’s do it **step-by-step (Laravel 12 updated way)** 👇

---

# 🚀 Step 1: Create Laravel 12 Project

```bash
composer create-project laravel/laravel product-api
cd product-api
```

Check version:

```bash
php artisan --version
```

---

# 🚀 Step 2: Install Sanctum

```bash
composer require laravel/sanctum
```

Publish & migrate:

```bash
php artisan vendor:publish --tag=sanctum-config
php artisan migrate
```

---

# 🚀 Step 3: Configure Sanctum (Laravel 12 way)

👉 Open `bootstrap/app.php`

Add this inside middleware:

```php
->withMiddleware(function ($middleware) {
    $middleware->statefulApi();
})
```

⚠️ Important:
Laravel 12 **does NOT use Kernel.php like old versions** for API config.

---

# 🚀 Step 4: Update User Model

`app/Models/User.php`

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens;
}
```

---

# 🚀 Step 5: Create Product Model + Migration

```bash
php artisan make:model Product -m
```

Migration:

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description')->nullable();
    $table->decimal('price', 10, 2);
    $table->integer('quantity');
    $table->timestamps();
});
```

Run:

```bash
php artisan migrate
```

---

# 🚀 Step 6: Create Controllers

```bash
php artisan make:controller API/AuthController
php artisan make:controller API/ProductController --api
```

---

# 🚀 Step 7: API Routes (Laravel 12)

👉 `routes/api.php`

```php
use App\Http\Controllers\API\AuthController;
use App\Http\Controllers\API\ProductController;

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('products', ProductController::class);
    Route::post('/logout', [AuthController::class, 'logout']);
});
```

---

# 🚀 Step 8: Auth Controller

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $request->validate([
            'name'=>'required',
            'email'=>'required|email|unique:users',
            'password'=>'required|min:6'
        ]);

        $user = User::create([
            'name'=>$request->name,
            'email'=>$request->email,
            'password'=>Hash::make($request->password),
        ]);

        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json(['token'=>$token]);
    }

    public function login(Request $request)
    {
        $user = User::where('email',$request->email)->first();

        if(!$user || !Hash::check($request->password,$user->password)){
            return response()->json(['message'=>'Invalid credentials'],401);
        }

        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json(['token'=>$token]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message'=>'Logged out']);
    }
}
```

---

# 🚀 Step 9: Product Controller

```php
use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index()
    {
        return Product::latest()->get();
    }

    public function store(Request $request)
    {
        $request->validate([
            'name'=>'required',
            'price'=>'required|numeric',
            'quantity'=>'required|integer'
        ]);

        $product = Product::create([
            'name'=>$request->name,
            'description'=>$request->description,
            'price'=>$request->price,
            'quantity'=>$request->quantity
        ]);

        return response()->json($product,201);
    }

    public function show($id)
    {
        return Product::findOrFail($id);
    }

    public function update(Request $request, $id)
    {
        $product = Product::findOrFail($id);

        $product->update($request->only([
            'name','description','price','quantity'
        ]));

        return response()->json($product);
    }

    public function destroy($id)
    {
        Product::findOrFail($id)->delete();

        return response()->json(['message'=>'Deleted']);
    }
}
```

---

# 🚀 Step 10: Product Model

```php
protected $fillable = [
    'name','description','price','quantity'
];
```

---

# 🚀 Step 11: Test in Postman

### 🔐 Login / Register → Get Token

Then use:

```
Authorization: Bearer YOUR_TOKEN
```

---

# 🔥 Laravel 12 Important Differences

| Feature       | Old Laravel       | Laravel 12                   |
| ------------- | ----------------- | ---------------------------- |
| Middleware    | Kernel.php        | bootstrap/app.php            |
| Sanctum Setup | manual config     | `$middleware->statefulApi()` |
| Token delete  | `$user->tokens()` | `currentAccessToken()`       |

---

# 🎯 Best Practice (Very Important)

Don’t stop here — real projects need:

### ✅ Use Form Request

```bash
php artisan make:request StoreProductRequest
```

### ✅ Use API Resource

```bash
php artisan make:resource ProductResource
```

### ✅ Pagination

```php
Product::paginate(10);
```

### ✅ Clean Architecture

* Controller → Service → Repository (you already started learning this 👍)

---

# 🚀 Next Step (I Recommend)

If you’re serious about backend:

* 👉 Add **Product Image Upload (API)**
* 👉 Add **Search + Filter API**
* 👉 Add **JWT vs Sanctum difference**
* 👉 Build **Full E-commerce API (Cart, Order, Payment)**

---

If you want, I can give you a **complete production-level Laravel 12 API structure** (like companies use).


Alright — now I’ll guide you **properly step-by-step** on how to **test your Laravel 12 API using Postman / Thunder Client**, and I’ll include **sample requests + responses** so you can verify everything clearly.

---

# 🚀 1. Start Your Laravel Server

```bash
php artisan serve
```

👉 Base URL:

```
http://127.0.0.1:8000
```

---

# 🚀 2. Check API Routes (VERY IMPORTANT)

Run:

```bash
php artisan route:list
```

👉 You should see:

```
POST   api/register
POST   api/login
GET    api/products
```

If not → API not configured (we already fixed earlier)

---

# 🚀 3. Test API using Postman / Thunder Client

---

# 🔐 STEP 1: Register User

## 👉 Request

* Method: **POST**
* URL:

```
http://127.0.0.1:8000/api/register
```

## 👉 Headers:

```json
Content-Type: application/json
Accept: application/json
```

## 👉 Body (raw JSON):

```json
{
  "name": "Gyan",
  "email": "gyan1@example.com",
  "password": "123456"
}
```

---

## ✅ Response:

```json
{
  "token": "1|abcxyz..."
}
```

👉 Copy this token 🔑

---

# 🔐 STEP 2: Login User

## 👉 Request

```
POST /api/login
```

## 👉 Body:

```json
{
  "email": "gyan1@example.com",
  "password": "123456"
}
```

---

## ✅ Response:

```json
{
  "token": "1|newtoken..."
}
```

---

# 🔐 STEP 3: Use Token (VERY IMPORTANT)

👉 For all protected routes:

## 👉 Header:

```text
Authorization: Bearer YOUR_TOKEN
Accept: application/json
```

---

# 📦 STEP 4: Create Product

## 👉 Request

* Method: **POST**

```
http://127.0.0.1:8000/api/products
```

## 👉 Headers:

```text
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json
```

## 👉 Body:

```json
{
  "name": "Laptop",
  "description": "Dell i5",
  "price": 50000,
  "quantity": 10
}
```

---

## ✅ Response:

```json
{
  "id": 1,
  "name": "Laptop",
  "description": "Dell i5",
  "price": "50000.00",
  "quantity": 10
}
```

---

# 📦 STEP 5: Get All Products

## 👉 Request

```
GET /api/products
```

## 👉 Headers:

```text
Authorization: Bearer YOUR_TOKEN
```

---

## ✅ Response:

```json
[
  {
    "id": 1,
    "name": "Laptop",
    "price": "50000.00"
  }
]
```

---

# 📦 STEP 6: Update Product

## 👉 Request

```
PUT /api/products/1
```

## 👉 Body:

```json
{
  "name": "Laptop Updated",
  "price": 55000,
  "quantity": 8
}
```

---

# 📦 STEP 7: Delete Product

## 👉 Request

```
DELETE /api/products/1
```

---

# 🚀 Thunder Client (VS Code) Steps

1. Open Thunder Client
2. Click **New Request**
3. Enter URL
4. Select Method (POST/GET)
5. Go to **Body → JSON**
6. Add headers in **Headers tab**
7. Click **Send**

---

# 🔥 Common Mistakes (VERY IMPORTANT)

❌ Forgot `/api/` prefix
❌ Token not added
❌ Wrong method (GET instead of POST)
❌ JSON not selected in body
❌ Server not running

---

# 🎯 Quick Testing Flow

1. Register → get token
2. Login → get token
3. Add token → test products API

---

# 🚀 Pro Developer Tips

* Use **Postman Collection** (save all APIs)
* Use **environment variables**:

  ```
  {{base_url}}/api/register
  ```
* Always send:

  ```
  Accept: application/json
  ```

---



