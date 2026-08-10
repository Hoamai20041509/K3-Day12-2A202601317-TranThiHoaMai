# Thông Tin Deploy — Checkpoint 5

> Bài lab đang dùng phương án local fallback được quy định trong hướng dẫn.
> Không có giá trị bí mật nào được lưu trong tài liệu này.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trần Thị Hoa Mai |
| Mã học viên | 2A202601317 |
| Repo | https://github.com/Hoamai20041509/K3-Day12-2A202601317-TranThiHoaMai |

## Service

| Mục | Nội dung |
|-----|----------|
| Địa chỉ kiểm tra | http://localhost:8001 |
| Platform | Local Docker Compose fallback; cloud platform dự kiến: Railway |
| Ngày kiểm tra | 2026-08-10 |

## Biến Môi Trường

Chỉ liệt kê tên biến và nguồn cấu hình, không ghi giá trị bí mật.

| Biến | Đã set | Nguồn |
|------|--------|-------|
| `PORT` | Có | Docker Compose |
| `AGENT_API_KEY` | Có | File `.env` không được Git theo dõi |
| `REDIS_URL` | Có | Docker Compose, trỏ tới service Redis |
| `RATE_LIMIT_PER_MINUTE` | Có | File `.env` / Docker Compose |
| `MONTHLY_BUDGET_USD` | Có | File `.env` / Docker Compose |
| `LOG_LEVEL` | Có | File `.env` / Docker Compose |
| `LOCAL_FALLBACK` | Có | File `.env` không được Git theo dõi |

## Kết Quả Chạy Thật

Stack được khởi động bằng `docker compose up -d --build --wait`.

```text
agent: Up (healthy), cổng 8001:8001
redis: Up (healthy), cổng 6379:6379

GET  http://localhost:8001/health -> 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET  http://localhost:8001/ready -> 200
{"status":"ready","redis":true}

POST http://localhost:8001/ask (không có X-API-Key) -> 401
{"detail":"invalid or missing API key"}
```

Docker image production:

```text
Repository=day12-agent Tag=prod Size=270MB
```

## Ảnh Chụp

- `screenshots/health.png`: ảnh chụp endpoint `/health` đang phục vụ từ container.

## Lý Do Dùng Phương Án Dự Phòng

Phiên làm việc không có thông tin đăng nhập hoặc ủy quyền tạo tài nguyên trên
nền tảng cloud. Vì vậy bài được kiểm tra bằng Docker Compose local theo phương
án fallback trong hướng dẫn. Khi có tài khoản Railway, cần deploy image này,
gắn Redis add-on và thay địa chỉ local bằng Public URL HTTPS thật.
