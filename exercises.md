# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng giữ chỗ bên dưới mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Thị Thùy Dương  Mã học viên: 2A202602905

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu tui quên đặt `AGENT_API_KEY` khi deploy mà app vẫn dùng khóa mặc định
> `"changeme"`, người khác có thể dễ dàng đoán khóa và gọi `/ask`, làm phát sinh
> chi phí. Cho app dừng ngay giúp tui biết đang thiếu cấu hình và sửa trước khi
> mở service cho người dùng.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log JSON tui thu được khi kiểm tra CP1:
>
> ```json
> {"event": "cp1_verified", "level": "info", "timestamp": "2026-09-29T04:35:21.655352+00:00", "service": "day12-agent"}
> ```
>
> Từ log này, tui có thể lọc hoặc đếm theo `event`, `level`, `service`. Tui cũng
> có thể dùng `timestamp` để biết sự kiện xảy ra lúc nào và theo dõi lỗi theo
> thời gian. Một dòng `print("đã trả lời xong")` không có đủ các thông tin đó.

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
| 1 stage (bản đầu) | 1.7 GB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage của tui nặng 1.7 GB, còn bản multi-stage chỉ 271 MB. Bản đầu dùng
> image Python đầy đủ nên chứa nhiều công cụ và file không cần khi chạy app.
> Multi-stage chỉ đưa thư viện đã cài và source cần thiết sang image cuối nên
> nhẹ hơn nhiều.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi tui build lại, Docker báo `CACHED` cho các bước cài thư viện. Nếu chỉ sửa
> `app/main.py`, bước copy `requirements.txt` và `pip install` vẫn dùng cache;
> bước `COPY app ./app` và các bước sau nó phải chạy lại. Nếu đặt `COPY . .`
> trước `pip install`, chỉ một thay đổi nhỏ trong source cũng làm Docker cài lại
> toàn bộ thư viện, khiến build chậm hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app có lỗ hổng cho phép chạy lệnh, kẻ tấn công sẽ có cùng quyền với process
> trong container. Chạy bằng root làm mức thiệt hại có thể lớn hơn, nhất là khi
> container bị cấu hình sai hoặc mount file nhạy cảm từ host. `USER appuser`
> giới hạn process ở user thường ngay từ đầu. Tui kiểm tra được kết quả
> `uid=10001(appuser) gid=10001(appuser)`.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi 20 request trong 2 giây: 10 request ở giây cuối của phút
> trước và 10 request ở giây đầu của phút sau. Bộ đếm theo phút vừa reset nên
> vẫn cho qua. Sliding window nhìn lại đúng 60 giây gần nhất nên sẽ thấy cả hai
> nhóm request và chặn trường hợp này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong một khoảng thời gian, còn cost guard giới
> hạn số tiền đã dùng trong tháng. Ví dụ, 5 request có nội dung rất dài vẫn có
> thể vượt ngân sách nên cost guard chặn dù rate limit cho qua. Ngược lại, một
> user còn nhiều ngân sách nhưng gửi hơn 10 request trong 60 giây sẽ bị rate
> limit chặn với lỗi 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Khi Redis mất kết nối, cả ba container sẽ cùng trả 503. Nếu dùng chung một
> endpoint, hệ thống có thể hiểu nhầm cả ba app đã chết và restart chúng. Redis
> chưa phục hồi thì các container mới vẫn tiếp tục lỗi, làm cả cụm bị gián đoạn
> lâu hơn. Tách riêng giúp `/health` báo app vẫn sống, còn `/ready` chỉ tạm ngừng
> nhận request cho đến khi Redis hoạt động lại.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Tui chạy ba container ở cổng 8000, 8001 và 8002 rồi gửi request với cùng
> `X-User-Id`. `history_length` tăng đều `0, 2, 4, 6, 8` dù request đi qua các
> container khác nhau, vì chúng dùng chung Redis. Nếu lưu bằng dict Python, mỗi
> container có lịch sử riêng nên số này có thể quay lại 0 hoặc tăng không đều
> khi request chuyển sang container khác.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần đầu deploy, `/health` và `/ready` đều báo `502 Bad Gateway`. Tui xem log
> và thấy app đang chạy ở cổng 8080, nhưng domain Railway lại trỏ tới cổng 8000.
> Tui đổi target port của domain thành 8080. Sau đó `/health`, `/ready` trả 200;
> `/ask` không có key trả 401 và có key đúng thì trả 200.
