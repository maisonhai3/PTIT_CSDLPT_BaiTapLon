# PHẦN VI: MÔ HÌNH ERD — Mô tả các thực thể

## 1. Site — Trung tâm vùng

Đại diện cho một vùng địa lý vận hành độc lập trong hệ thống (Bắc / Trung / Nam).  
Mỗi Site là một node cơ sở dữ liệu riêng biệt trong kiến trúc phân tán.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| SiteID | char(1) | PK | Mã vùng: 'N' (Bắc), 'C' (Trung), 'S' (Nam) |
| SiteName | nvarchar(50) | NOT NULL | Tên vùng (ví dụ: Miền Bắc) |

---

## 2. Hub — Kho / Trung tâm trung chuyển

Đại diện cho một kho hoặc trung tâm trung chuyển thuộc một Site.  
Hub là đơn vị vật lý nơi hàng hoá được nhận, lưu kho, xuất kho và trung chuyển.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| HubID | int | PK, IDENTITY | Mã hub (tự tăng) |
| SiteID | char(1) | FK → Site | Vùng sở hữu hub |
| HubName | nvarchar(100) | NOT NULL | Tên hub/kho |
| Address | nvarchar(200) | NULL | Địa chỉ vật lý |

---

## 3. Employee — Nhân viên

Nhân viên làm việc tại một Hub, đảm nhận các vai trò khác nhau trong quy trình nghiệp vụ.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| EmployeeID | int | PK, IDENTITY | Mã nhân viên |
| HubID | int | FK → Hub | Hub công tác |
| FullName | nvarchar(100) | NOT NULL | Họ và tên |
| Role | varchar(30) | NOT NULL, CHECK | Vai trò: CLERK / WAREHOUSE / DISPATCH / SHIPPER / ADMIN |
| Phone | varchar(15) | UNIQUE | Số điện thoại |

---

## 4. Customer — Khách hàng (Người gửi)

Khách hàng là người gửi hàng, tạo vận đơn trong hệ thống.  
Có thể gộp vai trò Sender vào thực thể này.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| CustomerID | int | PK, IDENTITY | Mã khách hàng |
| FullName | nvarchar(100) | NOT NULL | Họ và tên |
| Phone | varchar(15) | UNIQUE | Số điện thoại đăng ký |
| Address | nvarchar(200) | NULL | Địa chỉ mặc định |

---

## 5. Shipment — Vận đơn

Đơn vị giao dịch trung tâm của hệ thống. Mỗi vận đơn đại diện cho một yêu cầu giao hàng từ người gửi đến người nhận.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| ShipmentID | varchar(32) | PK | Mã vận đơn |
| OwningSiteID | char(1) | CHECK (N/C/S) | Site sở hữu vận đơn |
| SenderCustomerID | int | FK → Customer | Khách hàng gửi hàng |
| ReceiverName | nvarchar(100) | NOT NULL | Tên người nhận |
| ReceiverPhone | varchar(15) | NOT NULL | SĐT người nhận |
| ReceiverAddress | nvarchar(200) | NOT NULL | Địa chỉ giao hàng |
| OriginHubID | int | FK → Hub | Hub xuất phát |
| DestHubID | int | FK → Hub | Hub đích |
| CurrentStatus | varchar(30) | FK → StatusCode | Trạng thái hiện tại |
| CurrentHubID | int | FK → Hub, NULL | Hub hiện tại của kiện |
| CODAmount | decimal(18,2) | CHECK ≥ 0 | Tiền thu hộ (COD) |
| CreatedAt | datetime2 | NOT NULL | Thời điểm tạo đơn |
| UpdatedAt | datetime2 | NOT NULL | Thời điểm cập nhật cuối |
| RowVer | rowversion | — | Phục vụ optimistic concurrency |

---

## 6. Parcel — Kiện hàng

Một vận đơn có thể chứa nhiều kiện hàng vật lý. Mỗi kiện có khối lượng và mã vạch riêng.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| ParcelID | varchar(40) | PK | Mã kiện hàng |
| ShipmentID | varchar(32) | FK → Shipment | Vận đơn chứa kiện |
| WeightKg | decimal(10,2) | CHECK > 0 | Khối lượng (kg) |
| Barcode | varchar(40) | UNIQUE | Mã vạch kiện |

---

## 7. StatusCode — Mã trạng thái

Danh mục các trạng thái có thể có của vận đơn trong vòng đời vận chuyển.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| StatusCode | varchar(30) | PK | Mã trạng thái |
| Description | nvarchar(100) | NOT NULL | Mô tả (tiếng Việt) |

**Các giá trị chuẩn:**

| Mã | Mô tả |
|---|---|
| CREATED | Tạo vận đơn |
| IN_HUB | Đang ở kho/hub |
| IN_TRANSIT | Đang trung chuyển liên vùng |
| OUT_FOR_DELIVERY | Đang giao chặng cuối |
| DELIVERED | Giao thành công |
| FAILED | Giao thất bại |
| RETURNED | Hoàn hàng |

---

## 8. ShipmentStatusHistory — Lịch sử trạng thái vận đơn

