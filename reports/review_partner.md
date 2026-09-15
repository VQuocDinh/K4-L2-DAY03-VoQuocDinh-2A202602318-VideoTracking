# Peer review — Day 3

Reviewer chỉ ghi finding; tác giả tự sửa bài của mình và điền closure.

| Trường            | Giá trị                                                       |
| ------------------- | --------------------------------------------------------------- |
| Author              | Võ Quốc Dinh — 2A202602318                                   |
| Reviewer            | Bùi Quang Thái — 2A202603020                                 |
| Pair ID             | Võ Quốc Dinh (2A202602318) ↔ Bùi Quang Thái (2A202603020)  |
| CVAT version        | app.cvat.ai (bản web, tác giả dùng khi gán nhãn)          |
| Thời điểm review | 2026-09-15,**sau** pre-gold lock (17:08 Asia/Ho_Chi_Minh) |

**Phương pháp review.**  hỗ trợ soi frame và soạn finding; reviewer đọc và xác nhận từng finding. Review thực hiện trên file
MOT đã export (`annotations/clip_01/gt.txt`, trùng SHA-256 với snapshot pre-gold):

1. chạy `tools/check_mot_labels.py`;
2. vẽ bbox của tác giả lên ảnh gốc `data/clips/clip_01/img1/` và soi bằng mắt từng
   frame đầu/cuối track, đoạn occlusion sau xe buýt, đoạn giữa hai keyframe;
3. dùng chỗ bất đồng giữa BoT-SORT + ReID và nhãn tác giả (`outputs/eval_reid_vs_me.json`)
   để chọn frame cần soi.

Mọi finding dưới đây đều đã được xác nhận bằng ảnh gốc, không lấy kết luận từ
model hay gold. Vì review diễn ra sau lock, finding **không** được dùng để sửa
snapshot pre-gold.

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

CVAT đánh frame từ 0, MOT từ 1: CVAT frame = MOT frame − 1.

| # | CVAT frame | MOT frame | ID | Loại lỗi                          | Quan sát + rule áp dụng                                                                                                                                                                                    | Cách sửa đề xuất                                                                               | Closure: fixed / not-a-defect / needs-review                                  |
| -: | ---------: | --------: | -: | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| 1 |     83–99 |   84–100 |  6 | Bắt đầu track quá sớm          | Bbox nằm trên đuôi xe buýt ID 4; xe con phía sau chưa nhìn thấy được, chỉ lộ nóc xe từ khoảng frame 101–105. Rule: "bắt đầu từ frame đầu tiên xác định được là xe bốn bánh" | Dời keyframe đầu của ID 6 tới frame xe lộ ra (~101); đặt`outside` cho đoạn trước đó | needs-review — tác giả ghi nhận, chưa sửa trong CVAT                    |
| 2 |   138–141 |  139–142 |  5 | Bbox treo sau khi rời khung        | Bbox đứng im ở (0, 307, 64×47) trên mặt đường trống; xe đã ra khỏi rìa trái khoảng frame 138. Rule: bấm`outside` ở frame xe rời khung                                                    | Bấm`outside` ở frame đầu tiên không còn thấy xe                                           | needs-review — tác giả ghi nhận, chưa sửa                               |
| 3 |   148–150 |  149–151 |  4 | Bbox treo sau khi rời khung        | Xe buýt gần như đã ra khỏi rìa trái; frame 151 bbox 159×167 phủ mặt đường trống (interpolation vẫn chạy)                                                                                     | Bấm`outside` khoảng frame 149                                                                   | needs-review — tác giả ghi nhận, chưa sửa                               |
| 4 |         11 |        12 |  1 | Bbox treo 1 frame                   | Frame 12 bbox 35×58 ở rìa trái nhưng xe đã ra khỏi khung                                                                                                                                              | Bấm`outside` từ frame 12                                                                        | needs-review — tác giả ghi nhận, chưa sửa                               |
| 5 |   132–134 |  133–135 |  8 | Bắt đầu khi xe quá nhỏ         | Frame 133 bbox chỉ cao 7 px ở mép dưới ảnh, chưa xác định được là xe; frame 135 mới thấy nóc xe đỏ                                                                                         | Bắt đầu track khi phần nhìn thấy đủ nhận ra là xe (cao ≥ ~15 px)                         | needs-review — là ca mơ hồ, đã thêm ngưỡng vào`GUIDELINE_MINI.md` |
| 6 |     80–85 |    81–86 |  5 | Bbox không ôm phần nhìn thấy   | Xe con bị xe buýt che một phần; bbox 57×31 giữ nguyên kích thước và lấn sang đầu/đèn xe buýt thay vì co lại theo phần nhìn thấy                                                         | Thêm keyframe mỗi 2–3 frame trong đoạn bị che, co bbox theo phần nhìn thấy                 | needs-review — tác giả ghi nhận, chưa sửa                               |
| 7 |      0–14 |     1–15 |  3 | Validator cảnh báo bbox đứng im | Xe trắng đỗ bên đường, thật sự đứng yên; không phải quên`outside`                                                                                                                            | Không cần sửa                                                                                    | not-a-defect                                                                  |

