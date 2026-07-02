# Virtual Memory Page Replacement Simulator

Ứng dụng  mô phỏng trực quan 3 thuật toán thay thế trang bộ nhớ ảo kinh điển: **FIFO**, **LRU** và **OPT (Optimal)**

---

## Mục lục

- [Tính năng chính](#-tính-năng-chính)
- [Cấu trúc thư mục](#-cấu-trúc-thư-mục)
- [Yêu cầu hệ thống](#-yêu-cầu-hệ-thống)
- [Cài đặt & Chạy](#-cài-đặt--chạy)
- [Hướng dẫn sử dụng](#-hướng-dẫn-sử-dụng)
- [Định dạng file CSV](#-định-dạng-file-csv)
- [Unit Test & Stress Test](#-unit-test--stress-test)
- [Tài liệu tham khảo](#-tài-liệu-tham-khảo)

---

## Tính năng chính

| Nhóm | Chi tiết |
|---|---|
| **3 thuật toán** | FIFO, LRU (`OrderedDict` O(1)), OPT (Belady, precompute `next_use` O(n)). Có thêm biến thể `LRU_CLOCK` (Second-Chance xấp xỉ LRU) trong lõi thuật toán. |
| **Gantt Chart trực quan** | **animation step-by-step** với thanh tốc độ tuỳ chỉnh (30–600 ms/step), điều khiển Play/Pause/Reset/End. |
| **So sánh thuật toán** | Tab Comparison: card thống kê từng thuật toán (Faults, Hits, Hit Rate, Exec Time). |
| **Nhập liệu linh hoạt** | Load file CSV có sẵn hoặc nhập tay reference string + frame size ngay trên GUI. |
| **Xuất báo cáo CSV** | 2 chế độ: xuất từng thuật toán riêng lẻ, hoặc xuất đồng thời cả 3. |
| **Stress Test / Benchmark** | Script độc lập đo thời gian chạy 3 thuật toán với reference string tới 100,000 phần tử, xuất `stress_results.csv` + `stress_summary.txt`. |
| **Xử lý lỗi đầu vào** | Tự động bỏ qua ký tự không hợp lệ, số âm, kiểm tra `frame_size` trong khoảng 1–20, báo lỗi rõ ràng qua message box. |

---

## 📁 Cấu trúc thư mục

```
├── main.py                        # Entry point — chạy: python main.py
│
├── algorithms/                    # Lõi thuật toán thay thế trang
│   ├── base.py                    #   ABC PageReplacementAlgorithm (interface chung)
│   ├── fifo.py                    #   FIFO — deque, O(1) evict
│   ├── lru.py                     #   LRU (OrderedDict) + LRUClock (Second-Chance)
│   ├── opt.py                     #   OPT/Belady — precompute next_use O(n)
│   └── registry.py                #   AlgoRegistry — tra cứu thuật toán theo tên
│
├── models/
│   └── step.py                    # Step — trạng thái 1 bước (frames, fault/hit,
│                                   #        hit_rate, next_use, to_dict/to_csv_row...)
│
├── utils/
│   └── file_handler.py            # Đọc input CSV, ghi output CSV, tạo sample/stress inputs
│
├── GUI/
│   ├── display.py                 # App (tk.Tk) — cửa sổ chính, điều phối toàn bộ UI
│   ├── gantt.py                   # GanttTab — vẽ bảng Gantt + animation + tooltip
│   ├── compare.py                 # CompareTab — bar chart + bảng so sánh 3 thuật toán
│   └── widgets.py                 # Bảng màu (C), font helper (F), widget dùng chung
│                                   #   (PillStat, IconButton, ScrollableCanvas, Tooltip...)
│
├── unit_tests/
│   ├── test_algorithms.py         # TestRunner — đối chiếu giáo trình, Belady, Step model
│   └── stress_runner.py           # Benchmark hiệu năng (1k → 100k refs)
│
├── input/                         # File CSV đầu vào (mẫu + test case)
│   ├── input.csv
│   ├── basic_testcase.csv
│   ├── belady_test.csv
│   ├── stress_test_1.csv / _2.csv / _3.csv
│   ├── testcase_no_page_fault.csv
│   ├── testcase_large_frames.csv
│   ├── testcase_extreme_page_fault.csv
│   ├── testcase_invalid_characters.csv
│   └── testcase_invalid_negative_frame.csv
│
└── output/                        # File CSV kết quả sau khi export
    ├── output_FIFO.csv
    ├── output_LRU.csv
    └── output_OPT.csv
```

> ⚠️ **Lưu ý:** `main.py` import theo đường dẫn tuyệt đối từ gốc project (`from GUI.display import App`, `from algorithms.fifo import FIFO`, v.v.). Vì vậy cần giữ đúng cấu trúc thư mục và **luôn chạy chương trình từ thư mục gốc**.

---

## Yêu cầu hệ thống

- **Python 3.8+** 
- **Tkinter** — thường có sẵn trong Python chuẩn.
  - Windows/macOS: có sẵn.
  - Linux: nếu thiếu, cài thêm:
    ```bash
    sudo apt install python3-tk
    ```
- Không cần cài thêm thư viện ngoài (không dùng `matplotlib`, `pandas`,... — biểu đồ được vẽ thủ công bằng `tkinter.Canvas`).

---

## Cài đặt & Chạy

```bash
# 1. Clone / giải nén project, di chuyển vào thư mục gốc
cd OS_VME_08

# 2. (Tuỳ chọn) Tạo virtual environment
python -m venv venv
source venv/bin/activate      # Linux/macOS
venv\Scripts\activate         # Windows

# 3. Chạy chương trình
python main.py
```

Khi khởi động, chương trình tự tạo `input/input.csv` và `input/belady_test.csv` mẫu nếu chưa tồn tại, đồng thời tự động nạp `input/input.csv` làm dữ liệu mặc định.

### Chạy Unit Test độc lập (không cần GUI)

```bash
python unit_tests/test_algorithms.py
```

### Chạy Stress Test / Benchmark

```bash
python unit_tests/stress_runner.py
```

Kết quả được ghi vào `output/stress_results.csv` và `output/stress_summary.txt`.

---

## Hướng dẫn sử dụng

1. **Nhập dữ liệu**
   - Bấm **📂 Load CSV file** để chọn file `.csv` có sẵn (xem [định dạng](#-định-dạng-file-csv)), hoặc
   - Kéo thanh trượt **FRAME SIZE** (1–10) và gõ trực tiếp reference string (cách nhau bằng dấu phẩy) vào ô **REFERENCE STRING**.

2. **Chạy thuật toán**
   - Bấm **▶ Run FIFO / LRU / OPT** để chạy riêng lẻ từng thuật toán → tự động chuyển sang tab **Gantt Chart**.
   - Bấm **⚡ Run ALL & Compare** để chạy cả 3 cùng lúc → tự động chuyển sang tab **Comparison**.

3. **Xem Gantt Chart**
   - Di chuột lên từng ô để xem tooltip chi tiết (frame state, fault/hit, next use...).
   - Dùng thanh điều khiển **⏮ Reset / ▶ Play / ⏸ Pause / ⏭ End** và slider tốc độ để xem animation từng bước thay thế trang.

4. **So sánh (Comparison tab)**
   - Xem thẻ thống kê, biểu đồ cột (Faults / Hits / Hit Rate) và bảng xếp hạng theo số fault thấp nhất.

5. **Xuất kết quả**
   - **💾 Export FIFO/LRU/OPT CSV** — xuất kết quả của 1 thuật toán đã chạy.
   - **📦 Xuất 3 CSV** — xuất đồng thời cả 3 thuật toán (yêu cầu đã bấm **Run ALL** trước) vào 1 thư mục do bạn chọn.

6. **Unit Tests**
   - Tab **Unit Tests** → bấm **▶ Run All Unit Tests** để xem toàn bộ test case PASS/FAIL ngay trong ứng dụng.

7. **Thông tin nhóm**
   - Bấm nút **👥 Nhóm 5 - Info** ở góc trên bên phải để xem danh sách thành viên.

---

## Định dạng file CSV

### File đầu vào (input)

```
<frame_size>
<ref_1>, <ref_2>, <ref_3>, ...
```

- **Dòng 1**: số nguyên, frame size hợp lệ trong khoảng `1–20`.
- **Dòng 2**: chuỗi tham chiếu trang, các số nguyên **không âm** cách nhau bằng dấu phẩy.
- Ký tự không hợp lệ (chữ, số âm) sẽ tự động bị **bỏ qua** kèm cảnh báo ở console, không làm crash chương trình.

Ví dụ (`input/basic_testcase.csv`):
```
3
7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2
```

### File đầu ra (output)

File CSV xuất gồm 2 phần:

1. **Header block** — tên thuật toán, frame size, tổng số reference, tổng fault/hit, hit rate, fault rate, thời gian thực thi, và toàn bộ reference string.
2. **Bảng chi tiết từng bước** — cột `Step, Page, Fault/Hit, Evicted, Frame_1..N, Total_Faults, Total_Hits, Hit_Rate(%), Fault_Rate(%), Next_Use`.

Ví dụ trích từ `output/output_FIFO.csv`:

```
=== VIRTUAL MEMORY PAGE REPLACEMENT SIMULATOR ===
OS_VME_08 - Group 5 - HCMUTRANS

Algorithm,FIFO
Frame Size,3
Total References,12
Total Faults,10
Total Hits,2
Hit Rate (%),16.67
Fault Rate (%),83.33
Exec Time (ms),0.097

Reference String,7,0,1,2,0,3,0,4,2,3,0,3

Step,Page,Fault/Hit,Evicted,Frame_1,Frame_2,Frame_3,Total_Faults,Total_Hits
1,7,FAULT,-,7,-,-,1,0
2,0,FAULT,-,7,0,-,2,0
3,1,FAULT,-,7,0,1,3,0
4,2,FAULT,7,2,0,1,4,0
5,0,HIT,-,2,0,1,4,1
...
```

---

## Unit Test & Stress Test

### Unit Test (`unit_tests/test_algorithms.py`)

Bộ test bao gồm:

- **Đối chiếu giáo trình**: dùng ví dụ chuẩn `[7,0,1,2,0,3,0,4,2,3,0,3]`, `frame_size=3` từ *Operating System Concepts 10th Edition* (trang 400–403), kiểm tra tính chất tối ưu **OPT ≤ LRU ≤ FIFO**.
- **Edge cases**: reference string rỗng, `frame_size=1`, toàn bộ trang giống nhau, toàn bộ trang duy nhất, `frame_size <= 0` phải raise `ValueError`.
- **Belady's Anomaly**: kiểm tra FIFO với chuỗi `[1,2,3,4,1,2,5,1,2,3,4,5]` — 3 frame cho 9 fault, 4 frame cho 10 fault (tăng ngược đời khi tăng frame), đồng thời xác nhận OPT **không** bị hiện tượng này.
- **LRU_CLOCK**: kiểm tra độ lệch số fault so với LRU chính xác (cho phép ≤ 30% do là thuật toán xấp xỉ).
- **Step model**: `step_index`, `hit_rate + fault_rate == 1.0`, `to_dict()`, `to_json()`, `to_csv_row()`, so sánh `__eq__`.
- **OPT next_use**: kiểm tra trường `next_use` được gán đúng.

Chạy độc lập:
```bash
python unit_tests/test_algorithms.py
```

Hoặc chạy trực tiếp trong GUI qua tab **Unit Tests**.

### Stress Test (`unit_tests/stress_runner.py`)

Benchmark hiệu năng với ma trận:

- **Reference size**: 1,000 / 5,000 / 10,000 / 50,000 / 100,000
- **Frame size**: 3 / 5 / 10 / 20
- **3 thuật toán** × 3 lần lặp (`REPEAT=3`, lấy `min` thời gian bằng `time.perf_counter`)

Kết quả xuất ra:
- `output/stress_results.csv` — dữ liệu thô (faults, hits, hit rate, min/avg/max ms).
- `output/stress_summary.txt` — tóm tắt nhanh nhất/chậm nhất mỗi thuật toán + so sánh tại mốc 100k references.

### Bộ test case CSV có sẵn (`input/`)

| File | Mục đích |
|---|---|
| `input.csv`, `basic_testcase.csv` | Test case cơ bản (dựa trên ví dụ giáo trình) |
| `belady_test.csv` | Kiểm tra hiện tượng Belady's Anomaly |
| `testcase_no_page_fault.csv` | Reference lặp lại tuần hoàn đúng bằng frame size → không phát sinh fault sau vòng đầu |
| `testcase_large_frames.csv` | Frame size lớn (20) |
| `testcase_extreme_page_fault.csv` | Frame size = 1, toàn trang khác nhau → fault tối đa |
| `testcase_invalid_characters.csv` | Chứa ký tự không hợp lệ (`a`, `b`, `c`...) để test khả năng lọc bỏ |
| `testcase_invalid_negative_frame.csv` | Frame size âm → test validate lỗi |
| `stress_test_1/2/3.csv` | Dữ liệu ngẫu nhiên (seed cố định = 42) cho stress test quy mô 1k/10k/50k references |
---

## Tài liệu tham khảo

- Silberschatz, A., Galvin, P. B., & Gagne, G. — *Operating System Concepts*, 10th Edition, Chapter 9 (Virtual Memory), trang 400–410.

---

<p align="center">
  <sub>OS_VME_08 — Group 5 — HCMUTRANS © 2026</sub>
</p>
