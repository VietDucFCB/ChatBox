# Chatbot Phản Hồi Chính Sách Bảo Mật (Privacy Policy Chatbot)

Dự án này giúp bạn xây dựng một **chatbot** có khả năng **cung cấp phản hồi liên quan đến chính sách bảo mật** của một trang web. Dưới đây là **Hướng dẫn chi tiết** gói gọn trong **một file duy nhất** (không bao gồm code), giúp bạn hiểu rõ cách thức cài đặt và vận hành.

---

## 1. Giới Thiệu

### 1.1 Mục Tiêu
- **Cào dữ liệu (crawl)** từ URL chỉ định, thu thập các khối thông tin quan trọng (ví dụ: thẻ H2 và nội dung kèm theo).
- **Sinh embeddings** nhờ mô hình nhúng (Embeddings) của **Gemini**.
- **Lưu trữ dưới dạng FAISS Index** để cho phép tìm kiếm tương đồng (similarity search) hiệu quả.
- **Sinh câu trả lời** từ một mô hình sinh (Generative Model) của Gemini dựa trên các đoạn văn bản liên quan.

### 1.2 Lợi Ích
- Tự động **tải** và **phân tích** chính sách bảo mật (hoặc nội dung tương tự) mà không cần cập nhật thủ công.
- Cho phép **trò chuyện** (chat) và **đặt câu hỏi** về chính sách bảo mật, trả lời chính xác theo thông tin thu được.
- Dễ dàng **mở rộng** sang tài liệu khác như điều khoản dịch vụ, FAQ, v.v.

---

## 2. Kiến Trúc Giải Pháp

1. **Web Crawling**  
   - Sử dụng **Selenium** (trình duyệt headless) để lấy toàn bộ nội dung từ trang đích.  
   - Trích xuất thông tin quan trọng (các `h2` và nội dung kèm theo).

2. **Embeddings & Lưu trữ FAISS**  
   - Chia nhỏ (chunk) văn bản bằng **RecursiveCharacterTextSplitter**.  
   - Gọi **Gemini Embeddings** để chuyển nội dung sang dạng vector.  
   - Lưu vector này vào **FAISS Index** để có thể thực hiện tìm kiếm theo độ tương đồng.

3. **Tạo Câu Trả Lời**  
   - Khi người dùng gửi truy vấn (`query`), hệ thống sẽ tìm các chunk liên quan nhất trong FAISS.  
   - Tính toán trọng số cho từng chunk dựa trên mức độ khớp từ khóa.  
   - Truyền nội dung liên quan cùng câu hỏi vào **mô hình sinh (Generative Model)** của Gemini, lấy phản hồi cuối.

4. **Chatbot CLI**  
   - Chạy giao diện dòng lệnh (CLI) để nhập câu hỏi.  
   - Dừng (nhập `quit`) khi người dùng muốn thoát.

---

## 3. Yêu Cầu Hệ Thống

- **Python 3.x**
- **Trình duyệt Google Chrome** (hoặc Chromium) kèm **Chromedriver** phiên bản tương thích
- **API Key** hợp lệ cho **Gemini** (Dịch vụ cung cấp mô hình Embeddings và Generative Model)

### 3.1 Thư Viện Phụ Thuộc

*(Bạn có thể liệt kê những thư viện cần thiết trong file `requirements.txt` để cài đặt nhanh.)*

- `selenium`
- `beautifulsoup4`
- `langchain` (cung cấp Embeddings base, VectorStore FAISS, TextSplitter, v.v.)
- `faiss-cpu` (hoặc `faiss-gpu` nếu có GPU)
- `requests`
- `genai` (thư viện gọi mô hình Gemini)
- `chromedriver-autoinstaller` *(tùy chọn)*

---
### 4. Quy Trình Chatbot
- Cào Dữ Liệu (Crawl)
- Selenium mở trang web ở chế độ headless.
- Trích xuất thẻ h2 và nội dung ngay sau nó.
- Lấy toàn bộ nội dung trang trong thẻ <body>, lưu vào concatenatedText.
- Tiền Xử Lý & Embeddings
- Ghép các đoạn text, chia nhỏ bằng RecursiveCharacterTextSplitter.
- Gọi GeminiEmbeddings để tạo vector.
- Lưu tất cả vectors vào FAISS Index.
- Người Dùng Đặt Câu Hỏi
- Hệ thống tìm những chunk tương tự nhất trong FAISS.
- Tăng trọng số cho chunk chứa nhiều từ khóa từ câu hỏi.
- Kết nối các chunk liên quan, gửi vào mô hình Gemini GenerativeModel cùng câu hỏi.
- Sinh Câu Trả Lời
- Mô hình trả về câu trả lời.
- Hiển thị cho người dùng.
### 5. Tùy Chỉnh & Mở Rộng
- URL Mục Tiêu: Có thể thay đổi để cào dữ liệu từ bất kỳ trang web nào (miễn là cấu trúc không quá khác biệt).
- Chunk Size: Có thể tùy chỉnh trong RecursiveCharacterTextSplitter để phù hợp với độ dài văn bản và tài nguyên máy.
- K: Số lượng đoạn tương tự nhất trong FAISS (k=15 chẳng hạn).
- Prompt Engineering: Sửa prompt tạo câu trả lời (chèn hướng dẫn, ngữ cảnh, format, v.v.).
- Giao Diện: Chuyển từ CLI sang giao diện web (Flask, FastAPI) hoặc chatbot tích hợp vào trang web.
### 6. Xử Lý Sự Cố
- Lỗi Không Tìm Thấy Chromedriver
- Kiểm tra phiên bản Chrome/Chromium và Chromedriver có trùng khớp không.
- Cân nhắc dùng chromedriver-autoinstaller.
- Lỗi Hết Hạn API Key
- Đảm bảo Gemini API Key còn hiệu lực.
- FAISS Index Quá Lớn
- Giảm k hoặc tăng chunk_size để bớt chunk.
- Kết Quả Chatbot Không Liên Quan
- Thử tối ưu prompt, kiểm tra logic weighting chunk.
### 7. Đóng Góp (Contributing)
Báo lỗi (issue) hoặc gửi Pull Request với tính năng mới, đề xuất nâng cao hiệu năng, cải thiện prompt, v.v.


