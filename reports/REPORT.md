# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Trương Văn Vượng - 2A202602077`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục                               | Giá trị             |
| --------------------------------- | ------------------- |
| Công cụ                           | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `90` phút           |
| Thời gian gán `clip_01`           | `40` phút           |
| Số track đã vẽ trong `clip_01`    | `8`                 |
| Số keyframe trung bình mỗi track  | `20`                |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Track 4 — xe lớn đi gần camera, bbox thay đổi rất mạnh (frame 52–151).** Bbox mở rộng từ ~14px (frame 52) lên hơn 490px (frame 129), tức tăng ~35 lần. Chuyển động phi tuyến khiến nội suy CVAT lệch nhanh, cần đặt keyframe gần như mỗi 1–2 frame (~82 keyframe).
   Xử lý: tua chậm từng frame, đặt keyframe dày đặc ở giai đoạn xe tiến gần và kiểm tra ngược lại giữa mỗi cặp keyframe.
2. **Track 5 — xe xuất hiện từ xa rìa phải, ban đầu rất nhỏ (frame 60–140).** Ở frame 60 bbox chỉ rộng ~8px, rất khó xác định đó là xe bốn bánh hay vật thể khác. Phải quyết định frame nào bắt đầu vẽ track — quá sớm thì bbox chưa chắc chắn, quá muộn thì bị tính FN. Xử lý: tua đến frame xe rõ ràng là xe bốn bánh (khoảng frame 63–65 khi bbox > 20px) rồi quay lại frame 60 đặt track, tuân theo luật "frame đầu tiên xác định được đó là xe bốn bánh" trong GUIDELINE_MINI.
3. **Nhiều xe cùng lúc và che chắn lẫn nhau (frame 80–140).** Giai đoạn này có tới 6 track đồng thời (ID 3, 4, 5, 6, 7, 8), xe chồng chéo trên khung hình. Track 8 xuất hiện ở rìa dưới (y≈527, frame 134–171), có thể bị xe khác che một phần.
   Xử lý: gán **xong hẳn một xe rồi mới sang xe khác** để tránh nhầm ID; bật Occluded khi xe bị che bán phần và chỉ vẽ bbox phần nhìn thấy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Phát nhanh cả clip, chỉ nhìn số ID trên bbox — Track 3 (xe đỗ) giữ ID ổn định xuyên suốt 190 frame, không nhấp nháy. Track 1 (frame 1–11) và Track 2 (frame 1–45) biến mất đúng lúc xe rời khung, không có hiện tượng ID đổi số. Kết luận: **0 ID switch**, identity nhất quán.
- Lượt 2: Kiểm frame đầu và frame cuối từng track — Track 5 bắt đầu frame 60 khi xe rất nhỏ (~8px), sớm hơn gold 19 frame → bbox thừa. Track 4 kết thúc frame 151 nhưng gold kết thúc frame 148 → treo thêm 3 frame. Track 8 kết thúc frame 171, gold kết thúc frame 168 → treo thêm 3 frame. Phát hiện **5 chỗ bbox treo** (ID 4, 5, 6, 7, 8) cần chỉnh Outside.
- Lượt 3: Nhảy vào giữa hai keyframe xa nhất — Track 3 tại frame 95 (giữa clip): bbox vẫn khít vì xe đỗ yên, chỉ cần 2 keyframe. Track 4 tại frame 100: bbox khớp. Track 5 tại frame 83–89: IoU tụt xuống 0.52–0.58 so với gold → **bbox trôi**, cần thêm keyframe. Track 6 tại frame 108–109: IoU ~0.58 → cũng cần thêm keyframe.

