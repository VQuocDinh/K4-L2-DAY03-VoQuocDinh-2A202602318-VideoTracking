# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Võ Quốc Dinh — 2A202602318 (cá nhân)
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục                                 | Giá trị                             |
| ------------------------------------ | ------------------------------------- |
| Công cụ                            | CVAT, export MOT 1.1                  |
| Thời gian gán`clip_02` (warm-up) | ~5 phút                              |
| Thời gian gán`clip_01`           | ~5 phút                              |
| Số track đã vẽ trong`clip_01`  | 8 (ID 1–8; 609 bbox trên 190 frame) |
| Số keyframe trung bình mỗi track  | khoảng 10–20                        |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe đi phía sau xe buýt (ID 5, 6; frame ~80–115).** Xe con bị thân xe buýt
   (ID 4) che gần hết. Tôi bắt đầu track ngay khi thấy xe phía sau xuất hiện và
   giữ nguyên ID trong lúc bị che. Sau khi so với gold, ID 6 hóa ra được bắt đầu
   quá sớm (xem mục 3).
2. **Xe trắng đỗ bên đường (ID 3, frame 1–190).** Xe gần như đứng im suốt clip;
   validator cảnh báo "bbox đứng im frame 1–15". Tôi giữ một track duy nhất suốt
   190 frame, chỉ chỉnh bbox khi xe dịch nhẹ — đây là vật thể thật, không phải
   quên bấm `outside`.
3. **Xe rời khung hình (ID 1 frame 12, ID 4/5/6/8 ở cuối track).** Tôi bấm
   `outside` khi xe ra khỏi khung, nhưng interpolation vẫn kéo bbox treo thêm 1–4
   frame. Gold kết thúc sớm hơn 3–5 frame, nên đây là nguồn FP chính của tôi.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: không thấy ID nhảy số hay hai xe đổi ID cho nhau (khớp với IDSW = 0 khi chấm với gold).
- Lượt 2: kiểm frame đầu/cuối của 8 track, không sửa gì trước khi khóa. Sau khi chấm, gold cho thấy lượt này còn bỏ sót: ID 6 bắt đầu sớm (84 so với 101) và ID 4/5/6/8 kết thúc trễ 3–5 frame.
- Lượt 3: không thấy bbox trôi rõ rệt. Gold chỉ ra đoạn trôi nhẹ ở ID 5 frame 81–86 và ID 4 frame 54 (IoU ~0.54), đều ở lúc xe mới vào khung/bị che.
- Validator `check_mot_labels.py`: 0 lỗi trên cả hai clip.

Kiểm chéo với: **Bùi Quang Thái — 2A202603020**; review diễn ra sau pre-gold lock, trên file MOT export và ảnh gốc. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 6. Số lỗi bạn ấy tìm được trong bản của bạn: 6 lỗi + 1 not-a-defect (7 finding).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Hai bên khác nhau ở **thời điểm bắt đầu và kết thúc track**. Tôi bắt đầu ID 6 ngay khi thấy có xe sau xe buýt (frame 84) và cho rằng đã bấm `outside` khi xe ra khỏi khung; reviewer chỉ ra bbox ở frame 84–100 phủ lên đuôi xe buýt, và ID 1/4/5 còn bbox treo 1–4 frame sau khi xe biến mất. Luật còn thiếu trong `GUIDELINE_MINI.md`: ngưỡng cụ thể cho "xe xác định được" (đã thêm: cao ≥ 15 px, nhận ra được là xe) và cách kiểm tra điểm `outside` (tua từng frame ở cuối track). Ca ID 3 đứng im được thống nhất là not-a-defect.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                               | Giá trị                                                            |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| SHA-256 từ`evidence/pre-gold/clip_01/manifest.json` | `c6f51bda0c16e528f809ab3352ab55cf0ba226673f6fcbbd2c85387214500e05` |
| Thời điểm khóa                                     | 2026-09-15T10:08:32Z (17:08 Asia/Ho_Chi_Minh)                        |
| Số row / frame / track trước khi mở reference      | 609 / 190 / 8                                                        |

|               |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP | FN | IDSW |
| ------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | -: | ---: |
| Bản pre-gold | 0.831 | 0.809 | 0.857 | 0.892 | 0.966 | 0.930 | 0.879 | 38 |  2 |    0 |
| Sau rework    | 0.831 | 0.809 | 0.857 | 0.892 | 0.966 | 0.930 | 0.879 | 38 |  2 |    0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** — ngay từ bản
pre-gold. Không rework nên hàng "Sau rework" giống hệt bản pre-gold
(`annotations/clip_01/gt.txt` trùng hash với snapshot).

