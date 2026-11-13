# BÀI 11: CHUỖI PIPELINE TRÍCH XUẤT ĐÁNH GIÁ NHÀ CUNG CẤP DẠNG JSON SCHEMA

## PHẦN 1. TÌM HIỂU YÊU CẦU BÀI TẬP

### 1.1. Bài tập yêu cầu làm gì?

TechShop có nhiều nhà cung cấp linh kiện máy tính. Sau mỗi cuộc họp, nhân viên ghi lại thông tin như:
- Tỷ lệ chiết khấu.
- Thời hạn thanh toán.
- Chính sách đổi trả.
- Chính sách bảo hành.
- Các mặt hàng cung cấp.

Thông tin thường nằm trong một đoạn văn dài. TechShop muốn sử dụng AI để đọc đoạn văn đó và tự động chuyển thành dữ liệu JSON.

### 1.2. JSON là gì?

JSON là một cách lưu trữ dữ liệu dưới dạng các cặp tên và giá trị.

Ví dụ:

```json
{
  "vendor_name": "Tan Phat",
  "payment_term_days": 30
}
```

Trong đó:
- `vendor_name`: Tên trường dữ liệu.
- `"Tan Phat"`: Giá trị của trường.
- `payment_term_days`: Tên trường dữ liệu.
- `30`: Giá trị dạng số.

Hiểu đơn giản: JSON giúp máy tính đọc và lưu thông tin theo một cấu trúc thống nhất.

### 1.3. JSON Schema là gì?

JSON Schema là bộ quy tắc quy định dữ liệu phải có những trường nào và mỗi trường phải thuộc kiểu dữ liệu nào.

Ví dụ:
- Tên nhà cung cấp phải là chữ (String).
- Tỷ lệ chiết khấu phải là số (Float).
- Thời hạn thanh toán phải là số nguyên (Integer).
- Có hỗ trợ đổi trả hay không phải là đúng/sai (Boolean).

## PHẦN 2. XÂY DỰNG MASTER PROMPT

### 2.1. Master Prompt là gì?

Master Prompt là một câu lệnh đầy đủ, được viết sẵn để AI thực hiện một nhiệm vụ theo những quy tắc nhất định.

Trong bài này, Master Prompt yêu cầu AI đọc biên bản đàm phán và chuyển thông tin thành JSON đúng cấu trúc.

### 2.2. Master Prompt hoàn chỉnh

