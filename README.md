# Tình huống giả định:

**Bảng giá và quy tắc của tiệm nước:**

**Giá ly:** size M = 30.000đ, size L = 38.000đ (tiệm chỉ có 2 size này).

**Topping:** +5.000đ mỗi loại, mỗi ly.

**Mã GIAM10K:** giảm 10.000đ, chỉ khi tiền hàng từ 100.000đ trở lên.

**Giao tận nơi:** phí ship 15.000đ. Uống tại quán: không phí.

## Dữ liệu thử (có cài bẫy):

```
T01 | An   | M | 2 ly | 1 topping | giao tận nơi | không
T02 | Bình | L | 3 ly | 2 topping | giao tận nơi | GIAM10K
T03 | Chi  | S | 1 ly | 0 topping | tại quán     | không
T04 | Dũng | M | 0 ly | 1 topping | tại quán     | không
T05 | Em   | M | 2 ly | 0 topping | tại quán     | GIAM10K
```
**Các bẫy:**
- T03: size S không tồn tại.
- T04: số ly bằng 0.
- T05: có mã nhưng chưa đủ điều kiện 100.000đ.

# Hai Prompt đề xuất (Đơn giản/Chi tiết)

**Prompt 1: Đơn giản**

```
Bạn là nhân viên tiệm trà sữa. Tính tiền cho các đơn bên dưới:
(size M 30.000đ, L 38.000đ, topping 5.000đ, mã GIAM10K giảm 10.000đ,
giao tận nơi phí 15.000đ) rồi viết tin nhắn xác nhận cho từng khách.

Đơn: """[dán 5 đơn]"""
```

**Prompt 2: Chi tiết**

- *Bước 1: Kiểm tra đơn*

```
Vai trò: nhân viên kiểm tra đơn của tiệm trà sữa.
Điều kiện:
- Nếu size không phải M hoặc L thì trạng thái LOI, lý do "không có size này".
- Nếu số ly nhỏ hơn hoặc bằng 0 thì trạng thái LOI, lý do "số ly không hợp lệ".
- Không tự đoán hay sửa dữ liệu.
Yêu cầu: chỉ trả về JSON, không giải thích.
Mẫu: [{"ma_don":"...","trang_thai":"HOP_LE hoặc LOI","ly_do":"..."}]
Đơn: """[dán 5 đơn]"""
```

- *Chốt chặn:* chỉ đơn `HOP_LE` được sang Bước B. Đơn `LOI` gọi lại hỏi khách.

- *Bước 2: Tính tiền và nhắn tin*

```
Vai trò: nhân viên thu ngân tiệm trà sữa.
Với mỗi đơn, hãy suy nghĩ từng bước:
1. Giá 1 ly = giá size + 5.000 x số topping (M = 30.000, L = 38.000).
2. Tiền hàng = giá 1 ly x số ly.
3. Mã GIAM10K: giảm 10.000 CHỈ KHI tiền hàng >= 100.000. Nếu không đủ thì không giảm.
4. Ship: giao tận nơi +15.000, tại quán +0.
5. Tổng = tiền hàng - giảm giá + ship.
Kiểm tra lại từng bước. Ghi kết quả ở dòng cuối: [RESULT: mã đơn - số tiền]
Sau đó viết 1 tin nhắn xác nhận dưới 30 từ, nêu mã đơn và số tiền,
không hứa thêm điều gì. Nếu mã chưa đủ điều kiện, báo nhẹ nhàng là chưa áp dụng được.
Đơn: """[dán các đơn HOP_LE]"""
```

- *Prompt xử lý nếu bị đứt gãy:* Hãy viết tiếp ngay sau cụm từ: "[3 - 5 từ cuối bị ngắt]". Không lặp lại phần đã viết.

# Bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
|---|---|---|
| Độ dễ hiểu | **Cao** | Trung bình |
| Tốn token | **Ít** (1 lần gọi) | Nhiều hơn (2 lần gọi) |
| Tốc độ | **Nhanh** | Chậm hơn |
| Chặn lỗi biên (size lạ, số ly = 0) | Trung bình: AI tự đoán | **Cực tốt**: có điều kiện rõ |
| Tính đúng quy tắc mã giảm giá | Dễ nhầm điều kiện 100.000đ | **Đúng**: ghi rõ "CHỈ KHI" |
| Gỡ lỗi | Khó | **Dễ**: sai bước nào sửa bước đó |
| Xử lý đứt gãy | Chạy lại từ đầu | **Viết tiếp** đúng chỗ ngắt |

---

# Chọn phương án

### Lựa chọn: Prompt 2

**Lý do:**

1. Sử dụng tiền thật nên sai là thiệt hại ngay (tính tiền cho size không tồn tại, giảm giá nhầm).
2. Dữ liệu có bẫy, chỉ điều kiện viết rõ mới chặn ổn định.
3. Chỉ chia 2 bước nên chi phí tăng không đáng kể.
