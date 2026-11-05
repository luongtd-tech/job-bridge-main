# 📋 CANDIDATE_TASKS.md — Kế hoạch Triển khai Chi tiết Phân hệ Candidate (Ứng viên)
> **Dự án**: JOB BRIDGE — Nền tảng Website & Mobile hỗ trợ Tuyển dụng & Phát triển Sự nghiệp Tích hợp AI  
> **Thư mục gốc**: `C:\job-bridge-main`  
> **Tài nguyên Template**: `aznews/` (Client UI), `NiceAdmin/` (Admin Layout), `elFinder-Summernote/` (Editor & File Manager)  
> **Quy chuẩn Kiến trúc**: Tuân thủ nghiêm ngặt theo `AGENT_RULES.md` (ASP.NET Core 8 MVC, EF Core, SQL Server, SignalR, PagedList, SlugGenerator).

---

## 🛠️ CẤU TRÚC MÔ HÌNH DỮ LIỆU (DATABASE SCHEMAS & ENTITIES)

Tất cả các Model dưới đây được khai báo trong thư mục `Models/` và đưa vào `DataContext.cs`:

1. **`CandidateAccount`** (`tblCandidateAccount`): `CandidateID` (PK), `Email`, `PasswordHash`, `FullName`, `PhoneNumber`, `AvatarUrl`, `CreatedDate`, `IsActive`, `IsVIP` (Dùng cho AI Interview), `VIPExpireDate`.
2. **`CandidateProfile`** (`tblCandidateProfile`): `ProfileID` (PK), `CandidateID` (FK), `Title` (Vị trí mong muốn), `Bio`, `Address`, `CVFileUrl`, `DesiredSalary`, `WorkType`, `ExperienceYears`, `ProfileStatus` (Draft/Pending/Approved).
3. **`CandidateExperience`** (`tblCandidateExperience`): `ExpID` (PK), `CandidateID` (FK), `CompanyName`, `Position`, `StartDate`, `EndDate`, `IsCurrent`, `Description`.
4. **`CandidateEducation`** (`tblCandidateEducation`): `EduID` (PK), `CandidateID` (FK), `SchoolName`, `Major`, `Degree`, `StartDate`, `EndDate`, `GPA`.
5. **`CandidateSkill`** (`tblCandidateSkill`): `SkillID` (PK), `CandidateID` (FK), `SkillName`, `ProficiencyLevel`.
6. **`JobPost`** (`tblJobPost`): `JobID` (PK), `EmployerID` (FK), `Title`, `Slug`, `Description`, `Requirements`, `SalaryMin`, `SalaryMax`, `Location`, `Category`, `CreatedDate`, `ExpireDate`, `IsActive`.
7. **`JobSaved`** (`tblJobSaved`): `SavedID` (PK), `CandidateID` (FK), `JobID` (FK), `SavedDate`.
8. **`JobApplication`** (`tblJobApplication`): `ApplicationID` (PK), `CandidateID` (FK), `JobID` (FK), `AppliedDate`, `Status` (0: Tiếp nhận, 1: Đã xem, 2: Duyệt hồ sơ, 3: Cân nhắc, 4: Phù hợp, 5: Chưa phù hợp), `NoteByEmployer`.
9. **`AssessmentQuiz`** (`tblAssessmentQuiz`): `QuizID` (PK), `Title`, `Category`, `DurationMinutes`, `PassScore` (80%), `TotalQuestions`.
10. **`Question`** (`tblQuestion`): `QuestionID` (PK), `QuizID` (FK), `Content`, `OptionA`, `OptionB`, `OptionC`, `OptionD`, `CorrectOption`.
11. **`CandidateQuizResult`** (`tblCandidateQuizResult`): `ResultID` (PK), `CandidateID` (FK), `QuizID` (FK), `Score`, `IsPassed`, `CompletedDate`, `AnswersJson`.
12. **`AIMockInterviewSession`** (`tblAIMockInterviewSession`): `SessionID` (PK), `CandidateID` (FK), `JobCategory`, `TranscriptJson`, `AIScore`, `FeedbackText`, `CreatedDate`.
13. **`PlacementConnection`** (`tblPlacementConnection`): `ConnectionID` (PK), `CandidateID` (FK), `JobID` (FK), `Status` (0: EligibilityChecked, 1: AdminApproved, 2: Negotiating, 3: Dispatched, 4: Interviewing, 5: ResultUpdated), `InterviewDate`, `AdminNote`.
14. **`Notification`** (`tblNotification`): `NotifID` (PK), `CandidateID` (FK), `Title`, `Message`, `Type`, `IsRead`, `CreatedDate`, `TargetUrl`.
15. **`ChatMessage`** (`tblChatMessage`): `MessageID` (PK), `SenderID`, `ReceiverID`, `Content`, `AttachmentUrl`, `MessageType` (Text/Image/File/Audio/CallLog), `SentDate`, `IsRead`.

