---

Tệp này quy định toàn bộ kiến thức kiến trúc, quy chuẩn thư mục, mô hình dữ liệu, quy trình mã hóa và luồng phát triển cho AI Agent (hoặc Developer) khi triển khai một ứng dụng Web ASP.NET Core 8 MVC mới.

---

## 1. TỔNG QUAN CÔNG NGHỆ & MÔ HÌNH DỰ ÁN

* **Framework cốt lõi**: .NET 8.0 SDK, ASP.NET Core 8.0 MVC.
* **Môi trường & Công cụ**: Visual Studio Code, SQL Server 2022, .NET CLI.
* **Database & ORM**: SQL Server, Entity Framework Core (`Microsoft.EntityFrameworkCore.SqlServer`).
* **Các thư viện bổ trợ bắt buộc**:
  * `SlugGenerator`: Tạo đường dẫn tĩnh (URL Slug) chuẩn SEO.
  * `PagedList.Core.Mvc`: Phân trang dữ liệu danh sách.
  * `elFinder.NetCore`: Quản lý tệp tin & hình ảnh server.
  * `Microsoft.VisualStudio.Web.CodeGeneration.Design`: Hỗ trợ Scaffolding và phân vùng Area.

---

## 2. CẤU TRÚC THƯ MỤC CHUẨN (PROJECT STRUCTURE)

Dự án phải tuân thủ nghiêm ngặt cấu trúc phân tách giữa **Client (Người dùng)** và **Admin (Quản trị)**:

```text
[ProjectRoot]/
├── Areas/
│   └── Admin/                         # Phân vùng quản trị chuyên biệt
│       ├── Components/                # ViewComponents riêng cho Admin (e.g., AdminMenuComponent)
│       ├── Controllers/               # Controllers xử lý nghiệp vụ Admin
│       ├── Models/                    # Models/ViewModels riêng cho Admin
│       └── Views/                     # Razor Views quản trị
│           ├── Home/                  # Trang Dashboard Admin
│           ├── Login/ & Register/     # Trang xác thực
│           ├── [EntityName]/          # Các trang quản trị CRUD (e.g., Menu, Post)
│           └── Shared/                # Views dùng chung Admin (_LayoutAdmin, ViewComponents)
├── Components/                        # ViewComponents cho trang Client (MenuView, PostView)
├── Controllers/                       # Controllers trang Client (HomeController)
├── Models/                            # CSDL Models & Entity Framework DataContext
├── Utilities/                         # Lớp tiện ích tĩnh (Functions.cs: Slug, MD5, Session Check)
├── Views/                             # Razor Views Client
│   ├── Home/                          # Index.cshtml, Details.cshtml
│   └── Shared/                        # _Layout.cshtml, Components/
├── wwwroot/                           # Tài nguyên tĩnh
│   ├── assets/                        # CSS/JS/Images dành cho Client
│   ├── admin/                         # Tài nguyên Template Admin (NiceAdmin/Bootstrap)
│   │   └── lib/                       # Thư viện ngoài (elFinder, Summernote, jQuery UI)
│   └── uploads/                       # Thư mục chứa tệp tin/hình ảnh do elFinder tải lên
├── appsettings.json                   # Chứa ConnectionStrings kết nối SQL Server
└── Program.cs                         # Cấu hình Services, DbContext & Routing
```

---

## 3. NGUYÊN TẮC THIẾT KẾ CƠ SỞ DỮ LIỆU & EF CORE

1. **Đặt tên Bảng & Thuộc tính**:
   * Sử dụng Data Annotations `[Table("tblName")]` và `[Key]` cho mọi Class Model.
   * Khóa chính tự tăng: `int` hoặc `long` (VD: `MenuID`, `PostID`, `AdminMenuID`, `UserID`).
   * Các trường trạng thái & thứ tự: `IsActive` (`bit`), `Levels` (`int`), `ParentID` (`int`), `MenuOrder` (`int`).
