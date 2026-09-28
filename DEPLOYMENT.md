# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trần Thu Phương |
| Mã học viên | 2A202602734 |
| Repo | https://github.com/TranThuPhuong1111/K4-L3A-DAY12-TranThuPhuong-2A202602734-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://agent-production-104f.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt bằng Railway CLI cho service `agent`; giá trị được giữ bí mật |
| `REDIS_URL` | ✅ | tham chiếu tới `REDIS_URL` của Railway Redis service |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Các lệnh gọi tới service đã deploy:

```bash
# 1. Liveness
curl -i https://agent-production-104f.up.railway.app/health
# HTTP/1.1 200 OK
# {"status":"ok","service":"day12-agent","version":"1.0.0"}

# 2. Readiness (Redis đã kết nối)
curl -i https://agent-production-104f.up.railway.app/ready
# HTTP/1.1 200 OK
# {"status":"ready","redis":true}

# 3. Không có API key
curl -i -X POST https://agent-production-104f.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
# HTTP 401 Unauthorized

# 4. Có API key (giá trị chỉ lấy từ .env local, không ghi vào tài liệu)
curl -i -X POST https://agent-production-104f.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $DEPLOY_API_KEY" \
  -H "X-User-Id: sv-test-doc" \
  -d '{"question":"What is deployment?"}'
# HTTP 200 OK; history_length=0

# 5. Rate limit — 10 request được chấp nhận, 5 request cuối bị giới hạn
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://agent-production-104f.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $DEPLOY_API_KEY" \
    -H "X-User-Id: sv-test-rate" \
    -d '{"question":"test"}'
done; echo
# 200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Kết Quả Chạy Thật

Kết quả kiểm tra trực tiếp trên Railway ngày 2026-09-28:

```
GET /health  -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready   -> 200 {"status":"ready","redis":true}
POST /ask without API key -> 401 Unauthorized
POST /ask with API key -> 200 OK; history_length=0
15 POST /ask with same user -> 200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

