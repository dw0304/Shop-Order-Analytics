# TikTok Shop Order Analytics — Dashboard v2

Streamlit MVP for TikTok Shop All Order exports.

## Business rules
- 1 SKU row = 1 order record.
- Do not deduplicate Order ID.
- `Cancelation/Return Type = Return/Refund` => Refund.
- `Cancelation/Return Type = Cancel` => Cancel.
- Cancel + Tracking ID is a separate `Tracked Cancel` operational bucket; it is **not** counted as Refund.
- GMV = SUM of `SKU Subtotal After Discount` at SKU-line level.
- `Order Amount` is not summed because it can repeat across SKU lines for the same Order ID.
- Net GMV = Gross GMV - Refund GMV - Cancel GMV.
- Refund Type (System/Seller/Buyer) is not inferred unless a reliable source field exists.

## Run on Windows

```bat
pip install -r requirements.txt
streamlit run app.py
```
# TIKTOK SHOP — MONTHLY SALES, CANCEL & REFUND ANALYZER

Bạn là chuyên gia Data Analyst chuyên phân tích dữ liệu TikTok Shop / E-commerce.

Tôi sẽ upload file Excel/CSV export từ TikTok Shop Seller Center.

Hãy đọc toàn bộ dữ liệu và tạo báo cáo phân tích theo THÁNG, tập trung vào:

- Orders
- Sales / Actual Revenue
- Cancel
- Refund / Return
- Net Revenue
- Product / SKU performance

==================================================
1. XỬ LÝ FILE
==================================================

File TikTok Shop có thể có cấu trúc Excel/XML không chuẩn.

Nếu pandas hoặc thư viện Excel thông thường không đọc đúng toàn bộ dữ liệu, hãy đọc trực tiếp cấu trúc XML của workbook để khôi phục đầy đủ các cột.

Không được kết luận file thiếu dữ liệu chỉ vì cách đọc Excel thông thường không hiển thị đầy đủ cột.

Các cột quan trọng thường gồm:

- Order ID
- Order Status
- Order Substatus
- Cancelation/Return Type
- SKU ID
- Seller SKU
- Product Name
- Variation
- Quantity
- Sku Quantity of return
- Order Amount
- Order Refund Amount
- Created Time
- Cancel By
- Cancel Reason

Nếu tên cột khác một chút, hãy tự nhận diện cột tương ứng.

==================================================
2. ORDER LEVEL VS SKU LEVEL
==================================================

Một Order ID có thể xuất hiện nhiều dòng vì một đơn có nhiều SKU.

### ORDER LEVEL

Khi tính:

- Tổng đơn
- Cancel Orders
- Paid Orders
- Doanh thu
- Refund Orders
- Refund Amount

mỗi Order ID chỉ được tính MỘT LẦN.

Không cộng Order Amount hoặc Order Refund Amount nhiều lần nếu một Order ID có nhiều dòng.

### SKU LEVEL

Khi tính:

- Tổng SP
- Quantity
- Sku Quantity of return
- Product
- SKU
- Product performance

tính theo từng dòng SKU.

==================================================
3. XÁC ĐỊNH THÁNG
==================================================

Dùng:

Created Time

để xác định tháng.

Ví dụ:

01/07/2026 → 07/2026
31/07/2026 → 07/2026
01/08/2026 → 08/2026

Tự động phân tích TẤT CẢ các tháng có trong file.

Mỗi tháng = 1 dòng.

Mỗi KPI = 1 cột.

==================================================
4. LOGIC QUAN TRỌNG NHẤT — CANCEL ≠ REFUND
==================================================

PHẢI tách hoàn toàn:

CANCEL

và

REFUND / RETURN

Không được gộp hai loại này thành một.

--------------------------------------------------
4.1 CANCEL
--------------------------------------------------

Nếu:

Order Substatus = Cancel

thì xác định đây là một đơn CANCEL.

Đơn Cancel:

- Vẫn được tính vào Tổng đơn
- Được tính vào Cancel Orders
- Được tính vào Cancel Rate

NHƯNG:

- Không tính vào Paid Orders
- Không tính vào Actual Revenue
- Không tính vào Refund Orders
- Không tính vào Refund Amount
- Không tính vào Refund Rate

Lý do:

Đơn Cancel được xem là đơn khách chưa thực sự thanh toán và shop không thực sự thu khoản doanh thu đó.

--------------------------------------------------
4.2 PAID ORDERS
--------------------------------------------------

Paid Orders =

Tổng đơn − Cancel Orders

Chỉ Paid Orders mới được dùng để tính:

- Actual Revenue
- Refund
- Refund Rate
- Net Revenue

--------------------------------------------------
4.3 ACTUAL REVENUE
--------------------------------------------------

Actual Revenue =

SUM(Order Amount)

của các Order ID KHÔNG có:

Order Substatus = Cancel

Không được cộng Order Amount của Cancel Orders vào doanh thu.

--------------------------------------------------
4.4 REFUND ORDERS
--------------------------------------------------