2. **Kỹ thuật ánh xạ SQL View**:
   * Khi cần JOIN nhiều bảng phục vụ hiển thị Client, tạo **SQL View** trong SQL Server (VD: `viewPostMenu`).
   * Tạo Model tương ứng trong C# có thuộc tính `[Table("viewPostMenu")]` và khai báo `DbSet<viewPostMenu>` trong `DataContext`.
3. **Cấu hình `DataContext.cs`**:
   * Kế thừa từ `DbContext`.
   * Khai báo tất cả `DbSet<T>` của hệ thống tại đây.
4. **Chuỗi kết nối `appsettings.json`**:
   ```json
   "ConnectionStrings": {
     "DefaultConnection": "Data Source=YOUR_SERVER_NAME;Initial Catalog=YOUR_DB_NAME;Integrated Security=True;TrustServerCertificate=True"
   }
   ```
5. **Đăng ký Service tại `Program.cs`**:
   ```csharp
   var connection = builder.Configuration.GetConnectionString("DefaultConnection");
   builder.Services.AddDbContext<DataContext>(options => options.UseSqlServer(connection));
   ```

---

## 4. QUY CHUẨN ĐỊNH TUYẾN & TỐI ƯU SEO (ROUTING & SLUG)

1. **Định tuyến khu vực (Area Route)**:
   * Khai báo route Area tại `Program.cs` trước route default:
     ```csharp
     app.MapControllerRoute(
         name: "areas",
         pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
     ```
   * Tất cả Controller trong phân vùng Admin phải khai báo attribute `[Area("Admin")]`.
2. **Attribute Routing & Đường dẫn chuẩn SEO**:
   * Sử dụng `SlugGenerator` và tiện ích `Functions.TitleSlugGeneration(type, title, id)` để sinh chuỗi URL SEO.
   * Cấu hình Attribute Route trên Action hiển thị chi tiết:
     ```csharp
     [Route("/[type]-{slug}-{id:long}.html", Name = "Details")]
     public IActionResult Details(long? id)
     ```
   * Cấu hình Attribute Route cho trang phân trang:
     ```csharp
     [Route("/Admin/[controller]-index{page:int}.html", Name = "[Controller]Index")]
     ```

---

## 5. NGUYÊN TẮC PHÁT TRIỂN GIAO DIỆN & VIEWCOMPONENT

1. **Bóc tách Layout**:
   * Cấu hình đường dẫn tài nguyên tĩnh luôn có dấu `~` ở đầu: `~/assets/...` hoặc `~/admin/assets/...`.
   * Sử dụng `@RenderBody()` tại khung nội dung chính của `_Layout.cshtml` và `_LayoutAdmin.cshtml`.
2. **ViewComponent (Đổ dữ liệu động vào Layout)**:
   * Mọi khu vực dữ liệu tái sử dụng (Menu header, Sidebar Admin, Tin nổi bật) phải viết thành `ViewComponent`.
   * Tạo Class kế thừa `ViewComponent`, gắn annotation `[ViewComponent(Name = "Name")]`.
   * Đặt file View hiển thị đúng cấu trúc thư mục: `Views/Shared/Components/[ComponentName]/Default.cshtml`.
   * Nhúng ViewComponent vào Layout bằng cú pháp: `@await Component.InvokeAsync("ComponentName")`.
3. **Hiển thị dữ liệu Rich Text**:
   * Sử dụng `@Html.Raw(Model.Contents)` để render nội dung HTML từ trình soạn thảo mà không bị mã hóa mã lệnh.

---

## 6. QUY TRÌNH CHỨC NĂNG QUẢN TRỊ (ADMIN CRUD & EXTENSIONS)

