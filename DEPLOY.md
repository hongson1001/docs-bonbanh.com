# Hướng dẫn deploy

Tài liệu gồm hai phần: **(A)** đưa repo tài liệu lên GitHub, **(B)** deploy hệ thống thật (Angular + NestJS) lên VPS khi đã có mã nguồn ứng dụng.

---

## A. Đẩy repo docs lên GitHub

### Điều kiện

- Đã cài [Git for Windows](https://git-scm.com/download/win).
- Tài khoản GitHub `hongson1001` và repo rỗng: `https://github.com/hongson1001/docs-bonbanh.com.git`.
- Xác thực GitHub: **HTTPS** (Personal Access Token khi hỏi mật khẩu) hoặc **SSH** (`git@github.com:hongson1001/docs-bonbanh.com.git`).

### Lệnh (chạy trong thư mục `docs-bonbanh.com`)

```powershell
cd C:\Users\Admin\docs-bonbanh.com

git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/hongson1001/docs-bonbanh.com.git
git push -u origin main
```

Nếu remote đã tồn tại (lỗi `remote origin already exists`):

```powershell
git remote set-url origin https://github.com/hongson1001/docs-bonbanh.com.git
git push -u origin main
```

### Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|-------------|------------|
| `Authentication failed` | Tạo PAT: GitHub → Settings → Developer settings → Personal access tokens; khi `git push` dùng PAT thay mật khẩu, hoặc chuyển sang SSH. |
| `rejected (fetch first)` | Repo trên GitHub đã có commit (ví dụ README tạo sẵn): `git pull origin main --rebase` rồi `git push -u origin main`. |
| `SSL certificate problem` | Kiểm tra mạng/proxy; tạm thời chỉ môi trường dev: `git config --global http.sslBackend schannel` (Windows). |

### Hiển thị docs dạng “website” trên GitHub

Vào repo trên GitHub → **Settings** → **Pages** → Source: **Deploy from a branch** → Branch `main`, folder `/ (root)` hoặc `/docs` nếu sau này bạn chuyển file vào thư mục `docs/`. URL dạng: `https://hongson1001.github.io/docs-bonbanh.com/` (tuỳ cấu hình tên repo/Pages).

---

## B. Deploy production: Angular + NestJS + PostgreSQL (VPS)

Phần này áp dụng khi bạn đã có **code ứng dụng** (không chỉ folder tài liệu này).

### B.1 Kiến trúc triển khai gợi ý

```
Internet → Nginx (HTTPS, gzip) →
  ├─ /           → static files (Angular `ng build --configuration production`)
  ├─ /api        → reverse proxy → NestJS (port nội bộ, ví dụ 3000)
  └─ (tuỳ chọn)  → upload trực tiếp lên S3/MinIO, CDN phục vụ ảnh
PostgreSQL     → cùng máy hoặc managed DB
Redis          → tuỳ chọn (cache, queue)
```

### B.2 Chuẩn bị VPS

- **OS:** Ubuntu 22.04/24.04 LTS (phổ biến).
- **Tối thiểu:** 2 vCPU, 4 GB RAM (MVP); tăng khi traffic/ảnh lớn.
- Mở firewall: `22` (SSH), `80`, `443`. **Không** public port PostgreSQL.

### B.3 Domain & SSL

1. Trỏ **A record** domain về IP VPS.
2. Cài chứng chỉ miễn phí:

```bash
sudo apt update && sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

### B.4 Biến môi trường (ví dụ)

**NestJS (`.env` trên server, không commit):**

- `NODE_ENV=production`
- `PORT=3000`
- `DATABASE_URL=postgresql://user:pass@127.0.0.1:5432/oto_db`
- `JWT_SECRET=...` (chuỗi dài, ngẫu nhiên)
- `JWT_REFRESH_SECRET=...`
- `CORS_ORIGIN=https://yourdomain.com`
- Lưu ảnh: `S3_ENDPOINT`, `S3_BUCKET`, `S3_ACCESS_KEY`, `S3_SECRET_KEY` (hoặc đường dẫn local + backup).

**Angular:** build-time `environment.prod.ts` — `apiUrl: 'https://yourdomain.com/api'`.

### B.5 Build Angular (trên CI hoặc máy build)

```bash
npm ci
ng build --configuration production
```

Copy thư mục output (thường `dist/<project>/browser`) lên VPS, ví dụ `/var/www/oto-frontend/`.

### B.6 Chạy NestJS

**Cách 1 — PM2:**

```bash
cd /var/www/oto-api
npm ci --omit=dev
npm run build
pm2 start dist/main.js --name oto-api
pm2 save && pm2 startup
```

**Cách 2 — Docker:** dùng image Node, `Dockerfile` multi-stage build; `docker compose` gồm `api`, `db`, `nginx`.

### B.7 Nginx (mẫu ý tưởng)

```nginx
server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    # ssl_certificate ... (certbot sẽ chèn)

    root /var/www/oto-frontend;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:3000/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        client_max_body_size 25M;
    }
}
```

Chỉnh `proxy_pass` khớp prefix API của NestJS (có hoặc không có `/api` global prefix).

### B.8 Cơ sở dữ liệu

- Tạo user/database riêng, quyền tối thiểu.
- Chạy migration (TypeORM/Prisma) trên server sau khi deploy:

```bash
npm run migration:run
```

- **Backup:** `pg_dump` hằng ngày (cron) hoặc snapshot nhà cung cấp VPS.

### B.9 Angular SSR (Universal) — nếu có

- Process Node SSR (thay vì chỉ static) hoặc dùng **prerender** cho một số route.
- Nginx proxy tới cổng SSR; tăng RAM và tunning cache.

### B.10 Webhook thanh toán (VNPay/MoMo)

- URL webhook phải **HTTPS** công khai.
- Whitelist IP nếu cổng cung cấp; verify chữ ký server-side; idempotent xử lý đơn.

### B.11 Checklist sau deploy

- [ ] HTTPS hoạt động, redirect HTTP → HTTPS.
- [ ] Đăng ký/đăng nhập, upload ảnh end-to-end.
- [ ] CORS chỉ đúng domain production.
- [ ] Log rotation (PM2/`journald`), disk không đầy vì log.
- [ ] Backup DB đã lên lịch; có bản restore thử nghiệm.

---

## Tài liệu liên quan trong repo

- [PLAN_OTO_MARKETPLACE_BONBANH_STYLE.md](./PLAN_OTO_MARKETPLACE_BONBANH_STYLE.md) — phạm vi chức năng & kiến trúc.

*Cập nhật: tháng 3/2026.*