Refund Order là Paid Order có:

Order Refund Amount > 0

Không được coi Cancel Order là Refund Order dù Order Refund Amount có giá trị.

Refund Orders chỉ được tính trên Paid Orders.

--------------------------------------------------
4.5 ACTUAL REFUND AMOUNT
--------------------------------------------------

Actual Refund Amount =

SUM(Order Refund Amount)

chỉ của các Paid Orders.

Không lấy Refund Amount của Cancel Orders.

Mỗi Order ID chỉ được tính một lần.

--------------------------------------------------
4.6 REFUND RATE
--------------------------------------------------

Refund Rate =

Refund Orders / Paid Orders × 100

KHÔNG dùng:

Refund Orders / Total Orders

vì Total Orders bao gồm cả Cancel.

--------------------------------------------------
4.7 REFUND / DOANH THU THÁNG
--------------------------------------------------

Đây là KPI rất quan trọng.

Refund / DT tháng =

Actual Refund Amount / Actual Revenue × 100

Mẫu số phải là:

DOANH THU THỰC TẾ CỦA CHÍNH THÁNG ĐÓ.

Không dùng tổng doanh thu của toàn bộ file.

--------------------------------------------------
4.8 NET REVENUE
--------------------------------------------------

Net Revenue =

Actual Revenue − Actual Refund Amount

Đây KHÔNG phải Profit.

Không gọi là Profit nếu file không có:

- COGS
- Product Cost
- TikTok Fees
- Shipping Cost
- Advertising Cost
- Other Expenses

Hãy gọi:

Net Revenue sau Refund

==================================================
5. BẢNG THỐNG KÊ CHÍNH
==================================================

Tạo bảng theo chiều ngang:

| Tháng | Tổng đơn | Tổng SP | Doanh thu thực tế | Đơn Cancel | Cancel Rate | Paid Orders | Đơn Refund | Refund Rate | SP Refund | SP Refund Rate | Refund Amount | Refund / DT tháng | Net Revenue | Refund toàn bộ | Refund một phần |

Mỗi tháng = một dòng.

Sắp xếp tháng tăng dần.

==================================================
6. ĐỊNH NGHĨA CÁC KPI
==================================================

### Tổng đơn

Số Order ID duy nhất.

Bao gồm cả:

- Paid
- Cancel

### Tổng SP

SUM Quantity của tất cả SKU.

### Doanh thu thực tế

SUM Order Amount của Paid Orders.

Không tính Cancel Orders.

### Đơn Cancel

Số Order ID có:

Order Substatus = Cancel

### Cancel Rate

Cancel Orders / Total Orders × 100

### Paid Orders

Total Orders − Cancel Orders

### Đơn Refund

Số Paid Orders có:

Order Refund Amount > 0

### Refund Rate

Refund Orders / Paid Orders × 100

### SP Refund

SUM Sku Quantity of return

Chỉ dùng dữ liệu return/refund thực tế.

### SP Refund Rate

SP Refund / Tổng SP của Paid Orders × 100

Nếu không thể xác định chính xác Quantity thuộc Paid Orders thì ghi rõ giới hạn dữ liệu.

### Refund Amount

SUM Order Refund Amount của Paid Orders.

### Refund / DT tháng

Refund Amount / Actual Revenue × 100

### Net Revenue

Actual Revenue − Refund Amount

### Refund toàn bộ

Paid Orders có:

Order Refund Amount ≈ Order Amount

### Refund một phần

Paid Orders có:

0 < Order Refund Amount < Order Amount

==================================================
7. CANCEL VÀ REFUND PHẢI ĐƯỢC PHÂN TÍCH RIÊNG
==================================================

Sau bảng chính, tạo:

## CANCEL ANALYSIS

| Tháng | Tổng đơn | Cancel Orders | Cancel Rate | Paid Orders |
|---|---:|---:|---:|---:|

Nếu dữ liệu có:

- Cancel By
- Cancel Reason

hãy phân tích thêm nguyên nhân Cancel.

--------------------------------------------------

## REFUND ANALYSIS

| Tháng | Paid Orders | Refund Orders | Refund Rate | Refund Amount | Refund / DT tháng |
|---|---:|---:|---:|---:|---:|

==================================================
8. PRODUCT / SKU ANALYSIS
==================================================

Phân tích sản phẩm dựa trên Paid Orders.

Tạo:

| Rank | Product | Seller SKU | Sold | Refund/Return | Return Rate | Revenue | Refund Amount | Net Revenue | Status |
|---:|---|---|---:|---:|---:|---:|---:|---:|---|

Return Rate:

Refund/Return Quantity / Sold Quantity × 100

Không dùng Cancel Orders để tính Return Rate.

==================================================
9. PRODUCT STATUS
==================================================

### 🟢 SCALE

Sales cao + Return thấp + Refund thấp.

### 🟡 REVIEW

Sales tốt nhưng Return/Refund bắt đầu cao.

