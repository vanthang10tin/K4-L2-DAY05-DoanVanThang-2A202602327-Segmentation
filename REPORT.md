# Báo cáo Ngày 5 — Segmentation

**Họ tên:** Đoàn Văn Thắng &nbsp;·&nbsp; **MSSV:** 26A202602327 - T037

---

## 1. Điểm số thực tế (Trích xuất từ `reports/SCORECARD.md`)

> **Thời gian cập nhật:** 17/09/2026 (Lần chấm 3 — sau khi tinh chỉnh toàn diện nhãn cả 3 cấp độ)  
> **Tổng điểm hiện tại:** **51.8 / 100 điểm** *(Tăng từ 35.0 ban đầu $\rightarrow$ 47.2 $\rightarrow$ **51.8 điểm**)*  
> **Cờ gian lận (Anti-cheat flags):** Tuyệt đối sạch sẽ (`no anti-cheat flags`)

| Task | Nhóm (Group) | Loại phân vùng | Metric quan sát | Điểm số đạt được | Tiến độ cải thiện |
| :--- | :--- | :--- | ---: | ---: | :--- |
| **easy_semantic** | tiers | semantic (mIoU) | **0.810** (Coverage: 94.9%) | **18.2 / 20** | 🟢 Tăng mạnh từ 13.9 lên 18.2 |
| **medium_instance** | tiers | instance (matched-IoU × R) | **0.683** (Mean IoU: 0.795, Recall: 85.9%) | **20.1 / 32** | 🟢 Tăng từ 15.5 lên 20.1 |
| **hard_panoptic** | tiers | panoptic (PQ) | **0.403** (SQ: 0.619, RQ: 0.531) | **13.5 / 30** | 🟢 Tăng vọt từ 5.6 lên 13.5 |
| cp1_holes | checkpoints | instance | *(chưa nộp)* | 0.0 / 3 | — |
| cp2_slice | checkpoints | instance | *(chưa nộp)* | 0.0 / 3 | — |
| cp5_occlusion | checkpoints | instance | *(chưa nộp)* | 0.0 / 3 | — |
| cp3_thin | checkpoints | semantic | *(chưa nộp)* | 0.0 / 3 | — |
| cp4_curb | checkpoints | semantic | *(chưa nộp)* | 0.0 / 3 | — |
| cp6_coverage | checkpoints | semantic | *(chưa nộp)* | 0.0 / 3 | — |
| **TỔNG CỘNG** | | | | **51.8 / 100** | **Tăng từ 35.0 $\rightarrow$ 47.2 $\rightarrow$ 51.8** |

---

## 2. Một lớp / vật điểm thấp — và cách sửa (Minh chứng thực tế)

### Ca 1 (Semantic): Lớp `sky` và `vegetation` trên bộ Easy Semantic
- **Lớp phân tích:** `sky` (Bầu trời) và `vegetation` (Cây cối) trên ảnh `817bca71-00000000.jpg`.
- **Metric trước khi sửa:**
  - IoU của `sky` ở lần 1 chỉ đạt **0.062** (gần như mất trắng điểm lớp này).
  - Điểm task `easy_semantic` lần 1 chỉ đạt **13.9 / 20** (mIoU: 0.712).
- **Nguyên nhân (bằng chứng trên ảnh BDD100K):**
  - Khi thao tác bằng công cụ cọ Brush, lớp cây cối (`vegetation`) bị tô đè trùm lên gần như toàn bộ vùng nền trời phía trên. Diện tích bầu trời trong nhãn nộp lúc đầu chỉ còn 15.818 pixel trong khi Ground Truth thực tế là 255.740 pixel.
- **Quy tắc áp dụng + cách sửa:**
  - Áp dụng nguyên tắc hình học tại Mục 3 & 4 của `guideline-mini-sheet.md`: *"Vẽ từ xa đến gần — tô lớp nền to (sky) trước, sau đó mới tỉa ngọn cây (vegetation)"*.
  - Thao tác trên CVAT: Dùng cọ tô phủ lại toàn bộ mảng trời phía trên với nhãn `sky`, sau đó mới tỉa lại viền tán lá cây, tránh để cây lấn át bầu trời.
- **Minh chứng điểm trước $\rightarrow$ sau khi sửa:**
  - **Lớp `sky`:** IoU vọt từ **0.062 $\rightarrow$ 0.968**!
  - **Lớp `vegetation`:** IoU tăng từ **0.573 $\rightarrow$ 0.809**!
  - **Tổng điểm task `easy_semantic`:** Tăng từ **13.9 / 20 $\rightarrow$ 18.2 / 20** (tiệm cận điểm trần 20).

### Ca 2 (Instance): Bổ sung vật thể nhỏ, tách cụm xe và xử lý che khuất (Medium Instance)
- **Vấn đề trước khi sửa:**
  - Lần chấm trước đạt **15.5 / 32** điểm: chỉ số Recall@0.5 chỉ đạt **0.77** (77%), bỏ sót tới **16 vật thể** (FN = 16), TP chỉ có 55 trên tổng số 71 vật thể Ground Truth. Nhiều xe đỗ sát nhau ở xa bị gộp thành một khối mask duy nhất, và một số người đi bộ bị che khuất một phần bởi cột/biển báo không được gán nhãn đầy đủ.
