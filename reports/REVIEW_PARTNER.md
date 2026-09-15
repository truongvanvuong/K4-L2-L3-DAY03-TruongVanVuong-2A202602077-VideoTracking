# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Trương Văn Vượng - 2A202602077` |
| Reviewer | `Self-review (Tự rà soát độc lập trên evidence/pre-gold/clip_01/gt.txt)` |
| Pair ID | `Solo (Không ghép cặp)` |
| CVAT version | `CVAT 2.x` |
| Thời điểm review | `15/09/2026 17:30` |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | ---: | --- | --- | --- |
| 1 | 59–78 | 60–79 | 5 | Bbox treo sớm | Xe còn ở quá xa (< 10px), chưa rõ hình dáng xe 4 bánh | Dời frame bắt đầu track về frame 79 khi kích thước xe rõ ràng | fixed |
| 2 | 79–100 | 80–101 | 6 | Bbox treo sớm | Bbox xuất hiện trước khi xe hoàn toàn tiến vào làn đường | Chỉnh lại frame bắt đầu khớp frame 101 | fixed |
| 3 | 82–88 | 83–89 | 5 | Bbox trôi (drift) | Nội suy tuyến tính giữa 2 keyframe xa khiến bbox lệch khỏi xe | Đặt thêm keyframe bổ sung tại frame 83, 85, 88 | fixed |
| 4 | 148–150 | 149–151 | 4 | Bbox treo muộn | Xe đã lọt khỏi góc camera nhưng bbox còn treo lại 3 frame | Bấm Outside dứt điểm tại frame 149 | fixed |
| 5 | 168–170 | 169–171 | 8 | Bbox treo muộn | Xe đã chạy qua mép hình nhưng chưa ngắt track kịp thời | Bấm Outside ngay tại frame 169 | fixed |
| 6 | 107–108 | 108–109 | 6 | Bbox trôi (drift) | Xe tăng tốc rời camera, khoảng cách keyframe hơi thưa | Bổ sung keyframe tại frame 108 để nắn lại bbox ôm khít xe | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 tracks (ID 1 đến 8), 100% là ô tô 4 bánh |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0, mỗi xe duy trì 1 ID duy nhất |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 4 và 8 khi đi ngang qua nhau vẫn giữ nguyên ID |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | Tìm thấy 4 ca bbox treo tại thời điểm entry/exit (ID 4, 5, 6, 8) |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Bbox chạm mép ảnh đúng quy định, không vẽ ra ngoài |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Tìm thấy 2 ca bbox trôi nhẹ giữa các keyframe (ID 5, 6) |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Định dạng chuẩn MOT 1.1, 631 dòng, 190 frame |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Cả 6 finding đều có phương án xử lý rõ ràng |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | 8/8 tracks ổn định ID, không bị đứt đoạn vô cớ |
| 2 — endpoint/scope | ĐÃ SỬA | Đã xác định rõ các frame đầu/cuối của ID 4, 5, 6, 8 |
| 3 — geometry/interpolation | ĐÃ SỬA | Đã định vị các frame cần chèn thêm keyframe (ID 5, 6) |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `Cần xác định ngưỡng tối thiểu cho xe từ xa và bấm Outside ngay khi xe vừa khuất để tránh lỗi Bbox treo (FP).`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Không có (tất cả 6 finding đều là các điểm cần cải thiện độ chính xác).`
3. Một rule cần Lab Coach làm rõ (nếu có): `Quy định thống nhất kích thước pixel tối thiểu để bắt đầu gán nhãn cho xe mới xuất hiện từ phía xa.`