Kiểm chéo với: `Tự rà soát độc lập (Self-review trên evidence/pre-gold/clip_01/gt.txt do làm độc lập)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `N/A (làm độc lập)`. Số lỗi tự rà soát được trong bản pre-gold: `6 lỗi` (4 lỗi bbox treo thừa frame, 2 lỗi bbox trôi interpolation).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Do làm độc lập nên không có ca bất đồng giữa hai người, nhưng qua quá trình rà soát trực tiếp file `evidence/pre-gold/clip_01/gt.txt` phát hiện `GUIDELINE_MINI.md` còn thiếu:

- (1) **Ngưỡng kích thước bbox tối thiểu** để bắt đầu track xe từ xa — Track 5 bắt đầu từ frame 60 khi xe còn quá nhỏ (< 10px), cần quy định ngưỡng nhận diện tối thiểu (ví dụ $h \ge 15\text{px}$) mới bắt đầu mở track;
- (2) **Luật Outside dứt điểm tại biên ảnh** — Track 4 (frame 149–151) và Track 8 (frame 169–171) bị treo box rỗng 2–3 frame sau khi xe đã lọt hẳn ra ngoài; cần quy định bấm Outside ngay frame xe vừa khuất;
- (3) **Mật độ keyframe khi chuyển động phi tuyến** — Track 5 (frame 83–89) và Track 6 (frame 108–109) bị trôi box (drift) do nội suy giữa hai keyframe xa nhau, cần quy định đặt keyframe dày hơn (cách 3–5 frame) ở các khúc cua/tăng tốc.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                             | Giá trị                                                            |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `643df69aa6f38a27415c7157c136b3402314dfcf9375d6ff9eb206b621ecba1d` |
| Thời điểm khóa                                       | `2026-09-15T16:48:38`                                              |
| Số row / frame / track trước khi mở reference        | `631 / 190 / 8`                                                    |

|              |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------ | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| Bản pre-gold | 0.819 | 0.794 | 0.851 | 0.901 | 0.942 | 0.878 | 0.892 |  64 |   6 |    0 |
| Sau rework   | 0.819 | 0.794 | 0.851 | 0.901 | 0.942 | 0.878 | 0.892 |  64 |   6 |    0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi                    | Frame   | ID  | Đã sửa thế nào                                                                                           |
| --------------------------- | ------- | --- | -------------------------------------------------------------------------------------------------------- |
| Bbox treo (thừa trước gold) | 80–100  | 6   | ID 6 có bbox 20 frame trước khi gold track 6 xuất hiện → cần dời Outside sớm hơn đúng frame xe vào khung |
| Bbox treo (thừa trước gold) | 60–78   | 5   | ID 5 vẽ từ frame 60 nhưng gold bắt đầu frame 79 (xe còn quá nhỏ) → cần dời frame đầu track muộn hơn      |
| Bbox trôi (IoU thấp)        | 83–89   | 5   | IoU tụt 0.52–0.58 giữa hai keyframe → thêm keyframe tại frame 83, 85, 88 cho bbox bám sát                |
| Bbox treo (thừa sau gold)   | 149–151 | 4   | Xe đã rời khung nhưng bbox còn treo 3 frame → bấm Outside tại frame 149                                  |
| Bbox treo (thừa trước gold) | 101–105 | 7   | ID 7 xuất hiện sớm 5 frame so với gold → dời frame đầu track                                             |
| Bbox trôi (IoU thấp)        | 108–109 | 6   | IoU ~0.58 → thêm keyframe quanh frame 108 để chỉnh bbox                                                  |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị                                                 |
| ---------------------------------- | ------------------------------------------------------- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cpu` / `0.5.13`         |
| weights / hai tracker              | `yolo26n.pt` / `bytetrack.yaml` vs `botsort-reid.yaml`  |
| conf / IoU / imgsz / classes       | `0.25` / `0.70` / `960` / `[2, 5, 7]` (car, bus, truck) |
| device                             | `cpu`                                                   |