## Reviewer checklist

| Hạng mục                                                           | PASS / FINDING / N/A | Frame–ID–evidence                                                                                                                       |
| -------------------------------------------------------------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh            | PASS                 | 8 track; không thấy người, xe máy, xe đạp hay quầy hàng (quầy hàng mà model nhận nhầm ở frame 16–116 không được gán) |
| Một xe giữ một ID; không reuse ID cho xe khác                   | PASS                 | Mỗi ID 1–8 là một đoạn frame liên tục, không đứt quãng                                                                        |
| Occlusion ngắn giữ ID; crossing không đổi ID                    | PASS                 | ID 5 giữ nguyên qua đoạn bị xe buýt che (frame 81–115)                                                                             |
| Entry/exit đúng; không box treo sau khi xe rời khung             | FINDING              | #1 ID 6 f84–100; #2 ID 5 f139–142; #3 ID 4 f149–151; #4 ID 1 f12; #5 ID 8 f133–135                                                    |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING              | #6 ID 5 f81–86 lấn sang xe buýt                                                                                                        |
| Frame giữa hai keyframe không bị interpolation drift              | FINDING              | #3 ID 4 f151 và #6 ID 5 f81–86 đều do interpolation chạy tiếp khi xe đổi trạng thái                                             |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID   | PASS                 | Validator 0 lỗi trên cả hai clip; frame 1..190; 8 ID khác nhau                                                                        |
| Mọi finding có cách sửa và closure do tác giả điền          | PASS                 | 7/7 finding có cách sửa và closure                                                                                                    |

## Self-QC attestation của reviewer

Self-QC attestation dưới đây do reviewer tự điền về bản annotation của chính mình.

| Lượt                      | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --------------------------- | ------------------------------- | ------------------- |
| 1 — identity/timeline      |                                 |                     |
| 2 — endpoint/scope         |                                 |                     |
| 3 — geometry/interpolation |                                 |                     |

## Exit ticket

1. **Finding quan trọng nhất và rule dùng để kết luận:** #1, ID 6 bắt đầu ở frame 84
   trong khi xe chỉ nhìn thấy được từ khoảng frame 101. Rule: bắt đầu track từ frame
   đầu tiên xác định được là xe bốn bánh. Riêng lỗi này tạo 17 bbox thừa.
2. **Một finding tác giả đóng là `not-a-defect`, kèm lý do:** #7, ID 3 đứng im frame
   1–15 là xe đỗ thật, không phải quên bấm `outside`.
3. **Một rule cần Lab Coach làm rõ:** khi xe còn rất nhỏ ở mép ảnh (#5) hoặc bị che gần
   hết sau xe khác (#1), thì từ bao nhiêu pixel / bao nhiêu phần trăm nhìn thấy mới
   bắt đầu track?

**Warm-up `clip_02` (ghi chú thêm).** ID 4 chỉ có 2 frame (1–2) và ID 5 còn bbox ở
frame 57–60 sau khi xe rời khung; cả hai đều là lỗi biên track giống #2–#4 của `clip_01`.