---

## 🎯 DANH SÁCH NHIỆM VỤ THI CÔNG HỆ THỐNG CANDIDATE

---

### MÔ-ĐỦN 1: ĐĂNG NHẬP & ĐĂNG KÝ (ACCOUNT & AUTHENTICATION)
> **Controller**: `Controllers/CandidateAuthController.cs`  
> **Views**: `Views/CandidateAuth/Login.cshtml`, `Views/CandidateAuth/Register.cshtml`, `Views/CandidateAuth/ForgotPassword.cshtml`

- [ ] **Task 1.1**: Thiết kế giao diện Đăng ký (`Register.cshtml`) từ Template `aznews` / Form chuẩn UI.
  - Form validation client-side (`jquery.validate`) & server-side (`ModelState`).
  - Kiểm tra email đã tồn tại trong `tblCandidateAccount`.
- [ ] **Task 1.2**: Lập trình Action `Register [HttpPost]`:
  - Mã hóa mật khẩu bằng `Functions.MD5Password` (hoặc BCrypt).
  - Khởi tạo tự động bản ghi `tblCandidateProfile` rỗng gắn liền với `CandidateID`.
- [ ] **Task 1.3**: Thiết kế giao diện Đăng nhập (`Login.cshtml`) & Lập trình Action `Login [HttpPost]`:
  - Xác thực Email và PasswordHash.
  - Lưu phiên đăng nhập sử dụng **Cookie Authentication** (`ClaimsPrincipal`, `CookieAuthenticationDefaults.AuthenticationScheme`) hoặc thiết lập Session Candidate (`Functions._CandidateID`).
- [ ] **Task 1.4**: Xây dựng chức năng Quên mật khẩu & Đăng xuất:
  - Action `Logout`: Xóa Cookie/Session, chuyển hướng về trang chủ.
  - Action `ForgotPassword`: Gửi token xác minh qua Email (SmtpClient/SendGrid placeholder).

---

### MÔ-ĐỦN 2: CÀI ĐẶT HỆ THỐNG (SYSTEM SETTINGS)
> **Controller**: `Controllers/CandidateSettingsController.cs`  
> **Views**: `Views/CandidateSettings/Index.cshtml`, `Views/CandidateSettings/Security.cshtml`

- [ ] **Task 2.1**: Thiết kế trang Cài đặt hệ thống (`Index.cshtml`):
  - Tùy chỉnh bật/tắt nhận thông báo qua Email (Việc làm mới, Trạng thái hồ sơ, Lịch phỏng vấn).
  - Tùy chỉnh chế độ riêng tư hồ sơ (Cho phép Nhà tuyển dụng tìm kiếm CV / Bật chế độ Tìm việc kín).
- [ ] **Task 2.2**: Thiết kế trang Bảo mật tài khoản (`Security.cshtml`):
  - Form Đổi mật khẩu (Mật khẩu cũ, Mật khẩu mới, Xác nhận mật khẩu).
  - Lịch sử đăng nhập & Quản lý thiết bị kết nối.