| So sánh                   |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| bạn vs gold               | 0.819 | 0.794 | 0.851 | 0.901 | 0.942 | 0.878 | 0.892 |  64 |   6 |    0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 |  88 |  54 |    2 |
| BoT-SORT + ReID vs gold   | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 |  91 |  26 |    2 |
| ReID vs bạn               | 0.761 | 0.708 | 0.818 | 0.910 | 0.875 | 0.753 | 0.902 |  80 |  73 |    3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- **MOTA (0.878) thấp hơn IDF1 (0.942)**.
- Nếu một hệ thống có **MOTA cao mà IDF1 thấp**, điều đó phản ánh hệ thống phát hiện đối tượng rất tốt ở từng frame riêng lẻ (ít FP, ít FN), nhưng khả năng duy trì danh tính đối tượng qua thời gian (association) rất kém: các track liên tục bị đổi ID, nhảy ID (ID switches) hoặc phân mảnh track.
- **Vì sao MOTA không phạt nặng lỗi ID**: Trong công thức $MOTA = 1 - \frac{\sum (FN_t + FP_t + IDSW_t)}{\sum GT_t}$, lỗi hoán đổi ID ($IDSW$) chỉ bị tính phạt 1 điểm duy nhất đúng tại frame xảy ra chuyển đổi ($IDSW_t = 1$). Ở tất cả các frame tiếp theo, dù vật thể đang mang ID sai so với ban đầu, nó vẫn được tính là True Positive ($TP$) bình thường và không bị trừ thêm điểm nào. Ngược lại, $IDF1$ đo lường sự nhất quán danh tính trên toàn bộ quỹ đạo ($IDTP$); khi một track bị cắt đôi hoặc đổi ID, phần lớn các frame của track đó sẽ bị biến thành $IDFP$ và $IDFN$, kéo điểm $IDF1$ sụt giảm rất nặng.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- So với gold, BoT-SORT + ReID vượt trội hơn ByteTrack ở cả hai chỉ số liên kết danh tính:
  - **IDF1:** ReID đạt `0.900`, cao hơn ByteTrack (`0.875`, tăng +0.025).
  - **AssA:** ReID đạt `0.820`, cao hơn ByteTrack (`0.776`, tăng +0.044).
  - **IDSW:** Cả hai mô hình đều chỉ có `2` lần chuyển ID.
- **Dẫn chứng chuỗi frame (frame 80–120):** Khi xe số 5 và xe số 6 cùng chuyển động trong khung hình với tốc độ thay đổi và có thời điểm giao cắt / kích thước biến đổi nhanh, ByteTrack (chỉ dựa vào vị trí hình học qua Kalman filter và 2 tầng ngưỡng IoU) bị đứt kết nối khi khoảng cách giữa các frame giãn nở, dẫn đến số FN tăng vọt lên 54 (mất dấu nhiều đoạn). Ngược lại, BoT-SORT + ReID tận dụng thêm vector đặc trưng ngoại hình (visual appearance embedding) để ghép nối lại đúng đối tượng kể cả khi độ phủ IoU bị sụt giảm, giúp giảm FN xuống còn 26 và nâng cao AssA.
- **Lưu ý về tính nhân quả (causal effect):** Sự khác biệt này **không cô lập được hiệu ứng nhân quả thuần túy của riêng module ReID**, vì BoT-SORT và ByteTrack là hai thuật toán có cấu trúc cài đặt (implementation) khác nhau ở nhiều mặt: BoT-SORT sử dụng thêm Camera Motion Compensation (CMC) bằng thuật toán trích xuất điểm ảnh, cải tiến mô hình Kalman filter cho cả chiều rộng/chiều cao ($w, h$), và cơ chế kết hợp điểm tương đồng khác biệt so với logic 2 tầng của ByteTrack.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **Sự thay đổi giữa ByteTrack và BoT-SORT + ReID:**
  - **DetA:** Tăng từ `0.649` lên `0.711`.
  - **FN (bỏ sót):** Giảm mạnh hơn một nửa, từ `54` xuống `26` box.
  - **FP (báo nhầm):** Tăng nhẹ từ `88` lên `91` box.
