# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay các dòng chờ bên dưới bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Thu Phương  Mã học viên: 2A202602734

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu mình quên khai báo `AGENT_API_KEY` trên Railway, app dừng ngay lúc khởi động thay vì chạy với khóa mặc định ai cũng đoán được. Mình phát hiện lỗi cấu hình trước khi mở API công khai; nếu dùng `changeme`, deploy có thể trông như thành công nhưng người ngoài có thể gọi API bằng khóa đó và phát sinh chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log JSON mình nhận được từ Railway:
>
> ```json
> {"cost_usd":0.0000234,"timestamp":"2026-09-28T16:11:32.243742+00:00","tokens_in":4,"event":"ask_completed","message":"","tokens_out":38,"level":"info","user_id":"wrapup-log"}
> ```
>
> Mình có thể lọc request theo `user_id`/`event` để điều tra sự cố, và tổng hợp `cost_usd`/token theo user hoặc thời gian để theo dõi chi phí. Một câu `print` tự do không có các trường ổn định để truy vấn như vậy.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 269 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage để lại image `python:3.11` đầy đủ, cache tải pip và toàn bộ dependency trong cùng runtime image. Bản multi-stage dùng `python:3.14-slim`, cài dependency không lưu pip cache rồi chỉ copy phần cài đặt cần dùng sang runtime; các phần dư của builder không nằm trong image cuối. Hai image mình đo được lần lượt là 1.73 GB và 269 MB.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, layer cài dependency từ `requirements.txt` được dùng lại vì file requirements không đổi. Layer copy source bị làm mới; các bước runtime phía sau có thể chạy lại theo thứ tự Dockerfile. Nếu `COPY . .` đứng trước `RUN pip install`, mọi thay đổi source làm layer COPY đổi, khiến bước cài pip phía sau mất cache và phải chạy lại.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu có lỗ hổng cho phép thực thi lệnh trong Python, lệnh đó chạy với quyền của user đang chạy app. Khi container chạy root, kẻ tấn công có quyền root bên trong container; nếu khai thác thêm cấu hình/container escape hoặc mount/capability yếu, họ có thể tiếp cận quyền cao trên host. `USER appuser` giới hạn quyền của tiến trình ngay từ đầu, nên chiếm được app không đồng nghĩa có quyền root.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với giới hạn 10 request/phút theo đồng hồ, gửi 10 request ở giây 59 của phút này rồi 10 request ở giây 00-01 của phút kế tiếp sẽ tạo 20 request trong khoảng 2 giây mà bộ đếm đã reset giữa chừng.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong cửa sổ 60 giây; cost guard giới hạn tổng USD đã dùng trong tháng. Một request rất lớn có thể khiến chi phí vượt ngân sách dù user chưa chạm 10 request/phút, nên cost guard phải chặn. Ngược lại, nhiều request nhỏ có thể vẫn nằm dưới ngân sách tháng nhưng vượt 10 request/phút, nên rate limit chặn trước.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối làm cả ba instance báo unhealthy nếu `/health` cũng kiểm tra Redis. Orchestrator hiểu đó là process hỏng và restart cả ba; load balancer mất hết instance đang phục vụ. Khi Redis trở lại, cả cụm vẫn đang khởi động lại. Tách `/ready` giúp load balancer tạm ngừng gửi traffic mà không yêu cầu orchestrator restart; `/health` chỉ kiểm tra process.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis dùng chung, các request qua ba container vẫn nhìn cùng lịch sử và `history_length` tăng `0, 2, 4, 6, 8` trong lần kiểm tra. Nếu lưu vào dict trong RAM, mỗi container có lịch sử riêng: request chuyển sang container mới có thể thấy `history_length` thấp hoặc về 0, rồi tăng theo những request tình cờ quay lại đúng container đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần deploy CI đầu tiên báo `Unauthorized. Please check that your RAILWAY_TOKEN is valid and has access to the resource you're trying to use.` Mình mở log job Deploy trên GitHub Actions, rồi đối chiếu tài liệu Railway: `RAILWAY_TOKEN` cho `railway up` cần Project Token, không phải Account/Workspace Token. Mình tạo Project Token cho environment `production`, thay secret GitHub `RAILWAY_TOKEN`, rerun workflow và cả test, build, deploy lẫn smoke test đều xanh.
