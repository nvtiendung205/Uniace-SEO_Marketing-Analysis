# Từ điển Dữ liệu (Data Dictionary) - Uniace SEO Marketing

Tài liệu này mô tả chi tiết cấu trúc bảng, định nghĩa các trường dữ liệu gốc (Raw Event Logs), quy tắc làm sạch (Data Cleaning) và các chỉ số tính toán (SQL & DAX Metrics) trong dự án phân tích lưu lượng truy cập website Uniace (01/08/2021 - 24/08/2021).

---

## 1. Danh mục các tệp dữ liệu

| Tên tệp | Đường dẫn | Số dòng (bản ghi) | Mô tả nội dung |
| :--- | :--- | :--- | :--- |
| `Uniace_1_clean.csv` | `data/raw/Uniace_1_clean.csv` | 50.919 | Nhật ký tương tác người dùng giai đoạn 1 (19 cột) |
| `Uniace_2_clean.csv` | `data/raw/Uniace_2_clean.csv` | 23.385 | Nhật ký tương tác người dùng giai đoạn 2 (19 cột) |
| `Uniace_3_clean.csv` | `data/raw/Uniace_3_clean.csv` | 19.776 | Nhật ký tương tác, tài khoản và đơn hàng giai đoạn 3 (14 cột) |
| `sample_Uniace_*.csv` | `data/sample/` | 1.000 / tệp | Bộ dữ liệu trích xuất mẫu (1.000 dòng đầu tiên mỗi tệp) |

---

## 2. Cấu trúc các trường dữ liệu gốc (Raw Schema)

| STT | Tên trường | Kiểu dữ liệu | Nhóm chỉ tiêu | Mô tả ý nghĩa nghiệp vụ |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `ID` | Integer | Dimension (Key) | Mã định danh thứ tự bản ghi sự kiện trong từng bảng dữ liệu |
| 2 | `Email` | String | Dimension (User) | Địa chỉ email của người dùng (chỉ ghi nhận khi đã đăng ký/đăng nhập) |
| 3 | `Type` | String | Dimension (Event) | Phân loại sự kiện hệ thống (ví dụ: `page`, `event`, `email`) |
| 4 | `Name` | String | Dimension (Content) | Tên trang bài viết, khóa học hoặc hành động (`New Account`, `Completed Order`) |
| 5 | `Title` | String | Dimension (Content) | Tiêu đề hiển thị trên trình duyệt của trang web (Page Title) |
| 6 | `MA_URL` | String | Dimension (URL) | Đường dẫn URL đầy đủ của trang người dùng đang truy cập |
| 7 | `MA_Referrer` | String | Dimension (Channel) | Nguồn dẫn người dùng tới trang (Google, Cốc Cốc, Facebook, Beacons, Internal) |
| 8 | `ma_path` | String | Dimension (URL) | Đường dẫn tương đối (URI Slug) sau tên miền chính `uniace.vn` |
| 9 | `IP_Address` | String | Dimension (User) | Địa chỉ IP của thiết bị truy cập, dùng để định danh khách vãng lai |
| 10 | `cuid` | UUID / String | Dimension (User) | Mã định danh trình duyệt/thiết bị người dùng (Client Unique ID) |
| 11 | `Datetimem` | String | Dimension (Time) | Thời gian ghi nhận sự kiện ở định dạng thô `M/D/YYYY H:mm` |
| 12 | `Datetime` | Datetime | Dimension (Time) | Thời gian chuẩn hóa theo định dạng chuẩn ISO `YYYY-MM-DD HH:mm:ss` |
| 13 | `Message_Id` | String | Dimension (Campaign)| Mã thông điệp trong hệ thống Marketing Automation |
| 14 | `Template_Id` | String | Dimension (Campaign)| Mã mẫu giao diện email/thông báo (Template) |
| 15 | `List_Id` | String | Dimension (Campaign)| Mã danh sách phân khúc khách hàng nhận chiến dịch |
| 16 | `Scenario_Id` | String | Dimension (Campaign)| Mã kịch bản tự động hóa (Automation Scenario) |
| 17 | `URL` | String | Dimension (URL) | Đường dẫn đích trong các sự kiện tương tác thông điệp |
| 18 | `Link` | String | Dimension (URL) | Liên kết đính kèm phát sinh lượt nhấp (Click Link) |
| 19 | `Tag` | String | Dimension (Segment) | Nhãn phân loại hành vi hoặc phân khúc người dùng |

---

## 3. Quy tắc làm sạch và chuẩn hóa dữ liệu (Data Cleaning Rules)

