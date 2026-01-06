# ChromaDB - Vector Database Storage

Thư mục chứa ChromaDB vector database cho RAG system.

## Nội dung

- `chroma.sqlite3` - SQLite database chứa metadata
- `<collection-id>/` - Thư mục chứa vector data

## Collection: ksa_project

Database chứa các documents đã được embed cho IT Career chatbot.

## Sử dụng

Backend sẽ tự động đọc database từ thư mục này.

**Lưu ý:** Không chỉnh sửa trực tiếp các file trong thư mục này.

## Ingest dữ liệu mới

Để thêm dữ liệu mới vào database, chạy script ingest từ backend:

```bash
cd ../backend
python ingest.py
```
