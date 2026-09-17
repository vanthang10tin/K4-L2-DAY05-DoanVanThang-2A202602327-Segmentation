# Báo cáo Ngày 5 — Segmentation

**Họ tên:** Đoàn Văn Thắng &nbsp;·&nbsp; **MSSV:** 26A202602327 - T037

---

## 1. Điểm số thực tế (Trích xuất từ `reports/SCORECARD.md`)

> **Thời gian cập nhật:** 17/09/2026 (Lần chấm 2 sau khi sửa nhãn)  
> **Tổng điểm hiện tại:** **47.2 / 100 điểm** *(Tăng +12.2 điểm so với lần 1)*  
> **Cờ gian lận (Anti-cheat flags):** Không có (`no anti-cheat flags`)

| Task | Nhóm (Group) | Loại phân vùng | Metric quan sát | Điểm số đạt được | Tiến độ so với lần 1 |
| :--- | :--- | :--- | ---: | ---: | :--- |
| **easy_semantic** | tiers | semantic (mIoU) | **0.810** (Coverage: 94.9%) | **18.2 / 20** | 🟢 Tăng từ 13.9 lên 18.2 |
| **medium_instance** | tiers | instance (matched-IoU × R) | **0.618** (Mean IoU: 0.797) | **15.5 / 32** | ⚪ Giữ nguyên |
| **hard_panoptic** | tiers | panoptic (PQ) | **0.403** (SQ: 0.619, RQ: 0.531) | **13.5 / 30** | 🟢 Tăng từ 5.6 lên 13.5 |
| cp1_holes | checkpoints | instance | *(chưa nộp)* | 0.0 / 3 | — |
| cp2_slice | checkpoints | instance | *(chưa nộp)* | 0.0 / 3 | — |
| cp5_occlusion | checkpoints | instance | *(chưa nộp)* | 0.0 / 3 | — |
| cp3_thin | checkpoints | semantic | *(chưa nộp)* | 0.0 / 3 | — |
| cp4_curb | checkpoints | semantic | *(chưa nộp)* | 0.0 / 3 | — |
| cp6_coverage | checkpoints | semantic | *(chưa nộp)* | 0.0 / 3 | — |
| **TỔNG CỘNG** | | | | **47.2 / 100** | **Tăng từ 35.0 lên 47.2** |

---

## 2. Một lớp / vật điểm thấp — và cách sửa (Minh chứng thực tế)

### Phân tích ca thành công điển hình: Lớp `sky` và `vegetation` (Bài Easy Semantic)
- **Lớp phân tích:** `sky` (Bầu trời) trên ảnh `817bca71-00000000.jpg`.
- **Metric trước khi sửa:** 
  - IoU của `sky` ở lần 1 chỉ đạt **0.062** (gần như mất trắng điểm).
  - Điểm task `easy_semantic` lần 1 chỉ đạt **13.9 / 20** (mIoU: 0.712).
- **Nguyên nhân (bằng chứng trên ảnh BDD100K):**
  - Khi tô bằng cọ Brush, lớp cây cối (`vegetation`) bị tô đè trùm lên gần như toàn bộ vùng trời phía trên. Diện tích bầu trời chỉ còn 15.818 pixel trong khi Ground Truth thực tế là 255.740 pixel.
- **Quy tắc áp dụng + cách sửa:**
  - Áp dụng quy tắc tại Mục 3 & 4 của `guideline-mini-sheet.md`: *"Vẽ từ xa đến gần — tô lớp nền to (sky) trước, sau đó mới tỉa ngọn cây (vegetation)"*.
  - Thực hiện trên CVAT: Dùng cọ tô lại toàn bộ mảng trời phía trên với nhãn `sky`, sau đó mới tỉa lại viền ngọn cây, tránh để cây lấn át bầu trời.
- **Minh chứng điểm trước $\rightarrow$ sau khi sửa:**
  - **Lớp `sky`:** IoU vọt từ **0.062 $\rightarrow$ 0.968**!
  - **Lớp `vegetation`:** IoU tăng từ **0.573 $\rightarrow$ 0.809**!
  - **Tổng điểm task `easy_semantic`:** Tăng từ **13.9 / 20 $\rightarrow$ 18.2 / 20** (gần đạt tối đa).

### Phân tích bổ sung: Lớp `truck` và `road` (Bài Hard Panoptic)
- **Trước khi sửa:** Ảnh `000000460147.jpg` bị bỏ quên không gán `truck` (nhầm thành car) và ảnh `000000350023.jpg` quên gán nhãn `road` $\rightarrow$ PQ của cả `truck` và `road` đều bị **0.000 điểm**.
- **Sau khi bổ sung nhãn đúng:** 
  - Lớp `truck` đạt PQ = **0.592**; lớp `road` đạt PQ = **0.415**.
  - Điểm bài Hard tăng vọt từ **5.6 $\rightarrow$ 13.5 / 30 điểm**.