1. **Phân trang dữ liệu (`PagedList.Core.Mvc`)**:
   * Khai báo `@addTagHelper *, PagedList.Core.Mvc` trong `_ViewImports.cshtml`.
   * Xử lý truy vấn phân trang tại Controller:
     ```csharp
     var items = _context.Posts.OrderByDescending(p => p.PostID);
     PagedList<tblPost> models = new PagedList<tblPost>(items, page, pageSize);
     return View(models);
     ```
   * Render bộ nút phân trang tại Razor View dùng `<pager list="@Model" asp-area="Admin" asp-controller="..." asp-action="Index" />`.
2. **Trình soạn thảo Summernote & Quản lý tệp elFinder**:
   * Đăng ký route connector cho `FileManagerController` trỏ về thư mục `uploads`.
   * Nhúng Partial View `_SummerNote.cshtml` kết hợp Model `SummerNote.cs` để kích hoạt trình soạn thảo WYSIWYG có tích hợp nút chọn ảnh trực tiếp từ elFinder.
3. **Xử lý Form & Tag Helpers**:
   * Luôn sử dụng Tag Helpers: `asp-for`, `asp-controller`, `asp-action`, `asp-items`.
   * Truyền dữ liệu danh sách chọn (Dropdown) qua `ViewBag.mnList` dưới dạng `List<SelectListItem>`.
   * Kiểm tra tính hợp lệ dữ liệu bằng `if (ModelState.IsValid)` trước khi lưu vào CSDL qua EF Core `_context.SaveChanges()`.

---

## 7. BẢO MẬT & XÁC THỰC NGƯỜI DÙNG (AUTHENTICATION)

1. **Mã hóa Mật khẩu**:
   * Mật khẩu người dùng bắt buộc phải được băm qua hàm mã hóa `Functions.MD5Password(string text)` trước khi lưu CSDL hoặc so sánh đăng nhập.
2. **Kiểm soát truy cập (Authentication Guard)**:
   * Mọi Action thuộc Area Admin (ngoại trừ `LoginController` và `RegisterController`) bắt buộc phải đặt đoạn mã kiểm tra phiên đăng nhập ở đầu phương thức:
     ```csharp
     if (!Functions.IsLogin())
         return RedirectToAction("Index", "Login");
     ```
3. **Đăng xuất (Logout)**:
   * Xóa thông tin phiên làm việc tĩnh (`Functions._UserID = 0; Functions._UserName = "";`) và điều hướng về trang Login hoặc trang chủ Client.

---

## 8. LUỒNG TỰ ĐỘNG THI CÔNG KHI NHẬN YÊU CẦU DỰ ÁN MỚI

Khi triển khai một ứng dụng web mới, AI Agent phải thực hiện theo đúng 7 bước thứ tự sau:

* **Bước 1**: Khởi tạo dự án MVC bằng `dotnet new mvc`, tin cậy SSL và cài đặt đầy đủ các Package.
* **Bước 2**: Tạo CSDL SQL Server, thiết kế các bảng (`tblMenu`, `tblPost`, `AdminMenu`, `AdminUser`), tạo SQL Views và kết nối qua EF Core (`DataContext`).
* **Bước 3**: Nhúng Template Frontend Client, bóc tách `_Layout.cshtml`, xây dựng các `ViewComponent` cho Client.
* **Bước 4**: Khởi tạo `Area Admin` bằng lệnh CLI, nhúng Template Admin, cấu hình `_LayoutAdmin.cshtml` và `AdminMenuComponent` động.
* **Bước 5**: Xây dựng Lớp tiện ích `Functions.cs` (xử lý Slug, mã hóa MD5, kiểm tra trạng thái đăng nhập).
* **Bước 6**: Lập trình trọn vẹn các chức năng CRUD Admin có tích hợp Phân trang (`PagedList`), Trình quản lý tệp (`elFinder`) và Trình biên tập nội dung (`Summernote`).
* **Bước 7**: Lập trình mô-đun Xác thực (Đăng ký, Đăng nhập, Đăng xuất) và gắn bộ lọc bảo mật `Functions.IsLogin()` cho toàn bộ các Controller quản trị.
```

---