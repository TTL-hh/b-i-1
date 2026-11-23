# BÀI 1: Dựng hạ tầng Docker Compose

| Thông tin | Chi tiết |
|---|---|
| **Họ tên học sinh** | Trần Tùng Lâm |
| **MSSV** | K235480106039 |
| **Môn học** | Lập trình Web |
| **Môi trường thực hiện** | Windows + WSL Ubuntu |
| **Công nghệ sử dụng** | Docker, Docker Compose, Nginx, MariaDB, phpMyAdmin, Node-RED, Cloudflared |

---

## Cấu trúc thư mục

```text
bai1/
├── README.md
├── docker-compose.yml
├── .env.example
├── nginx/conf.d/
│   ├── site1.conf
│   └── site2.conf
├── sites/
│   ├── site1/index.html
│   └── site2/index.html
└── nodered-data/
```

---

## Yêu cầu

1. Giả lập Linux OS (Hyper-V, VirtualBox, VMware hoặc WSL).
2. Cài đặt Docker Compose trên OS đó.
3. Cài trên Docker Compose các dịch vụ: Nginx, Node-RED, MariaDB, phpMyAdmin, Cloudflared (cần domain).
4. Cấu hình Nginx chạy 2 website với 2 domain khác nhau.

## 1. Giả lập Linux bằng WSL

Mở PowerShell bằng quyền Administrator:

```powershell
wsl --install -d Ubuntu
```

Khởi động lại máy, mở Ubuntu và tạo tài khoản. Kiểm tra phiên bản WSL:

```powershell
wsl -l -v
```

> Kết quả cần thấy: distro `Ubuntu` chạy `VERSION 2`.

<!-- ![WSL Ubuntu](images/bai1-wsl.png) -->

## 2. Cài đặt Docker và Docker Compose

Trong terminal Ubuntu:

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Đăng xuất rồi đăng nhập lại Ubuntu, sau đó kiểm tra:

```bash
docker --version
docker compose version
```

<!-- ![Docker version](images/bai1-docker-version.png) -->

## 3. Chạy các dịch vụ bằng Docker Compose

File `docker-compose.yml` gồm 5 dịch vụ:

| Dịch vụ | Image | Cổng | Vai trò |
|---|---|---|---|
| nginx | `nginx:stable` | 80 | Web server, chạy 2 website |
| nodered | `nodered/node-red` | 1880 | Công cụ lập trình luồng, tạo API |
| mariadb | `mariadb:lts` | (nội bộ) | Cơ sở dữ liệu |
| phpmyadmin | `phpmyadmin` | 8080 | Giao diện quản trị MariaDB |
| cloudflared | `cloudflare/cloudflared` | (không mở) | Đưa website ra Internet qua Cloudflare Tunnel |

Các bước chạy:

```bash
cd bai1
cp .env.example .env
nano .env                                  # đổi DB_ROOT_PASSWORD
sudo chown -R 1000:1000 nodered-data
docker compose up -d
docker compose ps
```

Kiểm tra truy cập:

- Node-RED: http://localhost:1880
- phpMyAdmin: http://localhost:8080 (user `root`, mật khẩu theo file `.env`)

<!-- ![docker compose ps](images/bai1-compose-ps.png) -->

### Cloudflared (Cloudflare Tunnel)

Cloudflared cần domain đã quản lý trên Cloudflare nên được đặt trong profile `tunnel`, chỉ chạy khi bật riêng.

1. Vào Cloudflare **Zero Trust → Networks → Tunnels → Create a tunnel** (loại Cloudflared).
2. Sao chép token, dán vào biến `CF_TUNNEL_TOKEN` trong file `.env`.
3. Thêm 2 Public Hostname, cùng trỏ về service `HTTP` → `nginx:80`:
   - `site1.<domain-cua-em>` 
   - `site2.<domain-cua-em>`
4. Chạy tunnel:

```bash
docker compose --profile tunnel up -d
```

<!-- ![Cloudflare Tunnel](images/bai1-cloudflare-tunnel.png) -->

## 4. Cấu hình Nginx chạy 2 website với 2 domain

Nginx phân biệt website dựa vào `server_name` (virtual host). Mỗi website có một file cấu hình riêng trong `nginx/conf.d/`:

```nginx
# site1.conf
server {
    listen 80;
    server_name site1.example.com site1.local;
    root /usr/share/nginx/html/site1;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

`site2.conf` có cấu trúc tương tự, chỉ khác `server_name` và `root` (`site2`).

**Cách kiểm tra khi chưa có domain thật:** thêm dòng sau vào file hosts của Windows (`C:\Windows\System32\drivers\etc\hosts`, mở Notepad bằng quyền Administrator):

```text
127.0.0.1 site1.local site2.local
```

Mở trình duyệt:

- http://site1.local → hiển thị **Website 1**
- http://site2.local → hiển thị **Website 2**

Khi dùng domain thật: sửa `server_name` cho đúng domain rồi chạy `docker compose restart nginx`.

<!-- ![Website 1](images/bai1-site1.png) -->
<!-- ![Website 2](images/bai1-site2.png) -->

## Kết quả Bài 1

- [x] Chạy được Linux (WSL Ubuntu)
- [x] Cài được Docker và Docker Compose
- [x] Chạy được Nginx, Node-RED, MariaDB, phpMyAdmin, Cloudflared bằng Docker Compose
- [x] Nginx phục vụ 2 website với 2 domain khác nhau

---

## Lệnh hữu ích

```bash
docker compose ps                 # xem trạng thái các dịch vụ
docker compose logs -f nginx      # xem log nginx
docker compose restart nginx      # nạp lại cấu hình nginx
docker compose down               # dừng và xóa container
```
