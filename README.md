# So sánh Stateful và Stateless Authentication

Trong phát triển web và API, việc **chọn mô hình xác thực** ảnh hưởng trực tiếp đến **bảo mật, trải nghiệm người dùng và khả năng mở rộng hệ thống**. Dưới đây là so sánh giữa **Stateful** và **Stateless** authentication.

---

## 1. Khái niệm

| Thuật ngữ | Định nghĩa |
|-----------|------------|
| **Stateful Authentication** | Server **lưu trạng thái phiên (session)** cho mỗi user. Client chỉ giữ **session ID** (thường trong cookie). |
| **Stateless Authentication** | Server **không lưu trạng thái**, mỗi request phải **gửi toàn bộ thông tin xác thực** (thường là JWT/access token). |

---

## 2. Cách lưu token / session

| Mô hình | Server lưu | Client lưu |
|---------|------------|------------|
| Stateful | Session object `{userId, roles, expires, ...}` | Session ID trong **cookie HttpOnly** |
| Stateless | Không lưu gì | **JWT / access token** trong `localStorage`, `sessionStorage`, cookie, hoặc memory |

**Ghi chú:**  
- Stateful → server nắm toàn quyền kiểm soát session.  
- Stateless → client phải gửi token đầy đủ, server chỉ verify.

---

## 3. Revoke / Auto Logout

| Mô hình | Khả năng revoke / logout |
|---------|-------------------------|
| Stateful | ✅ **Dễ dàng**: xóa session trên server → tất cả request từ session này bị từ chối ngay lập tức. |
| Stateless | ⚠ **Khó khăn**: token vẫn hợp lệ **đến khi hết hạn**, server không lưu token → không thể revoke tức thì. |

**Giải pháp cho Stateless:**  
- Short-lived access token + refresh token  
- Token blacklist / blocklist (nhưng lúc đó server trở nên stateful)  

---

## 4. Token / Thông tin nhạy cảm

| Mô hình | Khả năng lộ dữ liệu |
|---------|------------------|
| Stateful | Thông tin nhạy cảm **chỉ lưu trên server** → client chỉ giữ session ID. Cookie HttpOnly → chống XSS. |
| Stateless | Token thường **lưu trên client** (`localStorage`, `sessionStorage`, memory). Nếu dùng `localStorage`, dễ bị XSS đánh cắp. JWT chứa dữ liệu → dễ bị lộ nếu sniff request. |

---

## 5. Xử lý quyền và cập nhật trạng thái

| Mô hình | Thay đổi quyền / khóa user |
|---------|---------------------------|
| Stateful | ✅ Ngay lập tức: server update session → client bị hạn chế quyền ngay. |
| Stateless | ⚠ Chậm: JWT cũ vẫn hợp lệ → quyền thay đổi chỉ có hiệu lực sau khi token hết hạn. |

---

## 6. Khả năng scale

| Mô hình | Khả năng scale |
|---------|---------------|
| Stateful | Khó hơn: cần **shared session store** (Redis, DB) nếu deploy nhiều server. |
| Stateless | Dễ dàng: server không cần lưu gì → **scale horizontal dễ dàng** (microservices, cloud). |

---

## 7. Khuyến nghị bảo mật

| Mô hình | Best practice |
|---------|---------------|
| Stateful | - Cookie HttpOnly + Secure + SameSite  
- Giới hạn thời gian session  
- Xác thực CSRF token |
| Stateless | - Token ngắn hạn (5–15 phút) + refresh token  
- Không lưu token trong localStorage nếu có thể  
- Cookie HttpOnly + Secure cho refresh token  
- Token rotation và blacklist nếu cần revoke |

---

## 8. Tổng kết

| Tiêu chí | Stateful | Stateless |
|----------|----------|-----------|
| Server lưu trạng thái | ✅ Có | ❌ Không |
| Client gửi gì | Session ID | JWT / access token |
| Revoke / Logout | Dễ, ngay lập tức | Khó, chờ token hết hạn |
| Bảo vệ XSS | ✅ Tốt (HttpOnly cookie) | ⚠ Token có thể bị lấy nếu lưu localStorage |
| Quyền thay đổi ngay | ✅ Có | ⚠ Phải đợi token hết hạn hoặc rotate |
| Scale / Microservices | ⚠ Khó | ✅ Dễ |
| Độ phức tạp | Trung bình | Cao (cần refresh + rotation) |

---

**Kết luận:**  
- **Stateful**: phù hợp cho web app truyền thống, cần logout/revoke ngay, bảo mật cao.  
- **Stateless**: phù hợp cho API, microservices, mobile app, cần scale tốt.  
- **JWT**: mạnh mẽ nhưng dễ bị dùng sai → cần token ngắn hạn, refresh token, tránh localStorage.  

---

*Nguồn tham khảo:*  
- REST API Security Best Practices  
- OWASP Authentication Cheat Sheet  
- JSON Web Token Security Guidelines
