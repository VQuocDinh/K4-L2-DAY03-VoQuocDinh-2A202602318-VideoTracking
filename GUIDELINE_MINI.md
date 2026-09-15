# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Võ Quốc Dinh
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                            | Không gán                                                   |
| ------------------------------- | ------------------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                               |
| van, minivan                    | xe đạp                                                      |
| xe buýt, minibus               | **xe máy / mô tô**                                   |
| xe tải, xe đầu kéo          | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |


Bổ sung của nhóm: xe đỗ bên đường vẫn gán (ví dụ `clip_01` ID 3). Không gán quầy hàng/ki-ốt, rào chắn hay biển kẻ ô đỏ-trắng, kể cả khi model nhận chúng là xe.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (= 2 giây @ 12.5 fps, mặc định của lab) | cùng một chiếc xe; tách ID sẽ làm IDF1/AssA giảm trên cả quãng đời còn lại của track |
| Xe bị che lâu hơn ngưỡng trên | kết thúc track bằng `outside` ở frame bị che hoàn toàn; khi hiện lại thì mở **track mới** | quá 2 giây thì không còn chắc chắn đó là cùng một xe; một ID mới dễ kiểm hơn một ID đoán sai |
| Xe rời khung hình rồi quay lại | **track mới** (mặc định của lab) | ra khỏi khung là mất hết bằng chứng để nối identity |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe giữ ID và bbox riêng; đặt keyframe ở frame trước, trong và sau lúc chồng nhau rồi tua chậm kiểm tra hai ID không đổi cho nhau | ví dụ xe buýt ID 4 đi qua xe con ID 5 và ID 6 ở frame 81–115; interpolation qua đoạn chồng nhau là nơi dễ đổi ID nhất |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh. Bấm `outside` ở **frame đầu tiên không còn thấy xe**, không để interpolation kéo bbox tới lúc xe biến mất |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được**; thêm keyframe mỗi 2–3 frame để bbox co theo, không giữ nguyên kích thước cũ |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: phần nhìn thấy **cao ≥ 15 px** và nhận ra được là xe (thấy nóc, kính hoặc bánh), không bắt đầu khi chỉ thấy một vệt màu ở mép ảnh |
| Xe đang đỗ, không di chuyển | vẫn gán một track suốt thời gian xe trong khung; chỉ thêm keyframe khi xe dịch hoặc bị che. Cảnh báo "bbox đứng im" của validator cho xe đỗ là not-a-defect |
| Keyframe đặt dày ở đâu | dày (mỗi 2–3 frame) lúc xe vào/ra khung, bị che, đổi hướng hoặc chồng xe khác; thưa (10–20 frame) khi xe đi thẳng đều hoặc đứng yên |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01` / frame 84–105 / ID 6
- Tình huống: xe con đi phía sau xe buýt ID 4, bị thân xe buýt che gần hết; ban đầu chỉ lộ ra một phần rất nhỏ rồi mới thấy rõ nóc xe ở khoảng frame 101–105.
- Quyết định: khi gán, tôi bắt đầu track ngay lúc thấy "có xe phía sau" (frame 84). Sau review và so với reference, luật đúng là bắt đầu ở frame nóc xe nhìn thấy được (~101).
- Lý do: bbox ở frame 84–100 thực chất phủ lên đuôi xe buýt, không phải phần nhìn thấy của xe con; 17 bbox này thành FP.

### Ca 2

- Clip / frame / ID: `clip_01` / frame 1–190 / ID 3
- Tình huống: xe trắng đỗ bên đường gần như đứng im cả clip; validator cảnh báo "bbox đứng im frame 1–15 — quên bấm outside?".
- Quyết định: giữ **một track duy nhất** suốt 190 frame, chỉ chỉnh bbox khi xe dịch nhẹ.
- Lý do: xe đỗ vẫn là `vehicle` theo schema; cảnh báo của validator chỉ là gợi ý kiểm tra, không phải lỗi.

### Ca 3

- Clip / frame / ID: `clip_01` / frame 137–142 / ID 5 (tương tự ID 4 frame 149–151, ID 1 frame 12)
- Tình huống: xe đi ra khỏi rìa trái. Tôi định bấm `outside` khi xe ra hẳn khỏi khung, nhưng bbox vẫn còn treo trên mặt đường trống 1–4 frame vì interpolation chạy tiếp.
- Quyết định: kết thúc track ở frame đầu tiên không còn thấy xe, và tua từng frame ở cuối track để kiểm tra.
- Lý do: bbox treo không ứng với vật thể nào nên bị tính là FP; đây là nguồn FP chính của tôi (reference kết thúc sớm hơn 3–5 frame).

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **"Frame đầu tiên xác định được là xe" còn mơ hồ** khi xe bị che sau xe khác hoặc chỉ lộ một vệt ở mép ảnh (ID 6 frame 84, ID 8 frame 133 cao 7 px). Đã thêm ngưỡng: phần nhìn thấy cao ≥ 15 px và nhận ra được là xe.
- **Luật `outside` thiếu cách kiểm tra**: "bấm khi xe rời khung" chưa đủ vì interpolation vẫn kéo bbox sang frame xe đã biến mất (ID 1, 4, 5). Đã bổ sung: bấm ở frame đầu tiên không còn thấy xe, rồi tua từng frame quanh điểm kết thúc.
- **Bbox khi bị che thiếu quy định keyframe**: ID 5 frame 81–86 giữ nguyên kích thước và lấn sang xe buýt. Đã bổ sung: keyframe mỗi 2–3 frame trong đoạn bị che.