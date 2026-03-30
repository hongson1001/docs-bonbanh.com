# Kế hoạch chi tiết: Website mua bán ô tô kiểu Bonbanh

**Tham chiếu sản phẩm:** [Bonbanh.com](https://bonbanh.com/)  
**Stack đề xuất:** Angular (Tailwind CSS + NG-ZORRO) · NestJS · PostgreSQL (hoặc MySQL) · Object storage (ảnh)  
**Phạm vi báo giá:** Chỉ phát triển phần mềm (code). Không gồm domain, VPS, CDN, phí cổng thanh toán, Zalo OA/SMS, thiết kế UI độc quyền từ đầu.

---

## 1. Đánh giá stack: Angular + Tailwind + NG-ZORRO + NestJS

| Thành phần | Đánh giá |
|------------|----------|
| **NestJS** | Phù hợp: API REST, JWT, upload file, module hoá, dễ scale; team VN dễ tuyển. |
| **Angular** | Phù hợp: app lớn, form nhiều (đăng tin), cấu trúc rõ; SEO cần SSR (Angular Universal) nếu muốn giống Bonbanh về Google — nên ghi rõ vào scope. |
| **NG-ZORRO** | Phù hợp: bảng, form, modal, date picker, admin; giảm thời gian UI. |
| **Tailwind** | Phù hợp: layout, spacing, responsive; **lưu ý:** trộn với theme NG-ZORRO cần quy ước (layout/spacing = Tailwind, component = ZORRO, tránh double style). |

**Kết luận:** Stack **khả thi và phổ biến** cho marketplace dạng classified. Nên bổ sung: **Angular SSR (Universal)** cho trang public (danh sách tin, chi tiết tin, salon) nếu mục tiêu SEO giống Bonbanh; admin có thể CSR.

---

## 2. Phân tích chức năng Bonbanh (baseline để “giống”)

### 2.1 Khu vực công khai (không đăng nhập)

- Trang chủ: khu vực tìm kiếm nhanh, shortcut hãng xe, vùng/miền/tỉnh, tin nổi bật (nếu có).
- **Danh sách tin bán:** lọc theo toàn quốc / tỉnh, hãng, (mở rộng) giá, năm SX, loại xe cũ-mới, sắp xếp; **phân trang** (15/20/30/60 tin/trang); hiển thị tổng số tin.
- **Chi tiết tin bán:** tiêu đề, mã tin, năm, dòng xe, giá, địa điểm, mô tả, thông số (nhập/lắp ráp, màu, nhiên liệu, dung tích, số chỗ, km, hộp số…), nhiều ảnh, thông tin liên hệ (tên, địa chỉ, SĐT nếu policy cho phép), badge xác thực (nếu có).
- **Tin mua (cần mua):** danh sách + chi tiết tương tự luồng riêng.
- **Salon ô tô:** danh sách salon, trang chi tiết salon (giới thiệu, địa chỉ, tin đang bán).
- **VIP Showroom:** block/trang liệt kê salon VIP (ưu tiên hiển thị).
- Trang **Giá xe ô tô** (nếu Bonbanh có — thường là nội dung/tin tức hoặc bảng giá): clone theo dạng **CMS bài viết / bảng giá thủ công** trong MVP+.
- **Tìm kiếm** text + bộ lọc kết hợp.
- Trang tĩnh: **Hướng dẫn, Liên hệ, Quy định đăng tin, Quy chế, Bảo mật, Trợ giúp, FAQ** (CMS hoặc markdown admin).
- **Footer:** thông tin công ty, hotline, copyright.
- **Cookie / thông báo** (banner) — tối thiểu thông báo như Bonbanh.
- **SEO:** meta title/description, URL thân thiện, sitemap, canonical; (tuỳ chọn) trang **từ khóa** dạng landing (danh sách link internal).

### 2.2 Tài khoản người dùng

- Đăng ký / đăng nhập / quên mật khẩu (email hoặc SMS — **SMS tính phí tích hợp + nhà mạng**, không gồm trong giá code thuần).
- Đăng nhập bằng username/email/số điện thoại (theo mô tả Bonbanh).
- **Đăng tin bán xe:** form đầy đủ field + upload ảnh (giới hạn số ảnh/dung lượng), lưu nháp / gửi duyệt.
- **Đăng tin mua xe:** form riêng.
- **Quản lý tin của tôi:** sửa, ẩn, xoá, gia hạn (nếu có policy ngày hết hạn).
- **Hồ sơ cá nhân / salon:** nếu user là salon — liên kết hồ sơ salon.

### 2.3 Gói VIP / thương mại (giống Bonbanh)

- Đăng ký **thành viên VIP** / bảng giá (hiển thị gói).
- **Giới hạn số tin** theo gói; **ưu tiên hiển thị** / gắn nhãn VIP trên listing.
- **Thanh toán online:** tích hợp 1 cổng (VNPay hoặc MoMo) — **phí cổng + tài khoản merchant do khách hàng**; code chỉ tích hợp API + webhook.

### 2.4 Uy tín / xác minh

- Upload giấy tờ (đăng ký, đăng kiểm) — lưu trữ bảo mật, chỉ admin xem.
- Trạng thái: chờ duyệt / đạt / từ chối; hiển thị **badge** trên tin khi đạt.

### 2.5 Admin / vận hành

- Duyệt / từ chối tin (bán & mua), lý do.
- Quản lý user: khoá/mở, phân quyền.
- CRUD **hãng xe**, **dòng xe** (model), **tỉnh/thành**, **miền** (cấu trúc cây).
- Quản lý salon, gán VIP, thứ tự hiển thị.
- Quản lý gói VIP & đơn hàng thanh toán (đối soát thủ công + webhook).
- Quản lý nội dung trang tĩnh, FAQ.
- Báo cáo cơ bản: số tin mới/ngày, user mới, doanh thu gói (nếu có).
- (Tuỳ chọn) nhật ký audit hành động admin.

### 2.6 Kỹ thuật & phi chức năng

- API có **phân trang, filter, sort** ổn định; index DB cho các cột lọc thường dùng.
- Upload ảnh: resize/compress phía server hoặc client; lưu object storage.
- Rate limit đăng tin / đăng nhập; chống spam cơ bản (captcha tuỳ chọn).
- Email transactional (đăng ký, quên mật khẩu, duyệt tin) — dùng SMTP/SendGrid v.v. (**chi phí dịch vụ email không gồm trong giá code**).

---

## 3. Kiến trúc tổng quan (gợi ý)

```
[User] → Angular (Public SSR tuỳ chọn + Admin CSR) — Tailwind + NG-ZORRO
              ↓ HTTPS / JSON
         NestJS API (Modules: Auth, Users, Listings, BuyRequests, Salons, VIP, Payments, Admin, Media, CMS)
              ↓
         PostgreSQL + Redis (session/cache/rate limit tuỳ chọn)
              ↓
         S3-compatible storage (ảnh + giấy tờ)
```

---

## 4. Mô hình dữ liệu lõi (rút gọn)

- `users` (role: user | salon_admin | admin)
- `provinces`, `regions` (Bắc/Trung/Nam hoặc nhóm cha-con)
- `brands`, `models`
- `listings` (type: new/used, FK brand/model/province/user/salon, price, year, specs JSON hoặc cột tách, status: draft/pending/active/rejected/expired, verified_flag)
- `buy_requests` (tương tự listing nhưng luồng “cần mua”)
- `salons` (profile, user_id, vip_level, sort_order)
- `listing_images`, `verification_documents`
- `vip_plans`, `subscriptions` hoặc `orders` + `payments`
- `cms_pages`, `faqs`
- `audit_logs` (tuỳ chọn)

---

## 5. Kế hoạch triển khai theo giai đoạn (deliverable rõ ràng)

### Giai đoạn 0 — Chuẩn bị (1–2 tuần)

- Wireframe / checklist màn hình bám Bonbanh; quy ước Tailwind + NG-ZORRO.
- Khởi tạo repo: Angular app (public + admin lazy routes), NestJS, Docker compose (local DB).
- CI cơ bản (lint + build).

### Giai đoạn 1 — Nền tảng & danh mục (2–3 tuần)

- Auth NestJS (JWT refresh), user CRUD; seed tỉnh/thành + hãng/dòng (import JSON).
- Admin: quản lý danh mục; Angular admin shell (ZORRO layout).

### Giai đoạn 2 — Tin bán (MVP vận hành) (4–6 tuần)

- API listing + filter + phân trang; upload ảnh; workflow duyệt tin.
- Angular public: danh sách, chi tiết, đăng tin, quản lý tin của tôi.
- Angular admin: duyệt tin, user.

### Giai đoạn 3 — Tin mua + Salon (3–4 tuần)

- `buy_requests` end-to-end; danh sách salon, chi tiết salon, gắn tin với salon.
- Block VIP showroom trên homepage.

### Giai đoạn 4 — VIP + thanh toán (3–5 tuần)

- Gói VIP, giới hạn tin, ưu tiên sort; tích hợp 1 cổng thanh toán + webhook.
- Trang bảng giá VIP.

### Giai đoạn 5 — Xác minh giấy tờ + CMS + SEO (2–4 tuần)

- Upload + duyệt giấy tờ + badge.
- CMS trang tĩnh, FAQ; sitemap, meta; (tuỳ hợp đồng) Angular Universal cho public.

### Giai đoạn 6 — cứng hoá & bàn giao (2 tuần)

- Test chức năng chính, fix bug; tài liệu API (Swagger), hướng dẫn deploy, bàn giao mã nguồn.

**Tổng lịch tham khảo (1 fullstack + 1 frontend hoặc 2–3 dev):** ~17–26 tuần làm việc cho **full parity** gần Bonbanh (có VIP + thanh toán + xác minh + CMS). Rút scope thì rút thời gian tương ứng.

---

## 6. Rủi ro & cách tránh “lỗ” khi báo giá rẻ

- **Scope SEO (SSR):** Universal tăng chi phí; nếu gói rẻ thì thỏa thuận **CSR + prerender một số route** hoặc làm SSR ở phase sau.
- **Thanh toán + kế toán đối soát:** luôn tính thêm buffer test + sandbox merchant.
- **Ảnh & lưu trữ:** object storage + CDN là phần hạ tầng (khách hàng trả); code phải có abstraction (interface storage).
- **Bonbanh có lịch sử 2006+:** không cam kết “giống 100% mọi corner case”; ký kết theo **danh sách chức năng đính kèm** (checklist section 2).

---

## 7. Báo giá tham khảo (chỉ code) — **mức hợp lý, không ép rẻ quá mức**

Giả định: đội có kinh nghiệm Angular/NestJS; UI dựa NG-ZORRO + Tailwind (không thiết kế Figma độc quyền); 2 vòng chỉnh sửa trong từng phase; bảo hành bug **30–60 ngày** sau bàn giao phase (thương lượng).

| Gói | Phạm vi (tóm tắt) | Thời gian (tuần) | Giá (VNĐ) — tham khảo |
|-----|-------------------|------------------|-------------------------|
| **A — MVP vận hành** | Tin bán: CRUD + duyệt + lọc tỉnh/hãng/năm/giá + phân trang + ảnh + user + admin cơ bản + trang tĩnh tối thiểu. **Không** VIP/thanh toán, **không** tin mua, **không** salon, **không** xác minh giấy. CSR (không Universal). | 10–14 | **185.000.000 – 265.000.000** |
| **B — Giống Bonbanh “đủ dùng”** | Gói A + **tin mua** + **salon** + VIP showroom (hiển thị thủ công) + CMS đầy đủ hơn + badge xác minh (upload + duyệt). Vẫn **chưa** cổng thanh toán tự động (VIP kích hoạt thủ công admin). SSR **tuỳ chọn** (+25–45tr nếu tách phase). | +8–12 | **+155.000.000 – 240.000.000** (cộng gói A) |
| **C — Gần parity đầy đủ** | Gói B + **gói VIP** + **1 cổng thanh toán** (VNPay hoặc MoMo) + webhook + quản lý đơn; tối ưu sort ưu tiên VIP; **Angular Universal** cho public (SEO). | +6–10 | **+135.000.000 – 220.000.000** (cộng gói A+B tương ứng) |

**Tổng gói C (ước lượng trọn gói):** khoảng **475.000.000 – 725.000.000 VNĐ** nếu làm xuyên suốt full scope như bảng trên (một team nhỏ, không redesign luxury).

**“Rẻ mà không lỗ” với freelancer solo:** chỉ nên nhận **Gói A** trong **10–14 tuần** với giá sàn ~**180–220tr** nếu họ tự làm fullstack và chấp nhận ít vòng chỉnh sửa; dưới mức đó rủi ro cắt scope hoặc nợ kỹ thuật.

---

## 8. Ngoài phạm vi (để không tranh cãi khi báo giá)

- Domain, VPS, SSL, backup, WAF, CDN, email SMTP, SMS OTP, tài khoản merchant thanh toán.
- Nội dung pháp lý, hợp đồng người dùng, DMCA, bảo vệ dữ liệu cá nhân (tư vấn luật).
- App mobile native; chat real-time nội bộ; AI gợi ý giá.
- Migrate/nhập dữ liệu từ hệ thống cũ; content writing.

---

## 9. Checklist nghiệm thu (trích để đưa vào hợp đồng)

- [ ] Danh sách tin bán: lọc đa điều kiện + phân trang 15/20/30/60.
- [ ] Chi tiết tin: đủ field thông số như mẫu Bonbanh (theo bảng field đính kèm).
- [ ] Đăng tin / sửa / ẩn; admin duyệt / từ chối.
- [ ] Đăng ký / đăng nhập / quên mật khẩu.
- [ ] (Theo gói) Tin mua, salon, VIP, thanh toán, xác minh, CMS, SSR.
- [ ] Swagger/OpenAPI; build Docker; README deploy.

---

**Tài liệu này** dùng để trao đổi với đội dev / freelancer: đính kèm **gói A/B/C**, **có/không SSR**, **có/không thanh toán**, và số vòng chỉnh sửa để chốt giá cuối.

*Cập nhật: tháng 3/2026.*
