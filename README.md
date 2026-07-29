# Marketing AI Agent (n8n)

Hệ thống AI Agent tự động hóa marketing cho **ISTAL – Viện Khoa học Công nghệ và Pháp luật**, xây dựng trên n8n. Người dùng nhắn tin (chữ hoặc voice) qua Telegram, một agent trung tâm sẽ hiểu yêu cầu và tự động giao việc cho đúng workflow con để tạo ảnh, video, bài blog, bài social hoặc làm SEO.

## 1. Kiến trúc tổng quan

```
Telegram (voice/text)
        │
        ▼
   MAIN_MARKETING  ← workflow chính (bộ não điều phối)
        │  (gọi các workflow con như "tool")
        ├── Create_Image        → tạo ảnh mới
        ├── Edit_image          → sửa ảnh có sẵn
        ├── blog_post           → viết bài blog + ảnh minh họa
        ├── social_content      → viết bài Facebook/LinkedIn
        ├── product_content_workflow → nội dung trang sản phẩm
        ├── seo_task            → nghiên cứu từ khóa, audit SEO, topical map
        └── video_faceless      → tạo video POV không lộ mặt (YouTube Shorts)
```

`MAIN_MARKETING` không tự làm nội dung — nó chỉ **đọc yêu cầu, chọn đúng tool, rồi gọi workflow con tương ứng** (mô hình AI Agent + Tool trong n8n).

## 2. Các file workflow

| File | Vai trò |
|---|---|
| `MAIN_MARKETING.json` | Workflow chính. Nhận tin nhắn Telegram (chữ hoặc voice), phân loại, điều phối tới các tool bên dưới bằng AI Agent (Google Gemini). |
| `Create_Image.json` | Nhận brief → viết prompt ảnh chi tiết → gọi Imagen 4 để tạo ảnh → lưu Google Drive + log vào Google Sheets → gửi ảnh qua Telegram. |
| `Edit_image.json` | Nhận ảnh (từ Drive hoặc URL ngoài) + yêu cầu chỉnh sửa → tải ảnh → tạo ảnh mới theo yêu cầu → lưu Drive + log Sheets → gửi qua Telegram. |
| `blog_post.json` | Nhận chủ đề + đối tượng độc giả → nghiên cứu → viết bài blog chuẩn SEO → tạo ảnh minh họa → lưu vào Google Docs, log Sheets, gửi Telegram. |
| `product_content_workflow.json` | Viết nội dung giới thiệu sản phẩm kèm ảnh, lưu Google Docs/Sheets. |
| `social_content.json` | Viết bài đăng Facebook/LinkedIn theo chủ đề + đối tượng, kèm ảnh minh họa. |
| `seo_task.json` | 4 agent con: nghiên cứu từ khóa, tạo topical map, audit SEO, và viết title/URL/meta description. |
| `video_faceless.json` | Tạo kịch bản 3 phần → gọi model Veo 3.1 tạo video → ghép/dựng → lưu Drive → gửi Telegram. |

## 3. Luồng hoạt động (ví dụ)

1. Người dùng nhắn Telegram: *"Viết cho tôi 1 bài blog về AI trong sản xuất"*.
2. `MAIN_MARKETING` nhận tin nhắn → AI Agent (Gemini) hiểu ý định → gọi tool **blogPost**.
3. Workflow `blog_post` chạy: nghiên cứu chủ đề → viết bài → tạo ảnh minh họa → lưu Google Docs, log vào Google Sheets, gửi bản nháp qua Telegram để duyệt.
4. Kết quả (link Docs, link ảnh) được trả lại cho người dùng qua Telegram.

Tương tự với "tạo ảnh", "sửa ảnh", "viết bài social", "làm SEO", "tạo video"…

## 4. Yêu cầu để chạy được

- Một instance **n8n** (self-host hoặc cloud) đã bật node cộng đồng `@n8n/n8n-nodes-langchain`.
- Import đủ **8 file JSON** vào cùng một n8n instance (các workflow con được `MAIN_MARKETING` gọi theo `workflowId`, nên cần import trước và giữ nguyên ID, hoặc cập nhật lại `workflowId` sau khi import).
- Credentials cần cấu hình lại trong n8n (không đi kèm trong file JSON, phải tự kết nối):
  - **Telegram Bot API** (nhận/gửi tin nhắn, ảnh, video)
  - **Google Drive OAuth2** (lưu ảnh/video)
  - **Google Sheets OAuth2** (log lịch sử tạo nội dung)
  - **Google Docs OAuth2** (lưu bài blog/nội dung)
  - **Google Gemini (PaLM) API** (agent + sinh ảnh Imagen 4 + video Veo 3.1)

## 5. ⚠️ Lưu ý bảo mật quan trọng

Trong `Create_Image.json` và `Edit_image.json`, node gọi Imagen đang **gắn cứng API key Google (`x-goog-api-key`) trực tiếp trong file**. Trước khi dùng lại hoặc chia sẻ file này, bạn nên:
- Thu hồi/tạo lại key đó trên Google Cloud Console (vì đã lộ trong file).
- Chuyển key sang lưu dưới dạng **Credential** trong n8n thay vì hard-code trong node, để tránh lộ khi chia sẻ workflow.

## 6. Thông tin thương hiệu đã cấu hình sẵn trong agent

- **Tên:** Viện Khoa học Công nghệ và Pháp luật (ISTAL)
- **Website:** https://khoahocvaphapluat.vn
- **Đối tượng mục tiêu:** cơ quan nhà nước, nhà hoạch định chính sách, chuyên gia pháp lý, doanh nghiệp, nhà nghiên cứu quan tâm đến giải pháp pháp lý – công nghệ – tiêu chuẩn.

Có thể chỉnh phần này trong `systemMessage` của node **AI Marketing Agent** (trong `MAIN_MARKETING.json`) nếu áp dụng cho thương hiệu khác.