- **Quy tắc áp dụng + cách sửa:**
  - Áp dụng kỹ thuật tách vật: Sử dụng công cụ **Slice (`Alt+J`)** trên CVAT để cắt rời các cặp xe đỗ kề sát nhau thành từng instance riêng biệt với ID độc lập.
  - Phóng to 200–300% để bổ sung các instance người đi bộ (`person`) và xe hơi (`car`) ở hậu cảnh xa và các vật thể bị che khuất nhưng còn nhìn thấy rõ hình dạng.
  - Tuân thủ quy tắc *"không khoét kính xe"* (giữ nguyên polygon liền khối bao bọc thân xe).
- **Minh chứng điểm trước $\rightarrow$ sau khi sửa:**
  - **Số lượng vật thể đúng (True Positives):** Tăng từ **55 $\rightarrow$ 61 vật**.
  - **Số lượng bỏ sót (False Negatives):** Giảm mạnh từ **16 $\rightarrow$ 10 vật**.
  - **Chỉ số Recall@0.5:** Tăng từ **0.77 (77%) $\rightarrow$ 0.859 (85.9%)**!
  - **Độ trùng khớp biên (Mean Matched IoU):** Đạt mức cao **0.795** (~80%).
  - **Tổng điểm task `medium_instance`:** Tăng từ **15.5 / 32 $\rightarrow$ 20.1 / 32 điểm** (+4.6 điểm).

### Ca 3 (Panoptic): Khắc phục lỗi nhầm lẫn nhãn `truck` vs `car` và thiếu lớp `road` (Hard Panoptic)
- **Trước khi sửa:** Ảnh `000000460147.jpg` bị bỏ quên không gán `truck` (nhầm sang car) và ảnh `000000350023.jpg` quên gán nhãn `road` $\rightarrow$ PQ của cả `truck` và `road` ban đầu đều rơi về **0.000 điểm**.
- **Sau khi bổ sung nhãn đúng:**
  - Lớp `truck` đạt PQ = **0.592** (SQ: 0.887, RQ: 0.667); lớp `road` đạt PQ = **0.415** (SQ: 0.830, RQ: 0.500).
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
  - IoU (Intersection over Union) là phép đo tính trên tổng thể diện tích pixel ($|A \cap B| / |A \cup B|$). Đối với các vật thể có diện tích lớn (như lòng đường, tòa nhà, bầu trời), phần lõi (core) chiếm đại đa số diện tích (>90–95%). Dù viền mép bị răng cưa, cọ vẽ lượn sóng hoặc lấn vài pixel vào vỉa hè, diện tích giao (Intersection) trên hợp (Union) vẫn có thể đạt mức rất cao (>0.85). Do đó, điểm IoU cao phản ánh độ phủ đúng vị trí nhưng chưa đảm bảo tính sắc nét và hoàn hảo về mặt hình học của đường biên.
- **Vì sao panoptic khó hơn instance?**
  - **Ràng buộc phủ kín không chồng lấn:** Instance segmentation chỉ quan tâm đến các vật thể đếm được (things) rời rạc và bỏ qua nền. Ngược lại, Panoptic segmentation đòi hỏi mọi pixel trong ảnh đều phải có đúng một nhãn (cả stuff như đường, vỉa hè, bầu trời lẫn things như xe, người) và tuyệt đối không được chồng lấn.
  - **Chỉ số PQ (Panoptic Quality) có luật trừ điểm kép:** $PQ = SQ \times RQ$. Để được tính là nhận diện đúng (True Positive), mask phải có $IoU > 0.5$ với Ground Truth. Chỉ cần vẽ sót một người đi bộ hoặc vẽ nhầm 1 chiếc xe tải thành xe con, chỉ số $RQ$ (Recognition Quality) sẽ bị phạt nặng nề, kéo sụt giảm điểm $PQ$ ngay lập tức (minh chứng là lần 1 bài Hard chỉ đạt 5.6đ, sau khi sửa đúng nhãn đã tăng lên 13.5đ).
- **Giải trình cờ nghi vấn (Anti-cheat flags):**
  - Kết quả kiểm tra từ hệ thống: Cả 3 bài nộp đều hiển thị `no anti-cheat flags`. Điểm số từng task (Easy: 18.2đ, Medium: 20.1đ, Hard: 13.5đ) nằm hoàn toàn trong ngưỡng tự nhiên của người gắn nhãn thủ công (IoU $\approx 0.80 - 0.81$, không chạm ngưỡng nghi vấn $\ge 0.985$). Kết quả phản ánh quá trình gán nhãn thực tế, có sự tiến bộ rõ rệt qua từng lần tự kiểm và sửa lỗi của học viên.