---

### MÔ-ĐỦN 3: THIẾT LẬP THÔNG TIN CÁ NHÂN & QUẢN LÝ CV (PROFILE & CV BUILDER)
> **Controller**: `Controllers/CandidateProfileController.cs`  
> **Views**: `Views/CandidateProfile/Index.cshtml`, `Views/CandidateProfile/Edit.cshtml`, `Views/CandidateProfile/ManageCV.cshtml`

- [ ] **Task 3.1**: Xây dựng trang tổng quan Hồ sơ cá nhân (`Index.cshtml`):
  - Hiển thị thanh tiến trình hoàn thiện hồ sơ (Profile Completion Bar %).
  - Render danh sách Học vấn, Kinh nghiệm, Kỹ năng và Dự án cá nhân.
- [ ] **Task 3.2**: Xây dựng Form cập nhật thông tin (`Edit.cshtml`):
  - Upload Ảnh đại diện (Avatar) có xem trước (Preview Image).
  - Cập nhật Họ tên, SĐT, Ngày sinh, Địa chỉ, Vị trí ứng tuyển mong muốn, Mức lương kỳ vọng.
  - Thêm / Sửa / Xóa danh sách Kinh nghiệm làm việc (`CandidateExperience`) & Học vấn (`CandidateEducation`) qua Ajax.
  - Thêm / Xóa Kỹ năng (`CandidateSkill`) dạng Tag Input.
- [ ] **Task 3.3**: Quản lý Tệp tin CV (`ManageCV.cshtml`):
  - Tải lên tệp CV dạng PDF/Word (lưu vào thư mục `wwwroot/uploads/cvs/`).
  - Xem trước CV bằng đĩa xem PDF trực tiếp trên Web (`<iframe>` hoặc `PDF.js`).
  - Tích hợp công cụ phân tích & chấm điểm CV bằng AI Placeholder (`AICVScorer`).

---

### MÔ-ĐỦN 4: QUẢN LÝ VIỆC LÀM (JOB PORTAL & APPLICATION TRACKING)
> **Controller**: `Controllers/JobController.cs`  
> **Views**: `Views/Job/Index.cshtml`, `Views/Job/Details.cshtml`, `Views/Job/SavedJobs.cshtml`, `Views/Job/MatchingJobs.cshtml`, `Views/Job/AppliedJobs.cshtml`

- [ ] **Task 4.1**: Tìm kiếm việc làm (`Index.cshtml`):
  - Bộ lọc đa tiêu chí: Từ khóa, Ngành nghề (`Category`), Địa điểm (`Location`), Mức lương, Hình thức làm việc.
  - Phân trang danh sách công việc sử dụng `PagedList.Core.Mvc`.
  - Tối ưu SEO URL bằng Attribute Routing: `[Route("/viec-lam-{slug}-{id:long}.html")]`.
- [ ] **Task 4.2**: Trang chi tiết việc làm (`Details.cshtml`):
  - Hiển thị đầy đủ Mô tả công việc, Yêu cầu, Quyền lợi (Dùng `@Html.Raw()` cho nội dung Summernote).
  - Nút "Ứng tuyển ngay" (Submit CV modal) & Nút "Lưu tin tuyển dụng".
- [ ] **Task 4.3**: Lưu việc làm (`SavedJobs.cshtml`):
  - Action `SaveJob [HttpPost]` (Ajax): Thêm/Xóa bản ghi `tblJobSaved`.
  - Hiển thị danh sách các công việc đã lưu kèm nút bỏ lưu.
- [ ] **Task 4.4**: Việc làm phù hợp (`MatchingJobs.cshtml`):
  - Thuật toán gợi ý: So khớp Kỹ năng (`CandidateSkill`) & Vị trí (`Title`) của ứng viên với Yêu cầu của `JobPost`.
  - Đánh giá % độ phù hợp (Matching Score %) cho từng công việc.