| Trường dữ liệu | Lỗi phát hiện trên dữ liệu thô | Quy tắc xử lý và chuẩn hóa |
| :--- | :--- | :--- |
| `Email` | Dữ liệu bị tràn hoặc lẫn sang cột `MA_URL` | Tách và khôi phục về đúng cột `Email`, chuẩn hóa chữ thường `LOWER(TRIM(Email))` |
| `MA_URL` | Bị khuyết thiếu ở một số bản ghi sự kiện | Nội suy và khôi phục đường dẫn dựa trên thông tin lưu tại cột `Tag` và `ma_path` |
| `MA_Referrer` | Dữ liệu bị nhảy lệch sang các cột `Message_Id`, `Template_Id` | Dịch chuyển lại đúng vị trí cột nguồn giới thiệu |
| `IP_Address` | Bị lệch vị trí cột (cột IP nhảy sang `Datetime`, `Datetime` nhảy sang `Template_Id`) | Căn chỉnh lại cấu trúc cột, loại bỏ khoảng trắng thừa bằng `TRIM(IP_Address)` |
| `cuid` | Bị xô lệch vị trí cột ở các dòng lỗi phân cách CSV | Cố định lại vị trí cột định danh thiết bị |
| `Datetime` | Bị khuyết giá trị (`NULL`) hoặc sai định dạng chuỗi thời gian | Chuyển đổi qua `STR_TO_DATE`, nội suy ngày bị khuyết từ bản ghi liền kề (`LAG` / `LEAD` theo `ID`) |

---

## 4. Từ điển các chỉ số phân tích (SQL & Power BI DAX Metrics)

| Tên chỉ số (Metric) | Nhóm | Công thức / Logic xác định | Ý nghĩa phân tích |
| :--- | :--- | :--- | :--- |
| `Total_visits` | Fact (Traffic) | Tổng số bản ghi tương tác hợp lệ | Tổng lưu lượng lượt xem/tương tác trên toàn website (78.000 lượt) |
| `total_user` | Fact (Traffic) | `COUNT(DISTINCT COALESCE(IP_Address, Email))` | Tổng số người dùng duy nhất truy cập hệ thống (13.619 người) |
| `DirectUsers` | Fact (Traffic) | Số người dùng có `MA_Referrer` trống hoặc trực tiếp | Lượng người dùng truy cập trực tiếp không qua trung gian (3.607 người) |
| `Session_Count` | Fact (Engagement) | Tổng số phiên làm việc của người dùng | Tổng số phiên truy cập ghi nhận trong chiến dịch (22.000 phiên) |
| `Avg_Sessions_Per_User` | Fact (Engagement) | `Session_Count / total_user` | Tần suất phiên truy cập trung bình trên mỗi người dùng (1.64 phiên/người) |
| `BounceRate` | Fact (Engagement) | Tỷ lệ phiên chỉ xem 1 trang duy nhất rồi thoát | Tỷ lệ thoát trang trung bình toàn chiến dịch (52.39%) |
| `Monthly_Returning_Rate` | Fact (Retention) | Tỷ lệ người dùng có ngày truy cập > `MIN(first_date)` | Tỷ lệ người dùng quay lại sau ngày truy cập đầu tiên (8.42% - 1.146 người) |
| `Monthly_CRR` | Fact (Retention) | Customer Retention Rate theo chu kỳ tháng | Tỷ lệ giữ chân người dùng dài hạn trên hệ thống (2.39%) |
| `Visitor_to_Lead_Rate` | Fact (Conversion) | `COUNT(DISTINCT Email) / total_user` | Tỷ lệ chuyển đổi từ khách vãng lai sang khách tiềm năng có email (14.45%) |
| `Lead_to_Buyer_Rate` | Fact (Conversion) | `COUNT(DISTINCT Email có Completed Order) / COUNT(DISTINCT Email)` | Tỷ lệ chuyển đổi từ khách hàng tiềm năng sang người mua thực tế (16.33%) |
| `AddToCart_To_Purchase_Rate` | Fact (Conversion) | Số người hoàn tất `Completed Order` / Số người thêm vào giỏ | Tỷ lệ thanh toán giỏ hàng thành công (5.37% - rơi rụng 94.63%) |
| `NumberOfBuyers` | Fact (Sales) | `COUNT(DISTINCT Email)` có `Name = 'Completed Order'` | Tổng số khách hàng mua hàng duy nhất (120 khách hàng) |
| `Orders_Per_Buyer` | Fact (Sales) | Tổng đơn `Completed Order` (480) / `NumberOfBuyers` (120) | Số lượng đơn hàng trung bình trên mỗi khách mua (4.00 đơn/khách) |
| `University_Student` | Dimension (Segment)| Phân loại tên miền email `.edu.vn` vs email phổ thông | Nhãn tách biệt tệp Sinh viên đại học (7.1%) và Người dùng phổ thông (92.9%) |
| `Access_type` | Dimension (Topic) | Phân nhóm chủ đề từ `Name` và `ma_path` | Nhãn chủ đề nội dung: Dữ liệu, Young Talent, Uniace, Excel, Khóa học |

