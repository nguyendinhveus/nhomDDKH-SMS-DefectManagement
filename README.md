# Báo cáo Quản lý Lỗi Phần Mềm (Defect Management) - Nhóm DDKH

## Thông tin nhóm
- Thành viên:  
Đào Tiến Đạt – BIT230081
Nguyễn Hà Kiên – BIT
Nguyễn Định – BIT230091
Nguyễn Thu Hiền – BIT
- Ứng dụng test: Student Attendance Management System (MERN stack)
- Link demo: https://student-attandance-management-system.netlify.app/
- Default credentials:
  - Teacher: ID `t001` / Password `abc@123` / Role Teacher
  - Student: ID `std_1` / Password `linus@123` / Role Student

## Link sản phẩm nộp
1. GitHub Repository (với Issues):https://github.com/nguyendinhveus/nhomDDKH-SMS-DefectManagement.git
2. Jira Project (Scrum): https://webtk.atlassian.net/jira/software/projects/SMS/boards/71/backlog?selectedIssue=SMS-2

## Danh sách bug
Nhóm đã ghi nhận tổng cộng **7 bug** qua manual testing trên demo online (tập trung vào chức năng đăng nhập, quản lý sinh viên CRUD, attendance, phân quyền).

### GitHub Issues (Phần A – Quản lý đơn giản)
- #1: [Bug] Thiếu chức năng Edit/Update thông tin sinh viên đã thêm – Major/High
- #2: [Bug] Thiếu search/filter trong Student List và Attendance – Medium/Medium
- #3: [Bug] Validation client-side yếu hoặc thiếu khi Add Student – Minor/Medium
- #4: [Bug] Xóa student không có xác nhận – Minor/Medium
- #5: [Bug] Attendance có thể bị chỉnh sửa trực tiếp mà không có audit trail, confirmation, hoặc version history – Critical/High
- #6: [Bug] Khả năng truy cập trái phép vào teacher dashboard – Critical/High
- #7: [Bug] Logout button không có xác nhận đăng xuất – Minor/Low

### Jira (Phần B – Mô phỏng doanh nghiệp, Scrum project)
- Tương tự danh sách trên (copy từ GitHub), tổng 7 bug (trong đó 2 Critical).
- Bug được tạo với custom fields: Severity, Priority, Test Environment ("Demo Online Netlify").
- Workflow: To do → In Progress → Fixed → QA Verify → Done.
- Các bug được kéo vào Sprint (1 tuần).

## 2 Bug nghiêm trọng (phân tích kỹ)
### Bug 1: Attendance có thể bị chỉnh sửa trực tiếp mà không có audit trail, confirmation, hoặc version history (#5 – Critical/High)
- **Mô tả ngắn gọn**: Teacher có thể thay đổi status Present/Absent bất kỳ lúc nào mà không cần xác nhận, không lưu lịch sử thay đổi (audit log), không có undo.
- **Steps to Reproduce**:
  1. Login Teacher → Vào Attendance section.
  2. Chọn student → chỉnh sửa status (editable field).
  3. Thay đổi nhiều lần → kiểm tra network/console (F12) → không thấy log thay đổi.
  4. Login Student → xem percentage/record đã thay đổi.
- **Expected Result**: Popup confirm trước save; audit log (ai thay đổi, thời gian, old/new value); nút Undo hoặc history.
- **Actual Result**: Update realtime (sync MongoDB) mà không confirm/audit → dễ gian lận (che giấu vắng mặt).
- **Severity/Priority**: Critical / High
- **Impact & Root cause giả định**:
  - Rủi ro cao về tính toàn vẹn dữ liệu: Attendance là yếu tố quan trọng đánh giá học vụ, thay đổi không traceable → có thể dẫn đến tranh chấp, gian lận điểm danh.
  - Trong doanh nghiệp thực tế: Vi phạm quy định kiểm toán, không tuân thủ GDPR-like cho dữ liệu giáo dục.
  - Root cause: Thiếu middleware audit ở backend (Express) và confirmation ở frontend (React).
- **Attachments**  
Trước chỉnh sửa
<img width="1341" height="636" alt="Image" src="https://github.com/user-attachments/assets/7e71eac0-64c5-47e6-84a0-e998cbf1d470" />
Cập nhật "vắng" ở cột ngày đầu tiên của bạn thứ 3 -> ghi nhận không có  confirm nào
<img width="1348" height="634" alt="Image" src="https://github.com/user-attachments/assets/b3390c13-fa72-4973-95f5-49daf2a71271" />