Ghi lại toàn bộ các thay đổi trạng thái của vận đơn theo thời gian tại từng Hub.  
Đây là bảng append-only, không xoá hay sửa bản ghi cũ.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| HistoryID | bigint | PK, IDENTITY | Mã bản ghi |
| ShipmentID | varchar(32) | FK → Shipment | Vận đơn |
| StatusCode | varchar(30) | FK → StatusCode | Trạng thái ghi nhận |
| HubID | int | FK → Hub | Hub ghi nhận sự kiện |
| EventTime | datetime2 | NOT NULL | Thời điểm xảy ra |
| Note | nvarchar(200) | NULL | Ghi chú thêm |

---

## 9. ScanEvent — Sự kiện quét (Scan)

Ghi lại mọi sự kiện quét mã vạch/vận đơn tại Hub. Đây là nguồn sự kiện bất biến (immutable event log) dùng để đồng bộ dữ liệu giữa các site trong kiến trúc phân tán.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| EventID | uniqueidentifier | PK | UUID của sự kiện |
| ShipmentID | varchar(32) | FK → Shipment | Vận đơn liên quan |
| HubID | int | FK → Hub | Hub xảy ra sự kiện |
| EventType | varchar(30) | NOT NULL | Loại sự kiện: INBOUND / OUTBOUND / LOAD / UNLOAD / DELIVER / FAIL |
| EventTime | datetime2 | NOT NULL | Thời điểm quét |
| OperatorID | int | FK → Employee, NULL | Nhân viên thực hiện |
| SourceSiteID | char(1) | CHECK (N/C/S) | Site nguồn phát sinh |
| SourceSeq | bigint | NOT NULL | Số thứ tự sự kiện theo site (unique/site) |
| PayloadJson | nvarchar(max) | NULL | Dữ liệu mở rộng dạng JSON |

> **Chỉ mục:** UNIQUE(SourceSiteID, SourceSeq) đảm bảo idempotent khi đồng bộ giữa các site.

---

## 10. Trip — Chuyến vận chuyển

Đại diện cho một chuyến xe/vận chuyển liên Hub, thường giữa hai Hub khác vùng (liên vùng Bắc–Trung–Nam).

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| TripID | bigint | PK, IDENTITY | Mã chuyến |
| FromHubID | int | FK → Hub | Hub xuất phát |
| ToHubID | int | FK → Hub | Hub đích |
| DepartTime | datetime2 | NULL | Giờ khởi hành thực tế |
| Status | varchar(20) | CHECK | Trạng thái: PLANNED / DEPARTED / ARRIVED |

---

## 11. TripItem — Chi tiết chuyến (Bảng trung gian Trip – Shipment)

Giải bài toán quan hệ N–N giữa Trip và Shipment: một chuyến chứa nhiều vận đơn, một vận đơn có thể trải qua nhiều chuyến.

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| TripItemID | bigint | PK, IDENTITY | Mã bản ghi |
| TripID | bigint | FK → Trip | Chuyến vận chuyển |
| ShipmentID | varchar(32) | FK → Shipment | Vận đơn trong chuyến |
| AddedAt | datetime2 | NOT NULL | Thời điểm gán vận đơn vào chuyến |

> **Chỉ mục:** UNIQUE(TripID, ShipmentID) ngăn gán trùng vận đơn vào cùng một chuyến.

---

## 12. DeliveryAssignment — Phân công giao hàng chặng cuối

Ghi lại lịch sử phân công giao hàng cho Shipper. Một vận đơn có thể được phân công lại nhiều lần (giao thất bại → giao lại).

| Thuộc tính | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| AssignmentID | bigint | PK, IDENTITY | Mã phân công |
| ShipmentID | varchar(32) | FK → Shipment | Vận đơn được phân công |
| ShipperID | int | FK → Employee | Shipper được giao |
| AssignedAt | datetime2 | NOT NULL | Thời điểm phân công |
| ResultStatus | varchar(30) | NULL | Kết quả: DELIVERED / FAILED |
| ResultTime | datetime2 | NULL | Thời điểm có kết quả |

---

## Tổng hợp quan hệ (Crow's Foot)

| # | Quan hệ | Cardinality | Ghi chú |
|---|---|---|---|
| A | Site → Hub | 1 – N | Một Site có nhiều Hub |
| B | Hub → Employee | 1 – N | Một Hub có nhiều nhân viên |
| C | Customer → Shipment | 1 – N | Một khách gửi tạo nhiều vận đơn |
| D | Shipment → Parcel | 1 – N | Một vận đơn có nhiều kiện hàng |
| E | Shipment → ShipmentStatusHistory | 1 – N | Vận đơn có nhiều lịch sử trạng thái |
| F | StatusCode → ShipmentStatusHistory | 1 – N | Mỗi lịch sử tham chiếu một mã trạng thái |
| G | Shipment → ScanEvent | 1 – N | Vận đơn có nhiều sự kiện quét |
| H | Hub → ScanEvent | 1 – N | Sự kiện quét xảy ra tại một Hub |
| I | Trip → TripItem | 1 – N | Một chuyến có nhiều TripItem |
| J | Shipment → TripItem | 1 – N | Một vận đơn xuất hiện trong nhiều chuyến |
| K | Shipment → DeliveryAssignment | 1 – N (0..∞) | Một vận đơn có thể được phân công giao nhiều lần |
| L | Employee → DeliveryAssignment | 1 – N | Một shipper nhận nhiều phân công |
