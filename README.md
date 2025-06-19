# 🍊 FruitOrder - Ứng dụng web bán trái cây trực tuyến

## 👨‍🎓 Thông tin sinh viên

* Họ tên: \[Đoàn Văn Khánh]
* Mã sinh viên: \[23010252]

---

## 📌 Mô tả dự án

Dự án xây dựng một website bán trái cây trực tuyến gồm hai vai trò: Admin (người quản lý) và User (khách hàng).

Chức năng của hệ thống:

* Quản lý sản phẩm (Admin CRUD)
* Quản lý đơn hàng (Admin CRUD)
* Quản lý khách hàng
* User đăng ký, đăng nhập, xem sản phẩm, đặt hàng

---

## 👥 Phân quyền hệ thống

### 1. 👤 **User (Khách hàng)**

* Đăng ký, đăng nhập
* Xem danh sách sản phẩm
* Thêm sản phẩm vào giỏ
* Đặt đơn hàng

### 2. 🛠 **Admin (Quản trị viên)**

* CRUD sản phẩm
* CRUD đơn hàng
* Quản lý khách hàng
* Chỉ Admin mới truy cập được trang quản trị

---

## 📚 Các đối tượng chính (Model)

* `Product`: tên, loại, giá, mô tả
* `User`: tên, email, số điện thoại
* `Order`: ngày đặt, thuộc về user, chứa nhiều product (n-n)

---

## 🔐 Bảo mật

* Xác thực qua Laravel Breeze
* Middleware `AdminMiddleware` kiểm tra vai trò
* CSRF Token, XSS escape, Validation input
* Session và cookie được bảo vệ

---

## 🌐 Kết nối dữ liệu Cloud

* Dùng Eloquent ORM
* Dùng file `.env` kết nối database Aiven PostgreSQL
* Migration, Seed dữ liệu thông qua `php artisan migrate`

---