- **Lỗi còn lại chủ yếu do DETECTOR hay ASSOCIATION?**
  - Lỗi còn lại chủ yếu là do **DETECTOR**:
    - Chỉ số $AssA = 0.820$ cao hơn đáng kể so với $DetA = 0.711$, và IDSW chỉ có 2 lỗi trên toàn bộ 190 frame, cho thấy khâu duy trì ID (Association) hoạt động rất tốt.
    - Điểm nghẽn hạn chế HOTA chủ yếu nằm ở khâu phát hiện (Detector) với **91 FP** (nhận nhầm các đốm sáng, cấu trúc lề đường hoặc bóng xe thành vật thể) và **26 FN** (bỏ sót các frame xe ở quá xa, kích thước quá nhỏ hoặc mép viền ngoài ảnh).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Frame 87, Track 5 (xe di chuyển ở làn giữa):**
  - **Đúng:** Trong bản gán nhãn của bạn, xe này được theo dõi liên tục từ frame 60 đến frame 140 với đúng 1 ID duy nhất (Track 5), không bị gián đoạn hay nhầm lẫn.
  - **ReID sai:** Tại frame 87, mô hình ReID gặp lỗi **ID Switch**: track đang được gán ID 17 đột ngột bị cắt đứt và chuyển sang ID 18 (bị phân mảnh thành `[17, 18]`). Model ReID bị sai do vector đặc trưng ngoại hình bị biến động giữa hai frame liên tiếp dẫn đến mất liên kết danh tính, trong khi người gán nhãn nắm rõ ngữ cảnh chuyển động liên tục của xe.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Frame 60–78, Track 5:**
  - Trong bản gán nhãn ban đầu của bạn, bạn đã mở track và vẽ bbox cho xe ID 5 ngay từ frame 60.
  - Tuy nhiên, model ReID chỉ bắt đầu phát hiện và theo dõi xe này từ frame 79–80 (hoàn toàn khớp với đáp án chuẩn `gold/gt.txt` bắt đầu từ frame 79).
  - Khi xem lại video tại các frame 60–78, chiếc xe này ở vị trí cực xa gần đường chân trời, kích thước chỉ có vài pixel mờ nhạt và chưa thể khẳng định chắc chắn là xe bốn bánh. Model ReID (và gold) đã đúng khi bỏ qua đoạn này. Điều này cho thấy người gán nhãn đã vẽ quá sớm (tạo ra 19 frame FP thừa), chỉ ra sự cần thiết phải chuẩn hóa một quy định rõ ràng về ngưỡng kích thước tối thiểu trong `GUIDELINE_MINI.md`.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa đổi trong `GUIDELINE_MINI.md`:**
  - Quy định rõ ràng **ngưỡng kích thước tối thiểu** ($h \ge 15\text{px}$ hoặc diện tích $\ge 150\text{px}^2$) để bắt đầu mở track, tuyệt đối không gán các chấm mờ ở đường chân trời để tránh tạo FP thừa.
  - Chuẩn hóa **luật Outside dứt điểm**: ngay khi xe vừa khuất hoàn toàn hoặc phần thân còn lại $< 10\%$ lọt ra mép hình thì phải bấm Outside ngay lập tức, tránh để bbox trôi tự do thêm 2–3 frame.
  - Hướng dẫn cụ thể **mật độ keyframe theo chuyển động**: xe tĩnh/đỗ chỉ cần 2 keyframe ở hai đầu; xe di chuyển thẳng cách 10–15 frame; xe cua góc/tăng tốc gần camera phải đặt keyframe dày cách 3–5 frame.

- **Thay đổi trong quy trình làm việc:**
  - **Chuyển hẳn sang chiến lược Track-by-track:** Gán trọn vẹn từng xe từ khi xuất hiện đến khi biến mất rồi mới chuyển sang xe tiếp theo, thay vì gán dàn trải theo từng frame (giúp loại bỏ 100% nguy cơ nhầm lẫn ID).
  - **Tự kiểm tra 3 lượt tua trước khi khóa nhãn:**
    - _Lượt 1 (Tốc độ 2x):_ Chỉ nhìn nhãn số ID để đảm bảo không bị nhấp nháy hoặc đổi ID giữa chừng.
    - _Lượt 2:_ Soi kỹ frame đầu và frame cuối của từng track để đảm bảo cắt đúng lúc xe vào/ra khung hình.
    - _Lượt 3:_ Nhảy vào các frame nằm giữa hai keyframe xa nhau để nắn lại bbox nếu bị trôi (drift).
  - **Chạy tự động công cụ `tools/check_mot_labels.py` ngay sau khi export:** Phát hiện tức thì các cảnh báo track quá ngắn, track đứt đoạn hay bbox treo trước khi nộp bài.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