```text
VAI TRÒ:
Bạn là trợ lý AI chuyên đọc và trích xuất thông tin
nhà cung cấp cho công ty TechShop.

NHIỆM VỤ:
Đọc biên bản đàm phán và lấy ra các thông tin:
1. Tên nhà cung cấp.
2. Tỷ lệ chiết khấu.
3. Số ngày thanh toán.
4. Có hỗ trợ đổi trả trực tiếp hay không.
5. Danh sách sản phẩm cung cấp.
6. Mức độ rủi ro.

VĂN BẢN ĐẦU VÀO:
Chỉ đọc nội dung nằm giữa hai dấu phân cách sau:

<<<BAT_DAU_BIEN_BAN>>>
[Nội dung biên bản đàm phán]
<<<KET_THUC_BIEN_BAN>>>

CẤU TRÚC JSON BẮT BUỘC:

{
  "vendor_name": "",
  "discount_rate": 0.0,
  "payment_term_days": 0,
  "supports_rma_direct": false,
  "core_product_lines": [],
  "risk_assessment": ""
}

QUY TẮC:
1. Chỉ lấy thông tin có trong biên bản.
2. Không tự bịa hoặc suy đoán dữ liệu.
3. discount_rate phải là số, bỏ ký hiệu %.
4. payment_term_days phải là số nguyên.
5. supports_rma_direct chỉ được dùng true hoặc false.
6. core_product_lines phải là danh sách các chuỗi văn bản.
7. risk_assessment chỉ được chọn:
   Low, Medium hoặc High.

NẾU THIẾU THÔNG TIN:
- String: ""
- Float: 0.0
- Integer: 0
- Boolean: false
- Array: []

ĐÁNH GIÁ RỦI RO:
- Low: Điều khoản rõ ràng, ít rủi ro.
- Medium: Có một số điều khoản chưa thuận lợi.
- High: Có nhiều điều khoản bất lợi hoặc rủi ro.

YÊU CẦU ĐẦU RA:
- Chỉ xuất JSON.
- Không thêm lời giải thích.
- Không thêm Markdown hoặc dấu ```json.
- Không thêm trường dữ liệu ngoài quy định.
- Không thêm dấu phẩy thừa.
- Kiểm tra lại cú pháp JSON trước khi trả lời.
```

### 2.3. Giải thích các thành phần trong Prompt

| Thành phần | Ý nghĩa |
|---|---|
| Vai trò | Cho AI biết mình cần đóng vai gì. |
| Nhiệm vụ | Nói rõ AI cần thực hiện công việc gì. |
| Delimiters | Dùng dấu phân cách để xác định đoạn văn cần đọc. |
| JSON Schema | Quy định cấu trúc dữ liệu đầu ra. |
| Quy tắc | Hướng dẫn AI chuyển đổi dữ liệu. |
| Dữ liệu thiếu | Quy định giá trị thay thế khi không tìm thấy thông tin. |
| Yêu cầu đầu ra | Đảm bảo kết quả là JSON sạch, không có nội dung thừa. |

## PHẦN 3. TÌM HIỂU CÁC KIỂU DỮ LIỆU

Đề bài yêu cầu sử dụng 5 kiểu dữ liệu cơ bản:

| Kiểu dữ liệu | Giải thích | Ví dụ |
|---|---|---|
| String | Dữ liệu dạng chữ | "Tan Phat" |
| Float | Số có phần thập phân | 12.5 |
| Integer | Số nguyên | 30 |
| Boolean | Giá trị đúng hoặc sai | true |
| Array | Danh sách nhiều giá trị | ["RAM", "SSD"] |

**Ví dụ dễ hiểu:**

- Chiết khấu 12.5% → ghi `12.5`, không ghi `"12.5%"`.
- Thanh toán sau 30 ngày → ghi `30`.
- Có hỗ trợ đổi trả → ghi `true`.
- Không hỗ trợ đổi trả → ghi `false`.
- Cung cấp RAM và SSD → ghi `["RAM", "SSD"]`.

Lưu ý: String và Array dùng dấu ngoặc kép cho nội dung chữ. Boolean và số không đặt trong dấu ngoặc kép.

## PHẦN 4. TẠO BIÊN BẢN ĐÀM PHÁN MẪU

### 4.1. Thông tin cuộc họp

- Công ty: TechShop.
- Nhà cung cấp: Viễn Thông Tân Phát.
- Nội dung: Đàm phán hợp tác cung cấp linh kiện máy tính.

### 4.2. Biên bản đàm phán

TechShop tiến hành cuộc họp với nhà phân phối Viễn Thông Tân Phát để thảo luận về các điều khoản hợp tác.

Hai bên thống nhất mức chiết khấu là 12.5% trên tổng giá trị đơn hàng.

Thời hạn thanh toán là 30 ngày kể từ ngày TechShop nhận đủ hàng và hóa đơn hợp lệ.

Đối với sản phẩm bị lỗi do nhà sản xuất, Tân Phát cam kết hỗ trợ đổi trả trực tiếp.

Nhà cung cấp cam kết bảo hành 36 tháng đối với sản phẩm RAM và SSD. Các sản phẩm mainboard và card đồ họa áp dụng chính sách bảo hành của hãng.

Các ngành hàng cung cấp chính gồm:
- RAM.
- SSD.
- Mainboard.
- Card đồ họa.

Nhìn chung, nhà cung cấp có các điều khoản hợp tác rõ ràng, hỗ trợ đổi trả trực tiếp và có chính sách bảo hành. Tuy nhiên, một số sản phẩm vẫn phụ thuộc vào chính sách của hãng.

## PHẦN 5. CHUYỂN BIÊN BẢN THÀNH JSON

### 5.1. Phân tích thông tin

Đọc biên bản ở Phần 4 và lấy từng thông tin tương ứng:

| Trường JSON | Thông tin lấy được |
|---|---|
| vendor_name | Viễn Thông Tân Phát |
| discount_rate | 12.5 |
| payment_term_days | 30 |
| supports_rma_direct | true |
| core_product_lines | RAM, SSD, Mainboard, Card đồ họa |
| risk_assessment | Low |

### 5.2. Kết quả JSON

```json
{
  "vendor_name": "Viễn Thông Tân Phát",
  "discount_rate": 12.5,
  "payment_term_days": 30,
  "supports_rma_direct": true,
  "core_product_lines": [
    "RAM",
    "SSD",
    "Mainboard",
    "Card đồ họa"
  ],
  "risk_assessment": "Low"
}
```

### 5.3. Giải thích kết quả

- `vendor_name`: Tên nhà cung cấp được lấy từ biên bản.
- `discount_rate`: Mức chiết khấu 12.5%.
- `payment_term_days`: Thời hạn thanh toán 30 ngày.
- `supports_rma_direct`: true vì có hỗ trợ đổi trả trực tiếp.
- `core_product_lines`: Danh sách 4 ngành hàng cung cấp.
- `risk_assessment`: Low vì các điều khoản cơ bản rõ ràng và có cam kết đổi trả, bảo hành.

## PHẦN 6. KIỂM TRA LỖI JSON

### 6.1. Những lỗi cần tránh

1. Thiếu dấu ngoặc kép ở tên trường.
2. Viết sai kiểu dữ liệu, ví dụ `"30"` thay vì `30`.
3. Thêm dấu phẩy sau phần tử cuối cùng.
4. Thêm nội dung giải thích bên ngoài JSON.
5. Tự ý thêm trường không có trong Schema.

### 6.2. Kiểm tra JSON bằng Python

Sử dụng đoạn code sau để kiểm tra JSON có đúng cú pháp hay không:

```python
import json