---

## 3. Ba tình huống mơ hồ (Trích từ `guideline-mini-sheet.md`)

### A — `car` hay `truck`/`van`? (pickup có thùng, minibus…)
- **Ảnh và vị trí vật:** Ảnh `000000460147.jpg` (Hard), vị trí các phương tiện lớn di chuyển ở làn xe phía sau/xa.
- **Dấu hiệu nhìn thấy:** Phần đuôi có kết cấu thùng hàng khối hộp kín và sàn xe chuyên chở hàng hóa rõ rệt, kích thước tổng thể lớn hơn hẳn ô tô con thông thường.
- **Quy tắc áp dụng + quyết định:** Áp dụng quy tắc tại Mục 2 của guideline: *"truck: có thùng/ben/sàn hàng rõ; car: ô tô con, SUV, taxi"*. Quyết định: Gán nhãn `truck`.

### B — Instance: hai vật hay một? (xe sát nhau / xe bị che cắt đôi)
- **Ảnh và vị trí:** Ảnh `000000373353.jpg` (Medium), vị trí 2 chiếc ô tô đỗ song song sát cạnh nhau trên lề đường.
- **Slice hay Join? Vì sao:** Chọn **Slice (Alt+J)**. Vì đây là hai phương tiện đếm được (instance) riêng biệt, nhưng khi dùng công cụ vẽ tự động/SAM dễ bị dính chung đường biên thành một khối. Cần dùng Slice để cắt rời thành 2 mask độc lập.

### C — Semantic: `road` hay `sidewalk` ở chỗ bó vỉa?
- **Ảnh và vị trí:** Ảnh `81ae7cbb-6bc63a4a.jpg` (Easy), đoạn mép vỉa hè tiếp giáp lòng đường xe chạy phía bên phải.
- **Bằng chứng ở mức phóng 100%:** Dù bề mặt đường và vỉa hè đều có màu xám bê tông/asphalt tương tự nhau, mức phóng 100% cho thấy rõ đường gờ bó vỉa nhô cao (curb line) và sự ngắt quãng của vạch kẻ mép đường xe chạy.
- **Quyết định:** Áp dụng quy tắc *"Bó vỉa (curb)"* tại Mục 4: Lấy ranh giới là mép kết thúc mặt đường xe chạy. Phần lòng đường gán `road`, gờ bó vỉa hắt vào trong phần cho người đi bộ gán `sidewalk`.

---

## 4. Suy ngẫm & Tự đánh giá

- **Vì sao IoU cao chưa chắc mép đẹp?**
  - IoU (Intersection over Union) là phép đo tính trên tổng thể diện tích pixel. Đối với các vật thể có diện tích lớn (như lòng đường, tòa nhà, bầu trời), phần lõi chiếm đại đa số diện tích (>90–95%). Dù viền mép bị răng cưa, cọ vẽ lượn sóng hoặc lấn vài pixel vào vỉa hè, diện tích giao (Intersection) trên hợp (Union) vẫn có thể đạt mức rất cao (>0.85). Do đó, điểm IoU cao phản ánh độ phủ đúng vị trí nhưng chưa đảm bảo tính sắc nét và hoàn hảo về mặt hình học của đường biên.
- **Vì sao panoptic khó hơn instance?**
  - **Ràng buộc phủ kín không chồng lấn:** Instance segmentation chỉ quan tâm đến các vật thể đếm được (things) rời rạc và bỏ qua nền. Ngược lại, Panoptic segmentation đòi hỏi mọi pixel trong ảnh đều phải có đúng một nhãn (cả stuff như đường, vỉa hè, bầu trời lẫn things như xe, người) và tuyệt đối không được chồng lấn.
  - **Chỉ số PQ (Panoptic Quality) có luật trừ điểm kép:** PQ = SQ × RQ. Để được tính là nhận diện đúng (True Positive), mask phải có IoU > 0.5 với Ground Truth. Chỉ cần vẽ sót một người đi bộ hoặc vẽ nhầm 1 chiếc xe tải thành xe con, chỉ số RQ sẽ bị phạt nặng nề, làm sụt giảm điểm PQ ngay lập tức (minh chứng là lần 1 bài Hard chỉ đạt 5.6đ, sau khi sửa đúng nhãn đã tăng lên 13.5đ).
- **Giải trình cờ nghi vấn (Anti-cheat flags):**
  - Kết quả kiểm tra từ hệ thống: Cả 3 bài nộp đều hiển thị `no anti-cheat flags`. Điểm số đạt được hoàn toàn phản ánh quá trình gán nhãn thủ công và tinh chỉnh có tiến bộ thực tế của học viên.
