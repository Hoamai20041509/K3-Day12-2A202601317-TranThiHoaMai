# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay các dòng giữ chỗ bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Thị Hoa Mai  Mã học viên: 2A202601317

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Một tình huống cụ thể là khi tôi deploy service nhưng quên khai báo
> `AGENT_API_KEY` trên môi trường chạy. Vì trường này không có mặc định,
> Pydantic báo `ValidationError` và tiến trình dừng ngay, nên tôi biết cấu hình
> đang thiếu trước khi public API. Nếu để mặc định `"changeme"`, service vẫn
> chạy và người khác có thể đoán khóa, gọi `/ask` rồi sử dụng tài nguyên của tôi.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi thu được là:
>
> ```json
> {"event":"ask_completed","level":"info","timestamp":"2026-08-10T04:47:34.437548+00:00","user_id":"sv-exercise","tokens_in":5,"tokens_out":37,"cost_usd":2.295e-05}
> ```
>
> Từ log này tôi có thể lọc theo `user_id` và cộng `cost_usd` để biết user nào
> tiêu nhiều chi phí nhất. Tôi cũng có thể đếm event, token hoặc nhóm theo thời
> gian để theo dõi lưu lượng và tạo cảnh báo. Dòng `print("đã trả lời xong")`
> không có các trường có cấu trúc để thực hiện hai việc đó.

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
| 1 stage (bản đầu) | Không hoàn tất — Docker báo ổ đĩa đầy khi tải base image |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi đo được image multi-stage là 270 MB. Khi build lại bản một stage dùng
> `python:3.11`, Docker Desktop báo `Disk full` và trước đó credential helper
> báo `The paging file is too small`, nên phép đo single-stage không hoàn tất.
> Điều quan sát được là bản một stage phải tải base image đầy đủ và giữ luôn
> dependency cùng các thành phần không cần cho runtime, trong khi bản
> multi-stage chỉ copy `/install` từ builder sang `python:3.11-slim`. Vì vậy
> runtime image nhỏ hơn và không mang theo môi trường build.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Với Dockerfile hiện tại, khi chỉ sửa `app/main.py`, các layer base image,
> `WORKDIR`, `COPY requirements.txt` và `RUN pip install` vẫn được lấy từ cache
> vì requirements không đổi. Cache bị mất từ layer `COPY app ./app` và các
> layer đứng sau nó phải được tạo lại. Nếu đặt `COPY . .` trước `RUN pip
> install`, mọi thay đổi nhỏ trong source đều làm layer copy đổi, kéo theo pip
> phải cài lại toàn bộ thư viện, khiến build chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi rủi ro bắt đầu khi code Python có lỗ hổng cho phép thực thi lệnh. Kẻ
> tấn công lợi dụng lỗ hổng để chạy lệnh trong container; nếu tiến trình đang
> chạy bằng root thì lệnh đó cũng có quyền root trong container. Khi container
> còn được mount thư mục nhạy cảm, cấp capability hoặc cấu hình host không an
> toàn, quyền cao này làm hậu quả nghiêm trọng hơn và có thể ảnh hưởng host.
> Lệnh `USER appuser` chuyển tiến trình sang UID 10001, nên mã bị khai thác chỉ
> có quyền của user thường và bị chặn khỏi nhiều thao tác đặc quyền.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 request trong 2 giây: gửi 10 request vào
> `10:00:59`, sau đó bộ đếm theo phút reset ở `10:01:00`, rồi gửi tiếp 10
> request vào `10:01:01`. Cả hai phút riêng lẻ đều không vượt giới hạn 10,
> nhưng thực tế có 20 request trong khoảng thời gian rất ngắn. Sliding window
> 60 giây nhìn thấy cả hai nhóm nên không có kẽ hở này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limiter giới hạn số request của một user trong cửa sổ 60 giây, còn cost
> guard giới hạn tổng số tiền user đã tiêu trong tháng. Trường hợp rate limit
> cho qua nhưng cost guard chặn là request đầu tiên trong phút của một user đã
> dùng hết ngân sách tháng. Trường hợp ngược lại là user vẫn còn nhiều ngân
> sách nhưng gửi request thứ 11 trong 60 giây; cost guard còn cho phép nhưng
> rate limiter trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp hai endpoint, khi Redis mất kết nối thì cả ba container cùng kiểm tra
> Redis và trả health check 503. Orchestrator xem cả ba là không còn sống nên
> lần lượt restart chúng, làm toàn bộ instance bị rút khỏi phục vụ cùng lúc.
> Trong lúc Redis chưa trở lại, container mới vẫn tiếp tục fail hoặc bị restart.
> Khi Redis phục hồi, các container còn cần thời gian khởi động nên hệ thống có
> một khoảng không phục vụ traffic. Tách `/health` và `/ready` giúp process vẫn
> được xem là sống, còn load balancer chỉ tạm ngừng gửi request cho instance
> chưa kết nối được Redis.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi gọi hai lần với cùng `X-User-Id`, tôi quan sát `history_length` tăng từ 0
> lên 2 vì mỗi request trước đó thêm một message user và một message assistant
> vào Redis. Với nhiều instance cùng dùng Redis, con số vẫn tăng theo lịch sử
> chung dù request rơi vào container khác. Nếu dùng một dict Python riêng cho
> từng container, mỗi instance chỉ biết lịch sử của chính nó nên kết quả có thể
> nhảy không đều như 0, 0, 2, 0 hoặc giảm khi request chuyển container, thay vì
> tăng ổn định.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Tôi dùng phương án local fallback thay vì tạo tài nguyên cloud. Lỗi thực tế
> tôi gặp là `curl: (7) Failed to connect` khi gọi port 8001, vì container cũ
> vẫn dùng mapping/healthcheck ở port 8000 hoặc server chưa được khởi động. Tôi
> kiểm tra bằng `docker compose ps` và log của service, sau đó đồng bộ `PORT`,
> port mapping và healthcheck sang 8001 rồi chạy lại `docker compose up -d
> --build`. Sau khi sửa, `/health` và `/ready` đều trả 200, còn `/ask` không có
> API key trả 401 đúng yêu cầu. Tôi ghi rõ local fallback trong
> `DEPLOYMENT.md` thay vì khai báo một URL cloud không có thật.
