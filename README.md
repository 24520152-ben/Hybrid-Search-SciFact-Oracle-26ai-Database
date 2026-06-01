# Thực nghiệm Hybrid Search với Tập dữ liệu SciFact trên Oracle 26ai Database

## 1. Giới thiệu
Thực nghiệm này triển khai và đánh giá hiệu suất của các phương pháp truy xuất thông tin (Information Retrieval) trên tập dữ liệu khoa học chuyên biệt **SciFact**. Bằng việc tận dụng nền tảng **Oracle 26ai Database** (tích hợp AI Vector Search và Oracle Text Search), hệ thống so sánh ba phương pháp truy xuất nhằm tìm ra chiến lược tối ưu để trích xuất bằng chứng khoa học phục vụ cho các mô hình RAG (Retrieval-Augmented Generation):
- **Semantic Search**
- **Keyword Search**
- **Hybrid Search**

## 2. Thông tin Tập dữ liệu (SciFact)
- **SciFact Corpus:** Gồm 5.183 tài liệu tóm tắt (abstracts) được lấy từ các tạp chí khoa học uy tín (Cell, Nature, JAMA, BMJ...). Trong đó có chứa các "tài liệu nhiễu" (distractor abstracts) để tăng tính thử thách cho mô hình truy xuất.
- **Queries:** Bộ truy vấn (gồm 300 câu ở tập test) được dùng để xác minh các luận điểm khoa học (scientific claim verification).
- **Nhiệm vụ:** Trích xuất chính xác các tài liệu làm bằng chứng (evidence abstracts) có khả năng hỗ trợ (SUPPORTS) hoặc bác bỏ (REFUTES) luận điểm đầu vào.

## 3. Cấu trúc Repository
```text
.
├── docs/
│   └── chapter6.pdf               # Tài liệu báo cáo/cơ sở lý thuyết chi tiết
├── notebooks/
│   ├── 01_ingest.ipynb            # Tiền xử lý dữ liệu (chunking), tạo embedding và lưu vào Oracle Database
│   ├── 02_semantic_search.ipynb   # Thực nghiệm tìm kiếm ngữ nghĩa với HNSW index
│   ├── 03_keyword_search.ipynb    # Thực nghiệm tìm kiếm từ khóa với Oracle Text Search
│   └── 04_hybrid_search.ipynb     # Thực nghiệm phương pháp tìm kiếm lai và so sánh kết quả
├── scifact/
│   ├── qrels/
│   │   ├── test.tsv               # Nhãn kết quả tham chiếu cho tập test (300 queries)
│   │   └── train.tsv             
│   ├── corpus.jsonl               # Kho ngữ liệu 5.183 tài liệu khoa học
│   └── queries.jsonl              # Tập các câu truy vấn
├── .gitignore                    
└── requirements.txt               # Các thư viện phụ thuộc
```

## 4. Công nghệ & Kỹ thuật sử dụng
- **Cơ sở dữ liệu:** Oracle Database có hỗ trợ AI Vector Search (Oracle 26ai).
- **Embedding model:** `sentence-transformers/all-MiniLM-L6-v2` từ HuggingFace.
- **Framework:** LangChain (`langchain-oracledb`, `langchain-huggingface`).
- **Text chunking:** Kỹ thuật chia nhỏ tài liệu dựa trên cấu trúc tự nhiên bằng thuật toán `RecursiveCharacterTextSplitter` với `chunk_size=500` và `chunk_overlap=50`, giúp giữ được mạch văn và tối ưu hóa cửa sổ ngữ cảnh (context window).

## 5. Kết quả Thực nghiệm
Dự án được đánh giá trên 300 truy vấn thuộc tập test. Thước đo hiệu năng là chỉ số **Hit@K** (tỷ lệ tài liệu truy xuất đúng nằm trong Top K). Kết quả thu được như sau:

| Phương pháp | Hit@5 | Hit@10 | Hit@50 |
|---|---|---|---|
| **Keyword Search** (Từ khóa) | ~59.67% | 65.67% | 78.67% |
| **Semantic Search** (Ngữ nghĩa) | ~73.33% | 78.67% | 88.67% |
| **Hybrid Search** (Tìm kiếm lai) | **~78.67%** | **82.33%** | **92.00%** |

**Kết luận:** Phương pháp **Hybrid Search** cho thấy sự vượt trội so với các phương pháp đơn lẻ ở mọi cấu hình thực nghiệm. Việc kết hợp giữa cơ chế hiểu ngữ nghĩa sâu của AI Vector Search và khả năng so khớp từ vựng chính xác của Oracle Text giúp hệ thống xử lý tốt các biến thể đồng nghĩa và những thuật ngữ chuyên ngành phức tạp trong miền tri thức khoa học.

## 6. Hướng dẫn cài đặt và sử dụng

**Bước 1: Clone dự án**

Tải mã nguồn dự án về máy:
```bash
git clone https://github.com/24520152-ben/Hybrid-Search-SciFact-Oracle-26ai-Database
cd Hybrid-Search-SciFact-Oracle-26ai-Database
```

**Bước 2: Chuẩn bị môi trường Cơ sở dữ liệu**
- Cài đặt **Oracle 26ai Database** thông qua máy ảo **VirtualBox** và phần mềm quản lý **SQL Developer**. 
- Vui lòng tham khảo kỹ **`chapter6.pdf`** trong thư mục `docs/` để biết chi tiết các bước thiết lập, cấu hình RAM (vector memory area) và fix lỗi tablespace (ASSM).

**Bước 3: Cài đặt thư viện Python**
Cài đặt các gói thư viện cần thiết (khuyên dùng Python 3.13):
```bash
pip install -r requirements.txt
```

**Bước 4: Cấu hình thông tin kết nối Cơ sở dữ liệu**
- Bật máy ảo Oracle trên VirtualBox, mở terminal của máy ảo và gõ lệnh `ip a` để lấy địa chỉ IP thực tế của cơ sở dữ liệu.
- Mở các file notebook trong thư mục `notebooks/` và cập nhật biến `dsn` bằng địa chỉ IP vừa lấy được, kèm theo `username` và `password` tương ứng của bạn:
```python
username = "system"
password = "oracle"
dsn = "<ĐỊA_CHỈ_IP_TỪ_LỆNH_IP_A>:1521/FREEPDB1"
```

**Bước 5: Chạy các Notebook (Thực thi theo thứ tự)**
- **`01_ingest.ipynb`**: Khởi tạo kết nối, đọc dữ liệu, chunking, tạo embedding và lưu vào bảng `SciFact` trong Oracle Database.
- **`02_semantic_search.ipynb`**: Khởi tạo HNSW index và chạy thử nghiệm đánh giá độ chính xác của Semantic Search.
- **`03_keyword_search.ipynb`**: Khởi tạo text index và kiểm tra hiệu suất của Keyword Search (fuzzy search).
- **`04_hybrid_search.ipynb`**: Chạy truy xuất hỗn hợp (Hybrid Search) và tính toán các chỉ số so sánh.