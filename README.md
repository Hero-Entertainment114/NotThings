# PsychoClass • Zulip Edition

PsychoClass là ứng dụng Node.js dùng GroqCloud tạo 10 câu hỏi, phản hồi từng câu và tổng kết. Khi người tham gia đồng ý, máy chủ tải báo cáo TXT lên Zulip rồi gửi một tin nhắn vào channel đã cấu hình. Email qua Resend vẫn có thể làm phương án dự phòng nếu người tham gia đồng ý riêng.

## Yêu cầu

- Node.js 20 trở lên.
- GroqCloud API key.
- Một Zulip organization/server và bot API credentials.
- (Tuỳ chọn) Resend API key, email gửi đã xác minh và địa chỉ nhận để bật dự phòng email.

## 1. Tạo channel nhận kết quả trong Zulip

1. Đăng nhập vào Zulip của bạn.
2. Tạo channel, ví dụ `psychoclass-reports`. Tên này sẽ được đặt vào `ZULIP_CHANNEL`.
3. Giới hạn thành viên được xem channel vì báo cáo có thể chứa thông tin riêng tư. Nếu cần, đặt channel ở chế độ private và chỉ mời những người thực sự cần đọc.

## 2. Tạo bot cho ứng dụng

1. Bấm biểu tượng bánh răng → **Personal settings** → **Bots** (hoặc **Organization settings** → **Bots** nếu bạn là quản trị viên).
2. Bấm **Add a new bot**.
3. Chọn bot loại **Generic** vì phiên bản này cần gọi API tải tệp TXT lên rồi đăng liên kết vào channel. Chỉ dùng bot riêng cho PsychoClass.
4. Ghi lại email bot và API key. Không dán API key vào HTML, Netlify, GitHub công khai hoặc tin nhắn. Người có API key có thể mạo danh bot.
5. Thêm bot vào channel `psychoclass-reports` và bảo đảm bot được phép đăng tin. Nếu Zulip của bạn có chính sách quyền riêng tư riêng, hãy kiểm tra bot truy cập được channel đích.

Tài liệu Zulip: [thêm bot](https://zulip.com/help/add-a-bot-or-integration), [quản lý bot/API key](https://zulip.com/help/manage-a-bot), [gửi tin nhắn qua API](https://zulip.com/api/send-message), [tải tệp lên](https://zulip.com/api/upload-file).

## 3. Cấu hình khi chạy trên máy tính

Sao chép `.env.example` thành `.env`, sau đó điền các giá trị:

```dotenv
GROQ_API_KEY=YOUR_GROQ_API_KEY
GROQ_MODEL=llama-3.3-70b-versatile
ZULIP_SITE_URL=https://your-org.zulipchat.com
ZULIP_BOT_EMAIL=psychoclass-bot@your-org.zulipchat.com
ZULIP_API_KEY=YOUR_ZULIP_BOT_API_KEY
ZULIP_CHANNEL=psychoclass-reports
ZULIP_TOPIC=PsychoClass - Báo cáo khảo sát
PORT=3000
```

Thay toàn bộ giá trị ví dụ. `ZULIP_SITE_URL` là địa chỉ gốc của tổ chức Zulip, không phải URL API hay URL webhook. `ZULIP_CHANNEL` phải trùng tên channel thật. `ZULIP_TOPIC` là topic của tin nhắn trong channel.

Chạy:

```bat
npm start
```

Mở `http://localhost:3000`. Kiểm tra `http://localhost:3000/api/health`; trường `zulipConfigured` phải là `true` khi cấu hình đủ. Endpoint này chỉ trả trạng thái, không tiết lộ API key.

## 4. Cấu hình trên Render

1. Mở Render → Web Service của PsychoClass → **Environment**.
2. Thêm các biến `ZULIP_SITE_URL`, `ZULIP_BOT_EMAIL`, `ZULIP_API_KEY`, `ZULIP_CHANNEL` và `ZULIP_TOPIC` như ví dụ ở trên.
3. Giữ `GROQ_API_KEY` và `GROQ_MODEL`. Nếu tiếp tục dùng email dự phòng, giữ thêm `RESEND_API_KEY`, `REPORT_EMAIL_FROM`, `REPORT_EMAIL_TO`.
4. Xóa biến `DISCORD_WEBHOOK_URL` nếu còn sót. Bản này không gửi qua Discord.
5. Lưu thay đổi và redeploy service.

Nếu Netlify chỉ phục vụ giao diện, đừng đặt Zulip API key lên Netlify hoặc trong `index.html`. Bí mật chỉ nằm ở phần **Environment** của dịch vụ backend Render. Đảm bảo cấu hình proxy trong `netlify.toml` vẫn trỏ tới backend Render của bạn.

## 5. Cách hoạt động

- Người tham gia tích **Tôi đồng ý gửi báo cáo đến Zulip** thì sau khi hoàn thành khảo sát, server tạo báo cáo TXT, upload lên Zulip và đăng liên kết cùng phần tổng quan vào channel/topic.
- Nếu Zulip thất bại, server chỉ gửi email khi người tham gia đã tích riêng lựa chọn email dự phòng và các biến Resend hợp lệ.
- Nếu không đồng ý gửi, báo cáo không được chuyển tới Zulip hoặc email; người tham gia vẫn có thể tải hoặc sao chép báo cáo trong trình duyệt.

## 6. Kiểm tra lần đầu

1. Mở `/api/health` và xác nhận `aiConfigured`, `zulipConfigured` là `true`.
2. Thử hoàn thành một khảo sát thử bằng nội dung không nhạy cảm.
3. Tích đồng ý gửi Zulip; nếu muốn kiểm tra đường dự phòng, tích cả ô email.
4. Kiểm tra channel `psychoclass-reports`, topic và liên kết TXT.
5. Nếu lỗi, xem Render → **Logs**. Log được thiết kế không ghi nội dung câu trả lời. Thông báo `channel does not exist` thường nghĩa là sai `ZULIP_CHANNEL`; lỗi 401/403 thường cần kiểm tra bot email/API key/quyền truy cập; lỗi upload thường cần kiểm tra bot quyền gọi API và giới hạn dung lượng của Zulip.

## Riêng tư và an toàn

Báo cáo gồm câu trả lời tự luận và phản hồi AI, vì vậy có thể nhạy cảm. Chỉ cho phép nhóm người thật sự cần thiết xem channel. Không sử dụng tên thật, thông tin nhận dạng hay câu trả lời nhạy cảm trong thử nghiệm. Thu hồi API key nếu vô tình đưa lên GitHub hoặc chia sẻ công khai. Ứng dụng không chẩn đoán tâm lý; nội dung AI có thể sai.