- [ ] **Task 4.5**: Quản lý Việc làm đã ứng tuyển (`AppliedJobs.cshtml`):
  - Pipeline theo dõi trạng thái hồ sơ 6 bước:  
    `0: Tiếp nhận` ➔ `1: Đã xem` ➔ `2: Duyệt hồ sơ` ➔ `3: Cân nhắc` ➔ `4: Phù hợp` ➔ `5: Chưa phù hợp`.
  - Giao diện Timeline trực quan minh họa từng bước xử lý của Nhà tuyển dụng.
  - Hiển thị Phản hồi / Ghi chú từ Nhà tuyển dụng (`NoteByEmployer`).

---

### MÔ-ĐỦN 5: ĐÁNH GIÁ NĂNG LỰC & KẾT NỐI TUYỂN DỤNG (ASSESSMENT & AI CONNECT)
> **Controller**: `Controllers/CandidateAssessmentController.cs`  
> **Views**: `Views/CandidateAssessment/QuizList.cshtml`, `Views/CandidateAssessment/TakeQuiz.cshtml`, `Views/CandidateAssessment/AIMockInterview.cshtml`, `Views/CandidateAssessment/Report.cshtml`, `Views/CandidateAssessment/PlacementMatching.cshtml`

#### 5.1. Luyện tập và đánh giá:
- [ ] **Task 5.1.1**: Danh sách bài kiểm tra trắc nghiệm (`QuizList.cshtml`):
  - Hiển thị các bài test theo kỹ năng/ngành nghề (Chuyên môn, Tiếng Anh, IQ/Logic).
- [ ] **Task 5.1.2**: Giao diện Làm bài test trắc nghiệm (`TakeQuiz.cshtml`):
  - Đếm ngược thời gian (Countdown Timer JavaScript).
  - Lưu kết quả vào `tblCandidateQuizResult`. Tự động tính điểm và đánh giá Đạt/Không đạt (Thang điểm Pass >= 80%).
- [ ] **Task 5.1.3**: Phỏng vấn mô phỏng với AI (`AIMockInterview.cshtml`):
  - Giao diện Chat/Voice tương tác với Nhà tuyển dụng AI (Speech-to-Text & Text-to-Speech Web API/Gemini API integration).
  - Kiểm tra giới hạn lượt dùng thử. Nếu hết lượt ➔ Bật Modal trả phí nâng cấp VIP (`IsVIP`).
- [ ] **Task 5.1.4**: Báo cáo tổng hợp & Đề xuất lộ trình (`Report.cshtml`):
  - Biểu đồ Ra-da (Radar Chart - Chart.js) thể hiện 5 khía cạnh năng lực.
  - Tư vấn điểm yếu & Đề xuất danh sách Khóa học / Lộ trình đào tạo phù hợp (Quy trình Monetization bán khóa học).

#### 5.2. Kết nối tuyển dụng VIP (Placement Matching):
- [ ] **Task 5.2.1**: Trang kiểm tra điều kiện Kết nối (`PlacementMatching.cshtml`):
  - Hệ thống tự động kiểm tra 2 điều kiện bắt buộc:
    1. Trạng thái ứng tuyển công việc đạt **Cân nhắc** (Status 3) hoặc **Phù hợp** (Status 4).
    2. Điểm kiểm tra trắc nghiệm chuyên môn đạt **>= 80%**.
- [ ] **Task 5.2.2**: Quy trình đẩy hồ sơ xét duyệt:
  - Đủ điều kiện ➔ Cho phép ứng viên gửi yêu cầu "Kết nối trực tiếp Doanh nghiệp".
  - Chuyển trạng thái sang `AdminApproved` ➔ Admin đàm phán với Doanh nghiệp ➔ Gửi danh sách Ứng viên VIP.
  - Lên lịch Phỏng vấn 1-1 & Cập nhật kết quả trúng tuyển trên hệ thống.

---