### Bug 2: Khả năng truy cập trái phép vào teacher dashboard (#6 – Critical/High)
- **Mô tả ngắn gọn**: Student có thể truy cập dashboard Teacher bằng cách thay đổi URL thủ công (permission bypass tiềm năng).
- **Steps to Reproduce**:
  1. Login Student (std_1 / linus@123).
  2. Quan sát URL sau login (thường /student hoặc root).
  3. Thử paste URL teacher dashboard (copy từ session Teacher) hoặc đoán /teacher, /dashboard.
- **Expected Result**: Redirect về student page hoặc 403 error (JWT/role check nghiêm ngặt).
- **Actual Result**: Có thể thấy dashboard Teacher → lộ chức năng quản lý (add/delete student, mark attendance).
- **Severity/Priority**: Critical / High
- **Impact & Root cause giả định**:
  - Rủi ro bảo mật nghiêm trọng: Student có thể thao túng dữ liệu (xóa student, thay đổi attendance) → ảnh hưởng toàn hệ thống.
  - Trong thực tế: Vi phạm nguyên tắc least privilege, có thể dẫn đến data breach hoặc lạm dụng.
  - Root cause: Frontend routing không check role đầy đủ, hoặc JWT không validate role ở một số route API.
- **Attachments**  
Đăng nhập tài khoản student
<img width="1357" height="676" alt="Image" src="https://github.com/user-attachments/assets/beae8717-16d6-4c60-8533-cb403ff7d9f1" />
Đổi URL thủ công /student thành /teacher -> thấy được dashboard của của teacher
<img width="1340" height="670" alt="Image" src="https://github.com/user-attachments/assets/8d207dd5-0230-4029-84ef-cd212dc84a8c" />
## So sánh GitHub Issues vs Jira

| Tiêu chí              | GitHub Issues                          | Jira (Scrum)                              |
|-----------------------|----------------------------------------|-------------------------------------------|
| Dễ sử dụng            | Rất dễ, tích hợp trực tiếp với repo, miễn phí hoàn toàn | Cần học curve, miễn phí cho team nhỏ (<10 users) |
| Workflow              | Cơ bản (labels + status thủ công)      | Chuyên nghiệp, customize workflow (New → QA Verify → Done) |
| Báo cáo & thống kê    | Hạn chế (filter, milestones cơ bản)    | Mạnh mẽ (Sprint Report, Burndown, Bug by Priority/Status, charts) |
| Phù hợp               | Dự án nhỏ, nhóm học tập, OSS           | Doanh nghiệp, team chuyên nghiệp, sprint-based |
| Attach file & media   | Tốt (kéo thả ảnh/video)                | Xuất sắc (upload trực tiếp, embed Loom/video) |
| Tích hợp vai trò      | Assignee + labels đơn giản             | Assignee, custom fields (Severity), permissions rõ ràng |

## Nhận xét quy trình Defect Management
Quy trình quản lý lỗi phần mềm (Defect Management) rất quan trọng để đảm bảo chất lượng sản phẩm, theo dõi tiến độ và phân công rõ ràng giữa Tester – Developer – Project Manager.

- **Ưu điểm**:
  - GitHub Issues phù hợp cho nhóm nhỏ: nhanh chóng tạo bug, gắn label, assign, dễ theo dõi vòng đời (Open → Fixed → Closed).
  - Jira (Scrum) mô phỏng doanh nghiệp tốt hơn: workflow chuyên nghiệp, báo cáo thống kê (Bug by Status/Priority, trend trong sprint), custom fields Severity/Priority giúp ưu tiên chính xác.
  - Vai trò xoay vòng giúp hiểu rõ trách nhiệm: Tester log & verify, Developer fix (giả lập), PM quản lý sprint/ưu tiên.

- **Hạn chế & bài học**:
  - App demo online đôi khi chậm (free host Netlify/Render) → ảnh hưởng test, nhưng tiện không cần setup local.
  - Quy trình giúp phát hiện sớm bug nghiêm trọng (Critical như permission bypass, no audit attendance) – tránh rủi ro data integrity/security.
  - Cần kết hợp cả hai công cụ: GitHub cho nhanh gọn, Jira cho báo cáo chuyên sâu và sprint kiểm thử.

Tổng thể, bài thực hành giúp nhóm nắm rõ Bug Life Cycle (New → Fixed → Verified → Closed), tầm quan trọng của Defect Report chuẩn (Steps, Expected/Actual, Severity, Attachments), và sự khác biệt giữa công cụ đơn giản vs chuyên nghiệp.

