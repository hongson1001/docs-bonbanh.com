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

## 7. Báo giá — **một gói duy nhất (Option 1)**

**Giá (chỉ code):** **15.000.000 – 20.000.000 VNĐ**  
**Tính chất giá:** đây là **mức đề xuất ban đầu** khi gửi khách / đối tác — **còn thương lượng (deal)** sau khi chốt phạm vi cuối (có/không bổ sung menu shell, số trang tĩnh, vòng chỉnh sửa, bảo hành).  
**Tên gói:** **Khởi động (Starter)** — đủ để demo luồng chợ xe + mở rộng sau; **không** tương đương toàn bộ [Bonbanh.com](https://bonbanh.com/) (mục 2).

### Điều kiện chung (để khớp mức giá)

- Stack: **Angular + Tailwind + NG-ZORRO** (frontend) · **NestJS + PostgreSQL** (API + DB).
- **Không** thiết kế Figma riêng; UI theo component có sẵn + chỉnh layout.
- Hỗ trợ **01 vòng** chỉnh sửa nhỏ trong phạm vi đã ký (fix lỗi hiển thị / text / lọc cơ bản).
- Bảo hành lỗi **14 ngày** kể từ ngày bàn giao mã nguồn (chỉ lỗi trong phạm vi gói).
- Bàn giao: mã nguồn + **README** chạy local (Docker Compose hoặc hướng dẫn `npm`); **không** gồm cấu hình VPS/domain thực tế (xem [DEPLOY.md](./DEPLOY.md) để khách tự hoặc thuê thêm).

### Phạm vi “đầy đủ” trong gói (nằm trong 15–20tr)

| Nhóm | Có trong gói |
|------|----------------|
| **Công khai** | Trang chủ (layout kiểu marketplace); **danh sách tin bán** có **phân trang**; **lọc** tối thiểu: **tỉnh/thành** + **hãng** (dữ liệu danh mục seed trong DB); **chi tiết tin** (tiêu đề, giá, mô tả, vài thông số cố định, ảnh — tối đa **5 ảnh**/tin, lưu **local disk** hoặc thư mục `uploads`, không tích hợp S3). |
| **Tài khoản** | **Đăng ký / đăng nhập** (JWT); **đăng tin bán** (form + upload ảnh); user **xem/sửa/xoá** tin của mình. |
| **Admin tối thiểu** | **01** role admin (hard-code hoặc flag DB): **duyệt tin** (chờ duyệt → hiển thị / từ chối), **danh sách user** (xem + khoá/mở đơn giản). |
| **API** | REST cơ bản + **Swagger** (OpenAPI). |
| **Dữ liệu** | Seed **tỉnh/thành** + **hãng** (bản rút gọn); **không** bắt buộc đủ mọi dòng xe Bonbanh. |

### Không nằm trong gói 15–20tr (mở rộng phase sau)

- Tin **cần mua**, **salon**, **VIP**, **thanh toán online**, **xác minh giấy tờ**, CMS trang tĩnh đầy đủ, **Angular Universal (SSR)**, email/SMS production, chat, tối ưu SEO nâng cao, import dữ liệu lớn, CDN/object storage.

### Thời gian tham khảo

- **3–5 tuần** (1 người fullstack, có thể dùng AI hỗ trợ code — vẫn cần tích hợp, test, bàn giao).

### Cách báo giá với khách (tránh hiểu nhầm)

- Ghi rõ: *“Gói Khởi động — 15–20 triệu — nền tảng tin bán + duyệt tin, không gồm các mục … (liệt kê như trên).”*
- Nếu đã **deal thêm** phần 7.1: ghi *“đã bao gồm / không bao gồm”* từng dòng trong bảng 7.1 vào phụ lục hợp đồng.

### 7.1 Bổ sung nhanh — **menu & khung trang giống Bonbanh** (deal sau)

Mục này để **lấp nhanh “khung” điều hướng** cho demo / báo giá mở rộng; khi deal có thể **gộp vào cùng gói** (nếu thời gian đủ) hoặc **cộng thêm phí / phase 2** — do bên dev & khách thỏa thuận.

| Thành phần (tham chiếu menu Bonbanh) | Bổ sung nhanh (ý nghĩa) | Ghi chú |
|--------------------------------------|-------------------------|---------|
| **Thanh trên:** Đăng ký · Đăng nhập · Hướng dẫn · Liên hệ | Link **Hướng dẫn** / **Liên hệ** → **trang tĩnh** (1 nội dung placeholder do khách cung hoặc text mẫu) | Đăng ký/Đăng nhập đã có trong lõi gói |
| **Menu ngang:** Trang chủ · Tìm mua ô tô · Salon Ô tô · Bán ô tô · Giá xe ô tô · Cần mua ? · My … | **Đủ route + nhãn menu**; **Trang chủ** = home; **Tìm mua ô tô** → trỏ **danh sách tin bán**; **Bán ô tô** → form **đăng tin bán**; **My** → **quản lý tin của tôi** | **Salon** / **Giá xe** / **Cần mua** → trang **shell** (“Đang cập nhật”) hoặc **nội dung tĩnh** — **chưa** backend đầy đủ trừ khi deal thêm |
| **Ô tìm kiếm + nút Tìm** (header) | **Tuỳ chọn deal:** search text đơn giản theo tiêu đề/mô tả **hoặc** chỉ UI + submit trỏ về trang list với `q=` | Làm nhanh thì 1 API `GET ?q=` |
| **Cột trái:** “Tìm theo hãng xe” | **Sidebar danh sách hãng** (seed) → click lọc như hiện có | Layout gần Bonbanh, không bắt buộc pixel-perfect |
| **Tab Tin bán / Tin mua** (khối giữa) | **Tin bán** = list thật; **Tin mua** = tab **disabled** / “Sắp ra mắt” **hoặc** trang shell | Full tin mua = ngoài gói lõi |
| **Tab địa điểm:** Toàn quốc · Hà Nội · TP HCM · … | **Tuỳ chọn deal:** shortcut set `provinceId` trên list (vài tỉnh hot + “Tất cả”) | Nhanh hơn dropdown-only |
| **Khối lọc nâng cao** (năm, số, slider giá…) | **Không** gói nhanh; deal **phase 2** | — |
| **Cột phải:** block CTA (đăng tin, salon, VIP…) | **Nút trỏ đúng route** đã có; mục chưa có chức năng → trang shell hoặc ẩn | “Thành viên cao cấp” = shell / ẩn |
| **Vip ShowRoom** | **Static block** (3–5 dòng + link) **hoặc** danh sách hard-code — **không** CMS salon thật | Deal sau: salon thật + VIP |
| **Footer:** Quy định · Quy chế · Bảo mật · Trợ giúp · Liên hệ | **5–6 trang tĩnh** (cùng template, nội dung placeholder) | Không CMS phức tạp trong bổ sung nhanh |
| **Trang đăng ký/đăng nhập** (nhiều field như Bonbanh) | **Form đăng ký tối thiểu** đã có; mở rộng field **theo deal** (địa chỉ, 2 SĐT…) | Mỗi field = thêm thời gian |

**Tóm tắt cho khách:** *“Lõi 15–20tr = tin bán + user + admin duyệt. Phần 7.1 = **làm nhanh khung menu + trang tĩnh + shell** cho giống Bonbanh; chức năng nặng (tin mua, salon DB, VIP, thanh toán…) **deal thêm**.”*

---

**Tham chiếu thị trường (gói lớn hơn, đã từng nêu trong tài liệu):** MVP vận hành đầy đủ hơn (nhiều filter, ảnh/object storage, vòng chỉnh sửa & bảo hành dài hơn) thường nằm khoảng **185–265 triệu** trở lên; gói **15–20tr** là **cắt scope có chủ đích**, không phải cùng một sản phẩm.

---

## 8. Ngoài phạm vi (để không tranh cãi khi báo giá)

- Domain, VPS, SSL, backup, WAF, CDN, email SMTP, SMS OTP, tài khoản merchant thanh toán.
- Nội dung pháp lý, hợp đồng người dùng, DMCA, bảo vệ dữ liệu cá nhân (tư vấn luật).
- App mobile native; chat real-time nội bộ; AI gợi ý giá.
- Migrate/nhập dữ liệu từ hệ thống cũ; content writing.

---

## 9. Checklist nghiệm thu

### Gói Khởi động 15–20tr (đưa vào hợp đồng)

- [ ] Danh sách tin: phân trang + lọc **tỉnh** + **hãng** (theo seed).
- [ ] Chi tiết tin: tiêu đề, giá, mô tả, thông số tối thiểu đã thống nhất, tối đa **5 ảnh**/tin.
- [ ] User: đăng ký / đăng nhập; đăng / sửa / xoá tin của mình.
- [ ] Admin: duyệt / từ chối tin; khoá / mở user cơ bản.
- [ ] Swagger/OpenAPI; README chạy local (hoặc Docker Compose).

### Bổ sung nhanh (mục 7.1 — chỉ tick nếu đã deal)

- [ ] Menu ngang đủ mục + route; các mục chưa có nghiệp vụ → shell / tĩnh theo bảng 7.1.
- [ ] Hướng dẫn, Liên hệ + footer (5–6 link) dạng trang tĩnh / placeholder.
- [ ] Sidebar “Tìm theo hãng” (layout) + lọc hãng.
- [ ] (Tuỳ deal) Ô search header; shortcut Hà Nội / TP.HCM / Toàn quốc trên list.
- [ ] (Tuỳ deal) Block VIP ShowRoom tĩnh hoặc hard-code.

### Mở rộng sau (không thuộc gói 15–20tr)

- [ ] Lọc nâng cao (giá, năm, km…), phân trang 15/20/30/60 tuỳ chọn, quên mật khẩu email.
- [ ] Tin mua, salon, VIP, thanh toán, xác minh giấy tờ, CMS đầy đủ, SSR.

---

**Tài liệu này** dùng để trao đổi với đội dev / freelancer: **một gói duy nhất (mục 7)** + **bổ sung nhanh menu (7.1)** nếu deal; giá **đề xuất**, chốt phạm vi & phí khi **deal**.

*Cập nhật: tháng 3/2026.*