### MÔ-ĐỦN 6: HỆ THỐNG THÔNG BÁO REAL-TIME (NOTIFICATIONS)
> **Controller**: `Controllers/CandidateNotificationController.cs`  
> **Hub**: `Hubs/NotificationHub.cs` (SignalR)  
> **Views**: `Views/CandidateNotification/Index.cshtml`, `Views/Shared/Components/Notification/Default.cshtml`

- [ ] **Task 6.1**: Cấu hình SignalR Hub (`NotificationHub.cs`) tại `Program.cs`:
  - `builder.Services.AddSignalR();` và `app.MapHub<NotificationHub>("/notificationHub");`.
- [ ] **Task 6.2**: Tích hợp ViewComponent Thông báo trên Header:
  - Hiển thị chuông thông báo + Số lượng thông báo chưa đọc (`IsRead == false`).
  - Dropdown danh sách 5 thông báo mới nhất.
- [ ] **Task 6.3**: Trang Quản lý Thông báo (`Index.cshtml`):
  - Phân loại thông báo (Tuyển dụng, Bài test, Hệ thống, Chat).
  - Nút "Đánh dấu tất cả là đã đọc" (Ajax call).

---

### MÔ-ĐỦN 7: TƯƠNG TÁC CHAT REAL-TIME KÍỂU ZALO (MESSAGING & INTERACTION)
> **Controller**: `Controllers/CandidateChatController.cs`  
> **Hub**: `Hubs/ChatHub.cs` (SignalR)  
> **Views**: `Views/CandidateChat/Index.cshtml`

- [ ] **Task 7.1**: Xây dựng Giao diện Chat Zalo-Style (`Index.cshtml`):
  - Cột trái: Danh sách cuộc trò chuyện với Nhà tuyển dụng / Tư vấn viên (Có Avatar, Tên, Tin nhắn cuối, Badge chưa đọc, Đèn xanh Online/Offline).
  - Cột phải: Khung chat chi tiết, Header thông tin đối phương, Khung cuộn tin nhắn, Input gõ tin nhắn + Icon đính kèm.
- [ ] **Task 7.2**: Lập trình Real-time Messaging với SignalR (`ChatHub.cs`):
  - Phương thức `SendMessage(string receiverId, string message)`: Tự động lưu bản ghi vào `tblChatMessage` và đẩy ngay lập tức tới Receiver qua `Clients.User(receiverId).SendAsync("ReceiveMessage", ...)`.
  - Trạng thái "Đã xem" (Read Receipt) & Trạng thái "Đang gõ..." (Typing indicator).
- [ ] **Task 7.3**: Đính kèm Tệp tin & Gọi thoại Placeholder:
  - Tải ảnh/tệp qua Ajax ➔ Render khung xem trước trong hội thoại.
  - Nút Gọi thoại / Video Call Modal UI (WebRTC / ZegoCloud Integration placeholder).

---

## 🚀 TIẾN TRÌNH THỰC THI CHUẨN ĐÀNH CHO AI AGENT

Khi bắt đầu code phân hệ Candidate, Agent phải chạy lần lượt các giai đoạn sau:

1. **Giai đoạn 1**: Tạo tất cả các Model C# trong `Models/` và cập nhật `DataContext.cs`. Chạy Migration hoặc tạo bảng SQL Server.
2. **Giai đoạn 2**: Viết `CandidateAuthController` & Views (Đăng ký, Đăng nhập, Cookie Auth).
3. **Giai đoạn 3**: Xây dựng `CandidateProfileController` & View chỉnh sửa CV/Hồ sơ.
4. **Giai đoạn 4**: Xây dựng `JobController` (Tìm kiếm, Chi tiết SEO, Lưu việc, Ứng tuyển & Theo dõi trạng thái Hồ sơ).
5. **Giai đoạn 5**: Lập trình Mô-đun Đánh giá (`CandidateAssessmentController`: Trắc nghiệm, AI Interview, Báo cáo & Kết nối VIP).
6. **Giai đoạn 6**: Cài đặt SignalR cho Thông báo (`NotificationHub`) và Khung Chat Zalo-style (`ChatHub`).