data = '''
{
  "vendor_name": "Viễn Thông Tân Phát",
  "discount_rate": 12.5,
  "payment_term_days": 30,
  "supports_rma_direct": true,
  "core_product_lines": ["RAM", "SSD", "Mainboard", "Card đồ họa"],
  "risk_assessment": "Low"
}
'''

try:
    result = json.loads(data)
    print("JSON hợp lệ!")
except json.JSONDecodeError:
    print("JSON không hợp lệ!")
```

Kết quả:

```text
JSON hợp lệ!
```

Giải thích:
- `json.loads()` dùng để đọc và chuyển chuỗi JSON thành dữ liệu Python.
- Nếu JSON đúng cú pháp, chương trình in ra JSON hợp lệ.
- Nếu sai cú pháp, chương trình in ra JSON không hợp lệ.

Lưu ý: Kiểm tra cú pháp JSON chưa đồng nghĩa với việc đã kiểm tra đầy đủ JSON Schema. Muốn kiểm tra đúng tên trường và kiểu dữ liệu, cần có thêm bước kiểm tra Schema.

## PHẦN 7. KẾT LUẬN

Qua bài tập, em hiểu được cách sử dụng AI để chuyển thông tin từ văn bản dài thành dữ liệu JSON có cấu trúc.

Những kiến thức đã thực hành:
1. Hiểu JSON và JSON Schema.
2. Biết cách viết Master Prompt.
3. Biết sử dụng Delimiters để phân tách văn bản.
4. Phân biệt String, Float, Integer, Boolean và Array.
5. Biết cách xử lý khi dữ liệu bị thiếu.
6. Biết kiểm tra lỗi cú pháp JSON bằng Python.
