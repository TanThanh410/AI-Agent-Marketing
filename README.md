# 🚀 Marketing AI Agent (n8n)

Hệ thống AI Agent tự động hóa marketing đa kênh được xây dựng trên nền tảng **n8n**. 

Người dùng chỉ cần gửi yêu cầu (văn bản hoặc tin nhắn thoại) qua **Telegram**, Agent trung tâm sẽ tự động phân tích ý định và điều phối công việc đến các workflow con tương ứng để tạo ảnh, làm video, viết bài blog, bài đăng mạng xã hội (social) hoặc thực hiện các tác vụ SEO.

---

## 🏗️ 1. Kiến Trúc Tổng Quan

**`MAIN_MARKETING`** đóng vai trò là "Bộ nào điều phối" trung tâm. Workflow này không trực tiếp khởi tạo nội dung mà đọc yêu cầu người dùng, phân tích ngữ cảnh, lựa chọn công cụ (tool) phù hợp và kích hoạt workflow con tương ứng (áp dụng mô hình *AI Agent + Tools* trong n8n).

---

## 📁 2. Danh Sách Workflow & Vai Trò

| Tệp Workflow | Vai Trò & Chức Năng |
| :--- | :--- |
| **`MAIN_MARKETING.json`** | Workflow chính. Nhận tin nhắn Telegram (chữ hoặc voice), phân loại và điều phối tới các tool bằng AI Agent (Google Gemini). |
| **`Create_Image.json`** | Nhận brief → Viết prompt chi tiết → Gọi Imagen 4 sinh ảnh → Lưu Google Drive + Log Google Sheets → Gửi ảnh qua Telegram. |
| **`Edit_image.json`** | Nhận ảnh (Drive/URL) + Yêu cầu chỉnh sửa → Tải ảnh → Tạo ảnh mới theo yêu cầu → Lưu Drive + Log Sheets → Gửi qua Telegram. |
| **`blog_post.json`** | Nhận chủ đề + Đối tượng độc giả → Nghiên cứu → Viết bài blog chuẩn SEO → Tạo ảnh minh họa → Lưu Google Docs + Log Sheets → Gửi Telegram. |
| **`product_content_workflow.json`** | Viết nội dung giới thiệu sản phẩm kèm ảnh minh họa, tự động lưu vào Google Docs & Sheets. |
| **`social_content.json`** | Viết bài đăng Facebook/LinkedIn theo chủ đề + Đối tượng mục tiêu, kèm ảnh minh họa. |
| **`seo_task.json`** | Tích hợp 4 Agent con: Nghiên cứu từ khóa, tạo Topical Map, Audit SEO, và viết Title/URL/Meta Description. |
| **`video_faceless.json`** | Tạo kịch bản 3 phần → Gọi model Veo 3.1 tạo video → Ghép/Dựng video → Lưu Drive → Gửi Telegram. |

---

## 🔄 3. Luồng Hoạt Động (Ví dụ minh họa)

1. **Gửi yêu cầu:** Người dùng nhắn qua Telegram: *"Viết cho tôi 1 bài blog về AI trong sản xuất"*.
2. **Điều phối:** `MAIN_MARKETING` nhận tin nhắn → AI Agent (Gemini) phân tích ý định → Định tuyến và kích hoạt tool `blogPost`.
3. **Thực thi:** Workflow `blog_post` tiến hành:
   * Nghiên cứu chủ đề.
   * Viết bài blog chuẩn SEO.
   * Sinh ảnh minh họa.
   * Lưu trữ tệp vào Google Docs, ghi nhật ký vào Google Sheets, gửi bản nháp qua Telegram.
4. **Phản hồi:** Trả kết quả (link Google Docs, link ảnh) trực tiếp cho người dùng qua Telegram.

*(Quy trình tương tự được áp dụng cho các tác vụ "Tạo ảnh", "Sửa ảnh", "Bài Social", "SEO", "Tạo video"...)*

---

## ⚙️ 4. Yêu Cầu Cài Đặt & Cấu Hình

* **Nền tảng:** Cần một instance n8n (Self-host hoặc Cloud) đã kích hoạt community node `@n8n/n8n-nodes-langchain`.
* **Import Workflows:** Import đầy đủ 8 tệp JSON vào n8n. 
  > ⚠️ **Lưu ý:** `MAIN_MARKETING` gọi các workflow con bằng `workflowId`. Hãy giữ nguyên ID sau khi import hoặc cập nhật lại `workflowId` tương ứng trong node gọi tool.
* **Cấu hình Credentials (Kết nối):**
  * **Telegram Bot API:** Nhận/gửi tin nhắn, hình ảnh, video.
  * **Google Drive OAuth2:** Lưu trữ hình ảnh và video đã khởi tạo.
  * **Google Sheets OAuth2:** Ghi log lịch sử khởi tạo nội dung.
  * **Google Docs OAuth2:** Lưu nội dung bài viết blog/sản phẩm.
  * **Google Gemini (PaLM) API:** Cung cấp năng lực cho Agent, sinh ảnh Imagen 4 và tạo video Veo 3.1. *(Đảm bảo khai báo API Key trong Credentials/Environment của n8n thay vì dán trực tiếp trong node)*.

---

## 🏢 5. Cấu Hình Thông Tin Thương Hiệu

Bạn có thể tùy chỉnh thông tin thương hiệu phù hợp với doanh nghiệp của mình bằng cách thay đổi phần `systemMessage` trong node `AI Marketing Agent` thuộc tệp `MAIN_MARKETING.json`:

* **Tên tổ chức/thương hiệu:** [Tên thương hiệu của bạn]
* **Website:** `https://your-domain.com`
* **Đối tượng mục tiêu:** [Mô tả khách hàng/độc giả mục tiêu của bạn]