**Đọc FP 38 / FN 2:** tôi gần như không bỏ sót xe nào (FN 2), nhưng có 38 bbox
thừa. Toàn bộ FP nằm ở biên track — bắt đầu sớm hoặc kết thúc trễ — chứ không
phải gán nhầm vật thể. Thói quen của tôi là gán dư ở hai đầu track.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID.
Tôi chưa sửa trong CVAT; bảng dưới là các lỗi đã xác nhận từ `outputs/eval_pre_gold.json`:

| Loại lỗi                 | Frame                                     | ID            | Đã sửa thế nào                                                                                                                                                               |
| -------------------------- | ----------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Bắt đầu track quá sớm | 84–100 (17 frame)                        | 6             | Chưa sửa. Xem lại frame 84–100: bbox nằm trên đuôi xe buýt, xe con chỉ thấy được từ khoảng frame 101–105. Cách sửa: dời keyframe đầu tới frame xe hiện ra |
| Kết thúc track trễ      | 157–161 / 139–142 / 149–151 / 169–171 | 6 / 5 / 4 / 8 | Chưa sửa. Bbox treo sau khi xe rời khung; cách sửa: bấm`outside` ở frame đầu tiên không còn thấy xe                                                                |
| Bắt đầu sớm            | 133–135                                  | 8             | Chưa sửa. Frame 133 bbox chỉ cao 7 px, chưa nhận ra là xe                                                                                                                   |
| Bbox trôi                 | 81–86 / 54                               | 5 / 4         | Chưa sửa. Cách sửa: thêm keyframe lúc xe mới vào khung/bị che                                                                                                            |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                               | Giá trị                                                                                            |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 (Google Colab)                                             |
| weights / hai tracker              | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment) |
| conf / IoU / imgsz / classes       | 0.25 / 0.70 / 960 / [2, 5, 7] (car, bus, truck)                                                      |
| device                             | `0` (CUDA GPU), `persist=True`, 190 frame                                                        |

| So sánh                  |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP | FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | -: | ---: |
| bạn vs gold              | 0.831 | 0.809 | 0.857 | 0.892 | 0.966 | 0.930 | 0.879 | 38 |  2 |    0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 |    2 |
| BoT-SORT + ReID vs gold   | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 |    2 |
| ReID vs bạn              | 0.769 | 0.713 | 0.830 | 0.899 | 0.884 | 0.765 | 0.891 | 85 | 56 |    2 |

## 5. Phân tích — năm câu hỏi

> Quy ước ID: "gold N" là track trong teaching reference. Track gold 4–8 trùng
> quãng frame với ID 4–8 của tôi; gold 1 (xe đỗ, frame 1–190) ứng với ID 3 của
> tôi, gold 2 ↔ ID 1, gold 3 ↔ ID 2.

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi (0.930) **thấp hơn** IDF1 (0.966), và IDSW = 0. Khoảng cách này
không đến từ lỗi ID mà từ 38 FP ở biên track: MOTA = 1 − (FP + FN + IDSW) / số
bbox gold = 1 − 40/573, nên mỗi bbox thừa bị trừ trọn một đơn vị. IDF1 =
2·IDTP / (bbox của tôi + bbox gold) = 2·571 / (609 + 573): 571/573 bbox gold vẫn
khớp đúng ID, còn bbox thừa chỉ làm tăng mẫu số nên bị phạt nhẹ hơn.

Trường hợp ngược lại (MOTA cao, IDF1 thấp) là dấu hiệu lỗi ID: MOTA chỉ tính
**một** IDSW tại frame đổi ID, còn IDF1 tính cả quãng đời track sau đó bị gán
sai. Nếu tôi tách một xe 60 frame thành hai ID ở giữa, MOTA chỉ mất 1/573, nhưng
IDF1 mất gần 30 bbox đúng identity, vì chỉ một nửa track được ghép với ID chính.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

Treatment tốt hơn ở identity: IDF1 0.875 → 0.900, AssA 0.776 → 0.820. IDSW
không đổi (2 → 2), và cả hai đều tách 3 xe thành nhiều ID.

- **Treatment tốt hơn — gold 4 (xe buýt), frame 54–60:** ByteTrack cấp ID 14 ở
  frame 56–57, mất detection ở frame 58, rồi mở ID 15 mới từ frame 59. BoT-SORT +
  ReID giữ **một ID 9 liên tục từ frame 55 đến 148**.
- **Cả hai đều sai — gold 5 (xe con sau xe buýt), frame 84–94:** xe bị xe buýt
  che. ByteTrack có ID 23 ở frame 85, mất detection ở frame 86–93, rồi mở ID 32 ở
  frame 94. ReID có ID 17 ở frame 85, mất ở frame 86, rồi mở ID 18 từ frame 87.
  ReID nhận lại xe nhanh hơn nhưng vẫn không nối được ID cũ. Giả thuyết: crop của
  một xe gần như bị che hết cho embedding kém tin cậy, khó vượt `appearance_thresh: 0.80`.
