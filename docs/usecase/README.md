# Bộ Hồ Sơ Thiết Kế Sơ Đồ Use Case & Phân Cấp Chức Năng (PlantUML) — HP STORE

Tài liệu này tổng hợp toàn bộ mã nguồn biểu đồ **PlantUML (`.puml`)** chuẩn UML dành cho dự án Quản lý bán hàng & Chuỗi cửa hàng HP STORE, được chuẩn hóa theo đúng mô hình **4 Tác nhân (Actors)** nghiệp vụ và 14 bảng cơ sở dữ liệu PostgreSQL thực tế.

---

## 1. Danh Mục Các File Mã Nguồn PlantUML Trong Thư Mục

| STT | Tên File | Loại Sơ Đồ | Mô Tả & Tác Nhân Tham Gia |
| :---: | :--- | :--- | :--- |
| **1** | [`usecase-overview.puml`](file:///c:/QLBH_project/docs/usecase/usecase-overview.puml) | **Use Case Tổng Quan** | Toàn bộ ranh giới hệ thống, 4 Actor (Khách hàng, Nhân viên, Quản lý chi nhánh, Quản lý tổng), quan hệ kế thừa và 6 phân hệ cốt lõi. |
| **2** | [`usecase-subsystem-sales-checkout.puml`](file:///c:/QLBH_project/docs/usecase/usecase-subsystem-sales-checkout.puml) | **Phân Hệ 1: Bán Hàng & Đặt Hàng** | Duyệt hàng, ảnh 5 góc, giỏ hàng, đặt hàng (SELECT FOR UPDATE), trừ điểm HP Member, thanh toán VNPAY QR / COD, hủy đơn hoàn tồn. |
| **3** | [`usecase-subsystem-order-fulfillment.puml`](file:///c:/QLBH_project/docs/usecase/usecase-subsystem-order-fulfillment.puml) | **Phân Hệ 2: Xử Lý Đơn Hàng** | Tiếp nhận đơn, cập nhật trạng thái (Chờ xử lý $\rightarrow$ Đang giao $\rightarrow$ Hoàn tất), duyệt hàng loạt, in phiếu giao hàng, hủy đơn. |
| **4** | [`usecase-subsystem-warehouse-multibranch.puml`](file:///c:/QLBH_project/docs/usecase/usecase-subsystem-warehouse-multibranch.puml) | **Phân Hệ 3: Kho Đa Chi Nhánh** | Quản lý kho chi nhánh, nhập kho (loaiGD=1), xuất kho bán lẻ (loaiGD=2), lập & duyệt điều chuyển liên kho, sổ cái thẻ kho (`giaodichkho`). |
| **5** | [`usecase-subsystem-catalog-branch-admin.puml`](file:///c:/QLBH_project/docs/usecase/usecase-subsystem-catalog-branch-admin.puml) | **Phân Hệ Quản Trị & Báo Cáo** | Quản lý danh mục & sản phẩm, bảng giá, NCC, quản lý danh sách chi nhánh, phân quyền nhân sự RBAC, báo cáo doanh thu chuỗi. |
| **6** | [`functional-hierarchy-bfd.puml`](file:///c:/QLBH_project/docs/usecase/functional-hierarchy-bfd.puml) | **Phân Cấp Chức Năng (BFD)** | Sơ đồ cây WBS phân rã đầy đủ 3 cấp: **Cấp 0** (Tổng quát), **Cấp 1** (6 Phân hệ), **Cấp 2** (28 Chức năng chi tiết). |

---

## 2. Mô Hình 4 Tác Nhân (Actors) & Cây Kế Thừa Quyền Hạn

```text
[Khách hàng (Customer)] -------- (Tương tác Storefront mua sắm, thanh toán, điểm thưởng)

[Nhân viên (Staff)] ------------ (Bán hàng tại quầy, tra cứu kho CN, xử lý & giao đơn)
         ▲
         │ (kế thừa quyền)
[Quản lý chi nhánh] ------------ (Quản lý kho CN, đề xuất chuyển kho, báo cáo doanh thu CN)
         ▲
         │ (kế thừa quyền)
[Quản lý tổng (Admin)] --------- (Toàn quyền chuỗi: cấu hình chi nhánh, giá, RBAC, duyệt chuyển kho)
```

---

## 3. Bảng Tra Cứu Quan Hệ `<<include>>` và `<<extend>>`

### 3.1 Quan Hệ Bắt Buộc (`<<include>>`)
> *Use Case nguồn không thể hoàn thành nếu thiếu Use Case được include.*

| Use Case Nguồn | Quan Hệ | Use Case Được Include | Nghiệp Vụ Trong Source Code |
| :--- | :---: | :--- | :--- |
| **UC104: Đặt hàng (Checkout)** | `<<include>>` | UC106: Kiểm tra tồn kho khả dụng | Transaction DB khóa dòng `SELECT ... FOR UPDATE` tránh bán âm (`checkoutModel.js`). |
| **UC104: Đặt hàng (Checkout)** | `<<include>>` | UC107: Lưu chi tiết đơn hàng | Lưu dữ liệu vào bảng `chitiet_donhang`. |
| **UC104: Đặt hàng (Checkout)** | `<<include>>` | UC108: Khởi tạo phiếu giao hàng | Ghi nhận bản ghi vận chuyển vào bảng `phieugiaohang`. |
| **UC104: Đặt hàng (Checkout)** | `<<include>>` | UC109: Ghi nhận thanh toán | Ghi nhận bản ghi thanh toán vào bảng `thanhtoan`. |
| **UC114: Hủy đơn hàng** | `<<include>>` | UC115: Hoàn trả tồn kho (Restock) | Tự động cộng lại số lượng vào bảng `tonkho` tại chi nhánh xuất. |
| **UC302: Nhập kho hàng hóa** | `<<include>>` | UC308: Ghi sổ cái biến động kho | Lưu vết giao dịch thẻ kho `loaiGD = 1` ghi nhận `tontruoc`, `tonsau`. |
| **UC302: Nhập kho hàng hóa** | `<<include>>` | UC309: Cập nhật số dư `tonkho` | Tăng số lượng tồn kho khả dụng tại chi nhánh. |
| **UC303: Xuất kho bán lẻ** | `<<include>>` | UC310: Kiểm tra tồn khả dụng | Kiểm tra `soluongton >= soluongxuat`, chặn xuất âm. |
| **UC303: Xuất kho bán lẻ** | `<<include>>` | UC308: Ghi sổ cái biến động kho | Lưu vết giao dịch thẻ kho `loaiGD = 2`. |
| **UC305: Duyệt điều chuyển kho** | `<<include>>` | UC308 & UC309 | Ghi 2 dòng thẻ kho (xuất kho A, nhập kho B) và cập nhật số dư 2 kho. |
| **UC402: Thêm mới sản phẩm** | `<<include>>` | UC404: Tải ảnh 5 góc lên Cloudinary | Upload và lưu link CDN ảnh mặt trước, mặt sau, viền, camera, hộp. |

---

### 3.2 Quan Hệ Mở Rộng (`<<extend>>`)
> *Chỉ được kích hoạt khi thỏa mãn điều kiện mở rộng cụ thể (Extension Point).*

| Use Case Mở Rộng | Quan Hệ | Use Case Cơ Sở | Điều Kiện Mở Rộng (Extension Condition) |
| :--- | :---: | :--- | :--- |
| **UC102: Xem chi tiết & 5 góc ảnh** | `<<extend>>` | UC101: Tìm kiếm & Xem SP | Khi người dùng click chọn 1 sản phẩm cụ thể. |
| **UC110: Áp dụng điểm HP Member** | `<<extend>>` | UC104: Đặt hàng (Checkout) | Khách hàng có điểm thưởng $\ge 75$ điểm và tích chọn đổi điểm. |
| **UC111: Thanh toán VNPAY QR** | `<<extend>>` | UC104: Đặt hàng (Checkout) | Khách chọn hình thức thanh toán điện tử qua VNPAY. |
| **UC112: Thanh toán COD** | `<<extend>>` | UC104: Đặt hàng (Checkout) | Khách chọn thanh toán tiền mặt khi nhận hàng. |
| **UC202: Xem chi tiết đơn hàng** | `<<extend>>` | UC201: Xem danh sách đơn | Nhân viên click vào 1 mã đơn hàng trên bảng danh sách. |
| **UC204: Cập nhật hàng loạt** | `<<extend>>` | UC203: Cập nhật trạng thái | Nhân viên tích chọn nhiều checkbox đơn để duyệt cùng lúc. |
| **UC205: In phiếu giao hàng** | `<<extend>>` | UC203: Cập nhật trạng thái | Khi đơn hàng chuyển sang trạng thái "Đang giao". |
| **UC307: Lọc thẻ kho theo SP** | `<<extend>>` | UC306: Tra cứu lịch sử thẻ kho | Khi quản trị viên chọn lọc theo 1 mã sản phẩm cụ thể. |
| **UC311: Cảnh báo tồn tối thiểu** | `<<extend>>` | UC309: Cập nhật số dư tonkho | Khi số dư tồn kho khả dụng sau giao dịch $\le 5$ sản phẩm. |
| **UC408: Phân quyền RBAC** | `<<extend>>` | UC407: Quản lý hồ sơ nhân viên | **Chỉ Quản lý tổng** mới có quyền đổi vai trò hoặc chi nhánh làm việc. |
| **UC412: Xuất dữ liệu báo cáo** | `<<extend>>` | UC410 / UC411: Báo cáo | Khi người dùng bấm nút "Xuất Excel" hoặc "Xuất PDF". |

---

## 4. Hướng Dẫn Xem Và Xuất Ảnh Sơ Đồ

### Cách 1: Xem trực tiếp trên VS Code
1. Cài đặt tiện ích mở rộng **PlantUML** (của tác giả *jebbs*) trong Visual Studio Code.
2. Mở bất kỳ file `.puml` nào trong thư mục này (`docs/usecase/`).
3. Nhấn tổ hợp phím **`Alt + D`** để xem biểu đồ hiển thị trực tiếp.
4. Nhấp chuột phải vào sơ đồ, chọn **Export Current Diagram** $\rightarrow$ chọn định dạng **`png`**, **`svg`** hoặc **`pdf`**.

### Cách 2: Xem và tải ảnh online (Không cần cài đặt phần mềm)
1. Mở file `.puml` bất kỳ, sao chép (Copy) toàn bộ nội dung.
2. Truy cập một trong các trang render PlantUML trực tuyến:
   - [https://www.planttext.com/](https://www.planttext.com/)
   - [https://www.plantuml.com/plantuml/uml/](https://www.plantuml.com/plantuml/uml/)
3. Dán (Paste) nội dung vào khung bên trái. Hệ thống sẽ tự động vẽ sơ đồ vector sắc nét ở khung bên phải, cho phép tải về dưới dạng PNG/SVG/PDF để chèn vào báo cáo Word hoặc Slide thuyết trình.