## 🧠 Sơ đồ Class Diagram
![image](https://github.com/user-attachments/assets/8a233be5-312e-4af0-b1fa-6e648ede8480)


---

## ⚖️ Sơ đồ thuật toán (Activity Diagram)
## 1. Tìm kiếm gần đúng tên sản phẩm
🟢 Start ⬇️  
👤 User nhập từ khóa vào thanh tìm kiếm ⬇️  
🔍 Hệ thống thực hiện fuzzy search (LIKE, Scout, Fulltext...) ⬇️  
📦 Truy vấn các sản phẩm có tên gần giống từ khóa ⬇️  
🖥️ Hiển thị kết quả tìm kiếm ⬇️  
🔴 End




## 2. Hiển thị đơn hàng theo khách hàng

🟢 Start<br>
   ↓<br>
🔐 Admin đăng nhập<br>
   ↓<br>
🧍‍♂️ Chọn khách hàng cần xem đơn hàng<br>
   ↓<br>
📦 Truy vấn đơn hàng theo customer_id<br>
   ↓<br>
📋 Hiển thị danh sách đơn hàng tương ứng<br>
   ↓<br>
🔴 End


## 📷 Ảnh màn hình giao diện

* Giao diện trang chủ, đăng nhập
* ![image](https://github.com/user-attachments/assets/ded61f34-236b-4476-a6c2-0e33f7bf7429)

* Danh sách sản phẩm
* ![image](https://github.com/user-attachments/assets/8b55e729-2c2b-4802-898d-df5395c70a1f)

* Đặt hàng
* ![image](https://github.com/user-attachments/assets/ec871aa9-159d-4c4c-a0cf-cf6d49bb9ec1)

* Trang quản lý admin
* ![image](https://github.com/user-attachments/assets/3066dd28-de5d-4a35-a911-a924efd9d61e)
* ![image](https://github.com/user-attachments/assets/72c13138-24dc-446d-bd1e-91cafefeb280)



---

## 💻 Code minh hoạ

### Controller `ProductController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;
use App\Models\Category;
use Illuminate\Support\Facades\Storage;

class ProductController extends Controller
{
    // 1️⃣ Hiển thị list sản phẩm (trang chủ)
    public function index(Request $request)
    {
        // build query
        $query = Product::with('category');

        // filter theo category nếu URL có ?category=slug
        if ($slug = $request->query('category')) {
            $query->whereHas('category', fn($q) => $q->where('slug', $slug));
        }

        // phân trang 12/sp/trang
        $products = $query->orderBy('created_at','desc')->paginate(12);

        // trả view resources/views/products/index.blade.php
        return view('products.index', compact('products'));
    }
    public function importedFruits()
    {
        // Lấy danh sách sản phẩm nhập khẩu
        $products = Product:: where('category_id', 1)->paginate(12);

        // Trả về view với danh sách sản phẩm
        return view('products.imported', compact('products'));

    }
    public function localFruits()
    {
        // Lấy danh sách sản phẩm Việt Nam
        $products = Product::where('category_id', 2)->paginate(12);

        // Trả về view với danh sách sản phẩm
        return view('products.local', compact('products'));
    }
    public function search(Request $request)
    {
        // Lấy từ khóa tìm kiếm từ yêu cầu
        $searchTerm = $request->input('search');

        // Kiểm tra nếu từ khóa tìm kiếm có ít nhất 3 ký tự
        if (strlen($searchTerm) >= 3) {
            // Tìm kiếm sản phẩm có tên gần giống với từ khóa, sử dụng 'like' để tìm kiếm gần đúng
            $products = Product::where('name', 'like', '%' . $searchTerm . '%')
                               ->paginate(12);  // Lấy tất cả sản phẩm có tên gần giống

            return view('products.search', compact('products', 'searchTerm'));  // Trả về view với danh sách sản phẩm
        } else {
            return redirect()->route('home')->with('error', 'Từ khóa tìm kiếm quá ngắn.');
        }
    }
    public function promotion()
    {
        $products = Product::where('category_id', 1)->paginate(12);
        // Trả về view với danh sách sản,  phẩm khuyến mãi
        return view('products.promotion', compact('products'));
    }

    // 2️⃣ Hiển thị form tạo (chỉ admin, sau này)
    public function create()
    {
        // TODO: load category list, trả view form admin
        // Lấy danh sách các category để chọn khi tạo sản phẩm mới
        $categories = Category::all();

        // Trả về view form tạo sản phẩm với danh sách category
        return view('admin.products.create', compact('categories'));
    }

    // 3️⃣ Lưu product mới (chỉ admin)
    public function store(Request $request)
    {
        // TODO: validate + lưu vào DB
         // Validate dữ liệu đầu vào
        $request->validate([
            'name' => 'required|string|max:255',
            'slug' => 'required|string|max:255|unique:products,slug',
            'category_id' => 'required|exists:categories,id',
            'price' => 'required|numeric',
            'description' => 'nullable|string',
            'image' => 'nullable|image|mimes:jpeg,png,jpg,gif|max:4096', // Giới hạn kích thước ảnh tối đa 4MB
        ]);

        // Lưu thông tin sản phẩm vào DB
        $product = new Product();
        $product->name = $request->name;
        $product->slug = $request->slug;
        $product->category_id = $request->category_id;
        $product->price = $request->price;
        $product->description = $request->description;

        // Xử lý ảnh nếu có
        if ($request->hasFile('image')) {
            // Tạo tên ảnh duy nhất
            $imageName = time() . '-' . $request->file('image')->getClientOriginalName();
            
            // Lưu ảnh vào thư mục 'public/products' trong 'storage/app'
            $imagePath = $request->file('image')->storeAs('products', $imageName);
            
            // Lưu đường dẫn ảnh vào cơ sở dữ liệu
            $product->image = 'products/' . $imageName; // Lưu đường dẫn tương đối
        }
        $product->save();

        // Redirect về trang danh sách sản phẩm
        return redirect()->route('admin.products.index')->with('success', 'Sản phẩm đã được tạo thành công!');
    }

    // 4️⃣ Hiển thị chi tiết 1 sản phẩm
    public function show($slug)
    {
        $product = Product::with('category')->where('slug', $slug)->firstOrFail();
        return view('products.show', compact('product'));
    }

    // 5️⃣ Hiển thị form edit (chỉ admin)
    public function edit($id)
    {
        // TODO: load product + categories, view form
        // Lấy thông tin sản phẩm và danh sách category để chỉnh sửa
        $product = Product::findOrFail($id);
        $categories = Category::all();

        return view('admin.products.edit', compact('product', 'categories'));
    }

    // 6️⃣ Cập nhật product (chỉ admin)
    public function update(Request $request, $id)
    {
        // TODO: validate + update DB
        // Validate dữ liệu đầu vào
        $request->validate([
            'name' => 'required|string|max:255',
            'slug' => 'required|string|max:255|unique:products,slug,' . $id,
            'category_id' => 'required|exists:categories,id',
            'price' => 'required|numeric',
            'description' => 'nullable|string',
            'image' => 'nullable|image|mimes:jpeg,png,jpg,gif|max:2048',
        ]);

        // Tìm sản phẩm cần cập nhật
        $product = Product::findOrFail($id);
        $product->name = $request->name;
        $product->slug = $request->slug;
        $product->category_id = $request->category_id;
        $product->price = $request->price;
        $product->description = $request->description;

        // Xử lý ảnh nếu có
        if ($request->hasFile('image')) {
            // Xóa ảnh cũ nếu có
            if ($product->image) {
                unlink(storage_path('app/' . $product->image));
            }
            $imagePath = $request->file('image')->store('public/products');
            $product->image = $imagePath;
        }

        // Lưu sản phẩm
        $product->save();

        // Redirect về trang chi tiết sản phẩm
        return redirect()->route('admin.products.show', $product->slug)->with('success', 'Sản phẩm đã được cập nhật thành công!');
    }

    // 7️⃣ Xóa product (chỉ admin)
    public function destroy($id)
    {
        // TODO: delete and redirect
        // Redirect về trang danh sách sản phẩm
          $product = Product::findOrFail($id);

        // Xóa ảnh nếu có
        if ($product->image) {
        // Xóa file ảnh khỏi thư mục lưu trữ
        Storage::delete($product->image);
        }
        $product->delete();
        return redirect()->route('admin.products.index')->with('success', 'Sản phẩm đã được xóa thành công!');
    }
}

```

### Model `Product.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory; 

class Product extends Model
{
    use HasFactory;
    public function category()
    {
        return $this->belongsTo(Category::class);
    }
    public function carts()
    {
        return $this->hasMany(Cart::class);
    }
    public function orderItems()
    {
        return $this->hasMany(OrderItem::class);
    }
}

```

### Route `web.php`

```php
<?php

use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

use App\Http\Controllers\ProductController;
use App\Http\Controllers\CartController;
use App\Http\Controllers\OrderController;
use App\Http\Controllers\AdminController;



// Site public
Route::get('/', [ProductController::class,'index'])->name('home');
Route::get('/product/{slug}', [ProductController::class,'show'])->name('product.show');
// Route::get('/categories', [CategoryController::class,'index'])->name('categories.index');
Route::get('/trai-cay-nhap-khau', [ProductController::class, 'importedFruits'])->name('importedFruits');
Route::get('/trai-cay-viet-nam', [ProductController::class, 'localFruits'])->name('localFruits');

// trang khuyen mai
Route::get('/khuyen-mai', [ProductController::class, 'promotion'])->name('products.promotion');

// tim kiem
Route::get('/search', [ProductController::class, 'search'])->name('products.search');

Route::get('/welcome', function () {
    return view('welcome');  // Đây là view mặc định trong Laravel
})->name('welcome');

Route::get('/dashboard', function () {
    return view('dashboard'); // hoặc view bất kỳ bạn muốn redirect đến
})->middleware(['auth'])->name('dashboard');

// Auth cần user login
Route::middleware('auth')->group(function(){

  Route::post('/cart/add',[CartController::class,'add'])->name('cart.add');
  Route::get('/cart',      [CartController::class,'index'])->name('cart.index');
  Route::post('/cart/update',[CartController::class,'update'])->name('cart.update');
  Route::post('/cart/remove/{id}',[CartController::class,'remove'])->name('cart.remove');

  Route::get('/checkout',  [OrderController::class,'create'])->name('order.create');
  Route::post('/checkout', [OrderController::class,'store'])->name('order.store');
  Route::get('/orders',    [OrderController::class,'index'])->name('order.index');
});

// auth cho admin
Route::middleware('auth', 'admin')->group(function (){ 
    Route::get('/admin', [AdminController::class, 'index'])->name('admin.products.index');
    
    // Route tạo sản phẩm
    Route::get('/admin/products/create', [ProductController::class, 'create'])->name('admin.products.create');
    Route::post('/admin/products', [ProductController::class, 'store'])->name('admin.products.store');
    
    // Route chỉnh sửa sản phẩm
    Route::get('/admin/products/{id}/edit', [ProductController::class, 'edit'])->name('admin.products.edit');
    Route::put('/admin/products/{id}', [ProductController::class, 'update'])->name('admin.products.update');
    
    // Route xóa sản phẩm
    Route::delete('/admin/products/{id}', [ProductController::class, 'destroy'])->name('admin.products.destroy');
    // xem danh sach don hang
    Route::get('/admin/orders', [OrderController::class, 'adminIndex'])->name('admin.orders.index');
    // xem chi tiet don hang
    Route::get('/admin/orders/{id}', [OrderController::class, 'adminShow'])->name('admin.orders.show');

    // Da giao hang
    Route::patch('/admin/orders/{id}/status', [OrderController::class, 'updateStatus'])->name('admin.orders.updateStatus');

    // Route xem chi tiết sản phẩm
    Route::get('/admin/products/{slug}', [ProductController::class, 'show'])->name('admin.products.show');
}); 



require __DIR__.'/auth.php';

```

---

## 📎 Liên kết dự án

* GitHub Repo: https://github.com/khanh21s/Fruit-Shop.git

---

