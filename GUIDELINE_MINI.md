# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Trương Văn Vượng-2A202602077`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                           | Không gán                                           |
| ----------------------------- | --------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                         |
| van, minivan                  | xe đạp                                              |
| xe buýt, minibus              | **xe máy / mô tô**                                  |
| xe tải, xe đầu kéo            | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có):

## 2. Luật ID — phần quan trọng nhất

| Tình huống                       | Luật của nhóm                                                                                                                                               | Vì sao                                                                                                                   |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Xe bị che một phần rồi hiện lại  | Giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps); frame nào bị che thì bật `Occluded` và chỉ vẽ phần thân xe nhìn thấy được.                  | Thời gian che ngắn, quán tính chuyển động không đổi nên người và tracker vẫn nhận diện đúng cùng một xe.                 |
| Xe bị che lâu hơn ngưỡng trên    | Đánh dấu `Outside` ngắt track khi xe khuất hẳn; nếu sau đó xuất hiện lại mà không chắc chắn 100% cùng xe thì gán ID mới.                                    | Tránh rủi ro gán nhầm sang phương tiện khác (ID switch) khi thời gian che khuất quá dài làm đứt mạch quỹ đạo.            |
| Xe rời khung hình rồi quay lại   | Mặc định: **gán track mới với ID mới**                                                                                                                      | Tuân thủ chuẩn MOT benchmark — một khi đối tượng đã ra khỏi trường nhìn (field of view) thì không thể truy vết liên tục. |
| Hai xe cắt nhau / chồng lên nhau | Xe ở gần camera vẽ bình thường; xe bị che ở sau bật `Occluded` ôm phần nhìn thấy; **tuyệt đối giữ nguyên ID cho cả 2 xe**, không tráo đổi ID khi tách nhau. | Bảo toàn tính nhất quán danh tính (tránh triệt để lỗi ID switch khi các bbox có độ trùng lặp cao).                       |

## 3. Luật bbox

| Tình huống                             | Luật của nhóm                                                                                                                                                                                                     |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                  | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                                                                                                                                     |
| Xe bị xe khác che một phần             | bbox ôm phần **nhìn thấy được**                                                                                                                                                                                   |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **chiều cao bbox $\ge 15\text{px}$** (hoặc nhận diện rõ đèn/bánh xe), không gán xe dạng đốm mờ $< 10\text{px}$ ở đường chân trời. |
| Xe đang đỗ, không di chuyển            | chỉ cần **2 keyframe** ở frame đầu và frame cuối (như Track 3); giữ cố định kích thước và vị trí bbox xuyên suốt clip.                                                                                            |
| Keyframe đặt dày ở đâu                 | đặt dày (**cách 3–5 frame**) tại các đoạn xe cua góc, đổi hướng, tăng tốc lướt gần camera hoặc khi bị che khuất; các đoạn đi thẳng đều đặt thưa hơn (10–15 frame).                                                |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01 / frame 1–190 / ID 3`
- Tình huống: Xe ô tô con đỗ cố định bên lề đường suốt cả clip, hoàn toàn không di chuyển.
- Quyết định: Tạo 1 track duy nhất xuyên suốt từ frame 1 đến 190, chỉ đặt đúng 2 keyframe (frame 1 và frame 190).
- Lý do: Xe tĩnh nên tọa độ và kích thước không đổi; đặt ít keyframe giúp bbox hoàn toàn bất động, tránh bị rung lắc hay trôi (drift) giữa các frame.

### Ca 2

- Clip / frame / ID: `clip_01 / frame 52–151 / ID 4`
- Tình huống: Xe tải lớn đi từ xa đến rất gần camera rồi rẽ thoát ra mép ảnh, kích thước bbox phóng to gấp 5-6 lần và chuyển động phi tuyến.
- Quyết định: Đặt keyframe dày dần (ở xa cách 10 frame, lại gần camera đặt cách 2–3 frame); bấm Outside dứt khoát tại frame 149 khi mép xe vừa chạm rìa ảnh.
- Lý do: Tốc độ thay đổi kích thước và góc nhìn khi ở gần camera rất nhanh, nếu không đặt keyframe dày thì nội suy tuyến tính sẽ khiến bbox bị lệch khỏi thân xe; bấm Outside sớm tránh lỗi bbox treo.

### Ca 3

- Clip / frame / ID: `clip_01 / frame 60–140 / ID 5`
- Tình huống: Xe xuất hiện từ phía xa tít đường chân trời, ban đầu chỉ là một chấm nhỏ mờ (~8px), sau đó phóng nhanh qua giao lộ.
- Quyết định: Không mở track từ frame 60 mà bắt đầu gán từ frame 79 khi chiều cao xe đạt $\ge 15\text{px}$ và nhìn rõ hình khối; bổ sung 3 keyframe tại frame 83, 85, 88 khi xe tăng tốc.
- Lý do: Tránh gán nhầm đốm mờ/nhiễu ảnh thành xe (tránh FP); bổ sung keyframe đoạn tăng tốc giúp bbox bám khít xe, không bị tụt IoU.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Bổ sung ngưỡng kích thước tối thiểu bắt đầu track:** Bắt buộc chiều cao bbox $\ge 15\text{px}$ và nhận diện được ít nhất 2 đặc trưng (đèn/bánh xe). Bỏ quy định cảm tính "vừa thấy thì gán", giúp loại bỏ 19 frame FP ở Track 5.
- **Quy định dứt điểm lệnh Outside tại rìa ảnh:** Ngay khi toàn bộ thân xe rời khỏi khung hình hoặc diện tích còn lại $< 10\%$ bị che khuất thì bấm Outside ngay lập tức, không để bbox trôi tự do thêm 2–3 frame (sửa lỗi bbox treo ở Track 4 frame 149 và Track 8 frame 169).
- **Chuẩn hóa mật độ keyframe theo khoảng cách camera:** Đoạn xe ở xa/đi thẳng đặt keyframe cách 10–15 frame; đoạn xe lại gần camera/cua góc đặt keyframe dày 3–5 frame một lần để tránh hiện tượng trôi box (drift).
