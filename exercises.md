# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Thị Thùy Dương  Mã học viên: 2A202602905

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy ứng dụng lên cloud, nếu tôi quên cấu hình `AGENT_API_KEY` mà code
> có khóa mặc định `"changeme"`, ứng dụng vẫn khởi động và endpoint `/ask` có
> thể bị người khác gọi bằng khóa dễ đoán đó, làm phát sinh chi phí. Việc không
> đặt giá trị mặc định khiến ứng dụng báo lỗi ngay khi khởi động, giúp tôi phát
> hiện cấu hình thiếu trước khi service nhận request và tránh vô tình công khai
> API.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log JSON tôi thu được khi kiểm tra CP1:
>
> ```json
> {"event": "cp1_verified", "level": "info", "timestamp": "2026-09-29T04:35:21.655352+00:00", "service": "day12-agent"}
> ```
>
> Với log có cấu trúc này, tôi có thể lọc và đếm các bản ghi theo `event`,
> `level` hoặc `service` thay vì phải tìm trong câu chữ tự do. Tôi cũng có thể
> dùng `timestamp` để sắp xếp sự kiện, thống kê số lần xảy ra trong một khoảng
> thời gian và thiết lập cảnh báo tự động. Dòng `print("đã trả lời xong")` không
> cung cấp các trường dữ liệu ổn định để thực hiện hai việc đó.

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

> Tôi build lại Dockerfile một stage ban đầu bằng `python:3.11` và đo được image
> 1.7 GB; image multi-stage dùng `python:3.11-slim` có kích thước 271 MB. Phần
> chênh lệch chủ yếu đến từ base image Python đầy đủ chứa nhiều gói hệ thống,
> công cụ và thành phần không cần cho runtime. Với multi-stage, stage cuối chỉ
> nhận các dependency đã cài từ builder cùng mã nguồn cần chạy, không mang theo
> toàn bộ môi trường build và các file trung gian.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi build lại mà source chưa đổi, Docker báo `CACHED` cho các layer cài
> dependency, copy source và tạo user. Với thứ tự hiện tại, nếu tôi sửa một ký
> tự trong `app/main.py`, các layer copy `requirements.txt`, chạy `pip install`
> và copy dependency từ builder vẫn được dùng lại; layer `COPY app ./app` và
> các layer runtime đứng sau nó phải tạo lại. Nếu đặt `COPY . .` trước
> `RUN pip install`, thay đổi nhỏ trong source sẽ làm mất cache của layer copy,
> khiến `pip install` phải tải và cài lại toàn bộ thư viện dù
> `requirements.txt` không thay đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng Python có lỗ hổng cho phép thực thi lệnh, kẻ tấn công có thể chạy
> lệnh với quyền của process trong container. Khi container chạy bằng root,
> người đó có quyền root trong container và có thể lợi dụng cấu hình sai, volume
> mount nhạy cảm hoặc lỗ hổng của container runtime để tác động tới host với
> quyền cao. Lệnh `USER appuser` làm process chỉ chạy với UID 10001 không đặc
> quyền, nên ngay từ bước thực thi lệnh qua lỗ hổng, quyền của kẻ tấn công đã bị
> giới hạn. Tôi đã kiểm tra image và nhận được
> `uid=10001(appuser) gid=10001(appuser)`.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 request trong khoảng 2 giây: gửi 10 request
> vào giây cuối của phút hiện tại, rồi gửi tiếp 10 request ngay giây đầu của
> phút kế tiếp sau khi bộ đếm fixed window được reset. Mỗi phút riêng vẫn chỉ
> ghi nhận 10 request, nhưng lưu lượng thực tế bị dồn thành 20 request liên tiếp.
> Sliding window 60 giây ngăn được cách lách qua ranh giới phút này vì tại thời
> điểm kiểm tra, cả 20 request vẫn nằm trong cùng 60 giây gần nhất.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số lượng request trong một khoảng thời gian, còn cost
> guard giới hạn tổng chi phí mà một user đã sử dụng trong tháng. Ví dụ, user
> chỉ gửi 5 request trong một phút nên rate limit cho qua, nhưng mỗi request có
> prompt rất lớn và tổng chi phí đã vượt ngân sách tháng thì cost guard phải
> chặn. Ngược lại, user vẫn còn nguyên ngân sách nhưng gửi liên tục quá 10
> request trong 60 giây thì cost guard chưa cần chặn, còn rate limit phải trả
> lỗi 429 để bảo vệ service khỏi lưu lượng dồn dập.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp `/health` và `/ready` rồi cho endpoint đó kiểm tra Redis, khi Redis
> mất kết nối thì cả ba container cùng trả 503. Orchestrator sẽ hiểu rằng cả ba
> process bị lỗi và lần lượt restart chúng, dù bản thân ứng dụng vẫn còn sống.
> Trong lúc Redis chưa phục hồi, các container vừa khởi động lại vẫn tiếp tục
> báo lỗi và có thể bị restart thêm lần nữa. Kết quả là sự cố 30 giây của một
> dependency biến thành thời gian gián đoạn của toàn bộ cụm. Tách riêng hai
> probe cho phép `/health` tiếp tục báo process còn sống, còn `/ready` chỉ rút
> các instance chưa phục vụ được ra khỏi load balancer mà không restart chúng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Tôi chạy ba container agent lần lượt trên các cổng 8000, 8001 và 8002, cùng
> kết nối tới một Redis, rồi gửi năm request với cùng `X-User-Id`. Dù request
> đi qua các container khác nhau, `history_length` tăng đều `0, 2, 4, 6, 8`,
> chứng minh mọi instance cùng đọc một lịch sử trong Redis. Nếu dùng dict
> Python, mỗi container sẽ có một bản lịch sử riêng: lần đầu đi vào từng
> container thường đều trả 0, và con số chỉ tăng khi request quay lại đúng
> container đã xử lý trước đó. Người dùng vì thế sẽ thấy lịch sử tăng không
> liên tục hoặc có vẻ bị mất ngẫu nhiên.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn*