- Gold 6 (frame 113, ID 24 → 31) chỉ bị tách ở ReID; gold 4 chỉ bị tách ở ByteTrack.

Đây là **system comparison**, không cô lập causal effect của ReID: BoT-SORT và
ByteTrack khác implementation (Kalman state, cách ghép, cách xử lý track mất dấu),
nên phần cải thiện có thể đến từ những khác biệt đó chứ không chỉ từ appearance
embedding.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA 0.649 → 0.711, FN 54 → 26, FP gần như không đổi (88 → 91). Hai run dùng
**cùng detector input** (cùng weights, conf, IoU, imgsz, classes), nên FN giảm
không phải vì YOLO thấy thêm xe. Nguyên nhân là tracker quyết định bbox nào được
xuất ra: ví dụ gold 8 ở frame 140–151 bị ByteTrack bỏ trống 12 frame, còn ReID xuất
liên tục ID 34 từ frame 136 đến 168.

Lỗi còn lại chủ yếu thuộc **detector**:

- FP ~90 gần như giữ nguyên ở cả hai run và đến từ vật thể tĩnh bị nhận nhầm là xe
  (xem câu 4).
- FN còn lại rơi vào lúc xe bị che hoặc mới vào khung (gold 5 frame 79–84, gold 6
  frame 101–112), khi YOLO không có bbox để tracker ghép.
- Association chỉ gây 2 IDSW và 3 lần tách track, đều xảy ra ngay sau khi detector
  mất bbox.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

ReID **ID 7, frame 16–116 (43 frame)**, bbox ở khoảng (492, 215, 101×57): đó là
**quầy hàng / ki-ốt bên đường**, không phải xe. Bbox gần như đứng im suốt đoạn đó.
ByteTrack cũng nhầm cùng chỗ (ID 10, frame 17–116). Tôi không gán vì schema chỉ
có xe bốn bánh, và gold cũng không có bbox này. Đây là FP của detector: tracker nối
FP qua nhiều frame thành một "track" trông rất ổn định, và ReID không sửa được lỗi
này vì appearance của một vật đứng yên luôn giống chính nó.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **ReID làm tôi xem lại — ID 6 của tôi, frame 84–100.** Khi so ReID với nhãn của
  tôi, track 6 chỉ khớp 45/78 frame; ReID chỉ bắt xe này từ frame 104. Xem lại
  frame 84–100, bbox ID 6 của tôi nằm trên đuôi xe buýt khi xe con phía sau chưa
  nhìn thấy được; xe chỉ hiện ra khoảng frame 101–105. Gold bắt đầu track 6 ở
  frame 101. Model đúng một phần ở chỗ này: tôi đã bắt đầu track quá sớm.
- **Evidence cho thấy model sai — ReID ID 27 (frame 106–121) và ID 38 (frame
  158–178).** Đây là bbox ~20×20 sát rìa trái (y ≈ 280), conf 0.29–0.38. Phóng to
  frame 110 và 165 thấy đó là **rào chắn/biển kẻ ô đỏ-trắng** đứng yên, không phải
  xe. Tôi giữ nguyên annotation và không gán hai bbox này.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

**Sửa trong `GUIDELINE_MINI.md`:**

- Thêm ngưỡng bắt đầu track: phần nhìn thấy cao ≥ 15 px và nhận ra được là xe (thấy nóc, kính hoặc bánh). Không bắt đầu khi xe còn nằm sau xe khác, như ID 6 frame 84–100.
- Viết lại luật `outside`: bấm ở **frame đầu tiên không còn thấy xe**, không phải "khi xe ra khỏi khung", vì interpolation vẫn kéo bbox treo thêm 1–4 frame (ID 1, 4, 5).
- Thêm luật keyframe khi bị che hoặc chồng xe: mỗi 2–3 frame, bbox co theo phần nhìn thấy (ID 5 frame 81–86).
- Ghi rõ vật thể không gán mà dễ nhầm: quầy hàng, rào chắn kẻ ô, và xe đỗ vẫn phải gán.

**Đổi trong quy trình:**

- Không gán vội trong 5 phút một clip: dành thêm thời gian cho **lượt tự kiểm 2 (frame đầu/cuối)**, vì 38/40 lỗi của tôi nằm ở biên track, không phải ở ID.
- Với mỗi track, tua **từng frame** quanh điểm bắt đầu và điểm `outside` thay vì nhảy keyframe.
- Chạy `visualize_tracks.py` sau khi export để soi frame cuối từng track trước khi khóa pre-gold. Validator chỉ bắt lỗi định dạng, không bắt được bbox treo.
- Làm warm-up nghiêm túc hơn: `clip_02` đã báo đúng loại lỗi này (ID 5 treo frame 57–60) nhưng tôi chưa rút kinh nghiệm trước khi gán `clip_01`.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
