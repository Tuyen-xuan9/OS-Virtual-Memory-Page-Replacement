# Virtual Memory Page Replacement Simulator

Ứng dụng  mô phỏng trực quan 3 thuật toán thay thế trang bộ nhớ ảo kinh điển: **FIFO**, **LRU** và **OPT (Optimal)**

## Mục lục

- [Tính năng chính](#-tính-năng-chính)
- [Cấu trúc thư mục](#-cấu-trúc-thư-mục)
- [Yêu cầu hệ thống](#-yêu-cầu-hệ-thống)
- [Cài đặt & Chạy](#-cài-đặt--chạy)
- [Hướng dẫn sử dụng](#-hướng-dẫn-sử-dụng)

## Tính năng chính

| Nhóm | Chi tiết |
|---|---|
| **3 thuật toán** | FIFO, LRU, OPT. |
| **Gantt Chart trực quan** | **animation step-by-step**, điều khiển Play/Pause/Reset/End. |
| **So sánh thuật toán** | Tab Comparison: Faults, Hits, Hit Rate, Exec Time. |
| **Nhập liệu linh hoạt** | Load file CSV có sẵn hoặc nhập tay. |
| **Xuất báo cáo CSV** | 2 chế độ: xuất từng thuật toán riêng lẻ, hoặc xuất đồng thời cả 3. |
| **Stress Test / Benchmark** | Script độc lập đo thời gian chạy 3 thuật toán với reference string tới 100,000 phần tử. |
| **Xử lý lỗi đầu vào** | Tự động bỏ qua ký tự không hợp lệ, số âm, kiểm tra `frame_size` trong khoảng 1–20. |


## 📁 Cấu trúc thư mục
```
├── main.py                        # Entry point 
│
├── algorithms/                    # Lõi thuật toán thay thế trang
│   ├── base.py                    #   ABC PageReplacementAlgorithm 
│   ├── fifo.py                    #   FIFO — deque
│   ├── lru.py                     #   LRU + LRUClock
│   ├── opt.py                     #   OPT/Belady 
│   └── registry.py                #   AlgoRegistry 
│
├── models/
│   └── step.py                    # Step — trạng thái 1 bước (frames, fault/hit,
│                                   #        hit_rate, next_use, to_dict/to_csv_row...)
│
├── utils/
│   └── file_handler.py            # Đọc input CSV, ghi output CSV, tạo sample/stress inputs
│
├── GUI/
│   ├── display.py                 # App (tk.Tk) 
│   ├── gantt.py                   # GanttTab
│   ├── compare.py                 # CompareTab 
│   └── widgets.py                 # Bảng màu (C), font helper (F), widget dùng chung
│                                   
│
├── unit_tests/
│   ├── test_algorithms.py         # TestRunner 
│   └── stress_runner.py           # Benchmark hiệu năng 
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
└── output/                        # File CSV mẫu kết quả sau khi export
    ├── output_FIFO.csv
    ├── output_LRU.csv
    └── output_OPT.csv
```

> ⚠️ **Lưu ý:** `main.py` import theo đường dẫn tuyệt đối từ gốc project (`from GUI.display import App`, `from algorithms.fifo import FIFO`, v.v.). Vì vậy cần giữ đúng cấu trúc thư mục và **luôn chạy chương trình từ thư mục gốc**.


## Yêu cầu hệ thống

- **Python 3.8+** 
- **Tkinter** — thường có sẵn trong Python chuẩn.
  - Windows/macOS: có sẵn.
  - Linux: nếu thiếu, cài thêm:
    ```bash
    sudo apt install python3-tk
    ```
- Không cần cài thêm thư viện ngoài.


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

### Chạy Unit Test độc lập (không cần GUI)

```bash
python unit_tests/test_algorithms.py
```

### Chạy Stress Test / Benchmark

```bash
python unit_tests/stress_runner.py
```

Kết quả được ghi vào `output/stress_results.csv` và `output/stress_summary.txt`.

## Hướng dẫn sử dụng

1. **Nhập dữ liệu**
   - Bấm **📂 Load CSV file** để chọn file `.csv` có sẵn, hoặc
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

<p align="center">
  <sub>OS_VME_08 — Group 5 — HCMUTRANS © 2026</sub>
</p>
