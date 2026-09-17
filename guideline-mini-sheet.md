# Phiếu quy tắc gán nhãn — Ngày 5 Segmentation

**Họ và tên:** Đoàn Văn Thắng &nbsp;·&nbsp; **MSSV:** 26A202602327 - T037

## 1. Ba loại bài — chọn đúng loại trước khi vẽ

| Loại | Câu hỏi | Đếm được không | Ví dụ lớp |
| --- | --- | --- | --- |
| **Semantic** | "pixel này là loại gì?" | không | road, sidewalk, sky (stuff) |
| **Instance** | "pixel này thuộc *vật nào*?" | có | car #1, car #2 (things) |
| **Panoptic** | cả hai: mọi pixel một nhãn, vật thì thêm ID | cả hai | road + car#1 + car#2 |

Chọn nhầm loại = làm lại. Easy = semantic, Medium = instance, Hard = panoptic.

## 2. Lớp cố định (đúng tên từng chữ — bộ chấm ghép theo tên)

**Stuff (semantic/panoptic):** `road`, `sidewalk`, `building`, `vegetation`, `sky`. (checkpoint semantic thêm: `pole`, `traffic sign`)
**Things (instance/panoptic):** `person`, `bicycle`, `car`, `motorcycle`, `bus`, `truck`, `traffic light`.

| Lớp | Gán khi | Không gán |
| --- | --- | --- |
| `road` | mặt đường xe chạy | vỉa hè, bó vỉa |
| `sidewalk` | vỉa hè, phần người đi | lòng đường |
| `car` | ô tô con, SUV, taxi | xe tải có thùng, bus |
| `truck` | có thùng/ben/sàn hàng rõ | ô tô con, van |
| `bus` | thân khách dài, nhiều cửa sổ | van nhỏ |

## 3. Quy tắc hình học (mọi loại)

- Biên **sát phần nhìn thấy**; không đoán phần bị che.
- **Lỗ thủng** (kính xe, khe): tính là một phần của vật — **không khoét** (trừ khi guideline bảo khoét).
- Vật bị vật khác **cắt làm đôi**: vẫn là **một** mask (brush làm được; polygon thì Join `J`).
- Hai vật sát nhau bị gộp một mask: **Slice `Alt+J`** để tách.
- **Vẽ từ xa đến gần**; biên chung chỉ vẽ một lần (Remove underlying pixels / z-order `[` `]`).

## 4. Semantic & Panoptic

- Tô kín, không để hai mask tranh cùng một pixel.
- **Bó vỉa (curb):** `road` và `sidewalk` cùng nhựa, khác chức năng — ranh giới là chỗ đường kết thúc.
- **Nét mảnh** (cột, biển báo): brush 2–3px.
- Panoptic: **mọi pixel đúng một nhãn**; pixel không quyết được → ghi vào nhật ký, để trống (giảm coverage) thay vì đoán bừa.

## 5. Ba tình huống mơ hồ (điền trước khi xem điểm)

### A — `car` hay `truck`/`van`? (pickup có thùng, minibus…)
- Ảnh và vị trí vật: Ảnh `000000460147.jpg` (Hard), vị trí các phương tiện lớn di chuyển ở làn xe phía sau/xa.
- Dấu hiệu nhìn thấy: Phần đuôi có kết cấu thùng hàng khối hộp kín và sàn xe chuyên chở hàng hóa rõ rệt, kích thước tổng thể lớn hơn hẳn ô tô con thông thường.
- Quy tắc áp dụng + quyết định: Áp dụng quy tắc tại Mục 2: "truck: có thùng/ben/sàn hàng rõ; car: ô tô con, SUV, taxi". Quyết định: Gán nhãn `truck`.

### B — Instance: hai vật hay một? (xe sát nhau / xe bị che cắt đôi)
- Ảnh và vị trí: Ảnh `000000373353.jpg` (Medium), vị trí 2 chiếc ô tô đỗ song song sát cạnh nhau trên lề đường.
- Slice hay Join? Vì sao: Chọn **Slice (Alt+J)**. Vì đây là hai phương tiện đếm được (instance) riêng biệt, nhưng khi dùng công cụ vẽ tự động/SAM dễ bị dính chung đường biên thành một khối. Cần dùng Slice để cắt rời thành 2 mask độc lập.

### C — Semantic: `road` hay `sidewalk` ở chỗ bó vỉa?
- Ảnh và vị trí: Ảnh `81ae7cbb-6bc63a4a.jpg` (Easy), đoạn mép vỉa hè tiếp giáp lòng đường xe chạy phía bên phải.
- Bằng chứng ở mức phóng 100%: Dù bề mặt đường và vỉa hè đều có màu xám bê tông/asphalt tương tự nhau, mức phóng 100% cho thấy rõ đường gờ bó vỉa nhô cao (curb line) và sự ngắt quãng của vạch kẻ mép đường xe chạy.
- Quyết định: Áp dụng quy tắc "Bó vỉa (curb)" tại Mục 4: Lấy ranh giới là mép kết thúc mặt đường xe chạy. Phần lòng đường gán `road`, gờ bó vỉa hắt vào trong phần cho người đi bộ gán `sidewalk`.

## 6. Tự kiểm tra

- [x] Đã chọn đúng loại phân vùng cho từng cấp.
- [x] Tên lớp đúng từng chữ như `classes.json`.
- [x] Không khoét lỗ thủng; không gộp/không tách nhầm instance.
- [x] Panoptic: không có pixel bị hai mask; đã kiểm coverage.
- [x] Đã tự chấm bằng `scoring/score.py` và sửa lớp/vật điểm thấp.
- [x] Không nộp ground truth, không sửa trực tiếp tệp xuất.