### 🔴 STOP / REVIEW

Return Rate > 40%

hoặc Refund Amount rất cao

hoặc doanh thu cao nhưng phần lớn doanh thu bị refund.

Không được kết luận chỉ dựa trên tỷ lệ nếu sample quá nhỏ.

Ví dụ:

1 sold / 1 return = 100%

không được đánh giá giống:

50 sold / 40 return = 80%.

==================================================
10. MONTHLY TREND
==================================================

Phân tích theo tháng:

- Tổng đơn
- Cancel Rate
- Paid Orders
- Actual Revenue
- Refund Rate
- Refund Amount
- Refund / DT tháng
- Net Revenue

Xác định:

- Tháng tốt nhất
- Tháng tệ nhất
- Tháng có Cancel cao nhất
- Tháng có Refund cao nhất
- Tháng có Actual Revenue cao nhất
- Tháng có Net Revenue cao nhất

Đặc biệt theo dõi:

Refund / DT tháng

để đánh giá chất lượng doanh thu.

==================================================
11. BIỂU ĐỒ
==================================================

Nếu dữ liệu có từ 3 tháng trở lên, tạo biểu đồ đường cho:

### Chart 1
Refund / DT tháng theo thời gian.

### Chart 2
Actual Revenue theo tháng.

### Chart 3
Cancel Rate và Refund Rate theo tháng.

Nếu biểu đồ có nhiều series khác đơn vị, không đặt chung một trục Y nếu gây hiểu nhầm.

==================================================
12. THÁNG CHƯA HOÀN CHỈNH
==================================================

Nếu tháng cuối cùng chưa kết thúc, phải ghi rõ.

Ví dụ:

"08/2026 mới có dữ liệu đến 23/08/2026."

Không được kết luận tháng chưa hoàn chỉnh tốt/xấu hơn các tháng đầy đủ mà không cảnh báo.

==================================================
13. CẢNH BÁO
==================================================

### CANCEL

🔴 CRITICAL:
Cancel Rate > 40%

🟠 HIGH:
Cancel Rate 25–40%

🟡 WATCH:
Cancel Rate 15–25%

🟢 GOOD:
Cancel Rate < 15%

### REFUND

🔴 CRITICAL:
Refund / DT tháng > 40%

🟠 HIGH:
25–40%

🟡 WATCH:
15–25%

🟢 GOOD:
<15%

Đây là ngưỡng phân tích nội bộ, không phải quy định chính thức của TikTok Shop.

==================================================
14. KẾT LUẬN
==================================================

Cuối báo cáo phải trả lời:

1. Tổng volume đơn hàng là bao nhiêu?
2. Bao nhiêu đơn bị Cancel?
3. Cancel Rate bao nhiêu?
4. Doanh thu thực tế bao nhiêu?
5. Bao nhiêu đơn thực sự Refund?
6. Refund Rate trên Paid Orders là bao nhiêu?
7. Bao nhiêu tiền thực sự bị Refund?
8. Refund chiếm bao nhiêu % doanh thu tháng?
9. Net Revenue sau Refund là bao nhiêu?
10. Vấn đề chính của shop nằm ở Cancel hay Refund?
11. SKU nào đáng SCALE?
12. SKU nào cần REVIEW?
13. SKU nào nên STOP?

==================================================
15. QUY TẮC KIỂM TRA CUỐI
==================================================

Trước khi trả kết quả, bắt buộc kiểm tra:

1. Order ID có bị đếm trùng không?
2. Order Amount có bị cộng nhiều lần không?
3. Order Refund Amount có bị cộng nhiều lần không?
4. Cancel Orders có bị đưa vào Revenue không?
5. Cancel Orders có bị đưa vào Refund không?
6. Refund Rate có dùng Paid Orders làm mẫu số không?
7. Refund / DT tháng có dùng Actual Revenue của chính tháng đó không?
8. Net Revenue = Actual Revenue − Actual Refund Amount?
9. Cancel và Refund đã tách riêng chưa?
10. Tháng cuối có phải tháng chưa hoàn chỉnh không?

Nếu phát hiện sai, phải sửa trước khi đưa kết quả.

==================================================
MỤC TIÊU
==================================================

Báo cáo phải giúp tôi nhìn được rõ chuỗi:

TỔNG ĐƠN
↓
CANCEL
↓
PAID ORDERS
↓
REFUND
↓
ACTUAL REVENUE
↓
NET REVENUE

Mục tiêu cuối cùng:

"Shop nhận được bao nhiêu đơn?"

"Bao nhiêu đơn bị Cancel?"

"Bao nhiêu đơn thực sự được thanh toán?"

"Shop thực sự tạo ra bao nhiêu doanh thu?"

"Bao nhiêu tiền thực sự bị Refund?"

"Refund đang ăn bao nhiêu % doanh thu?"

"Cuối cùng shop còn lại bao nhiêu Net Revenue?"

"SKU nào đang đáng scale và SKU nào đang đốt doanh thu?"
