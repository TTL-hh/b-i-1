# BÀI 1: Dựng hạ tầng

## 1. Giả lập Linux + 2. Cài Docker Compose
WSL2 (PowerShell admin): `wsl --install -d Ubuntu`, khởi động lại, mở Ubuntu rồi:
    curl -fsSL https://get.docker.com | sudo sh
    sudo usermod -aG docker $USER   # đăng xuất / đăng nhập lại
    docker compose version
(Hoặc dùng Hyper-V / VirtualBox / VMware cài Ubuntu Server, rồi cài Docker như trên.)

## 3. Chạy nginx, nodered, mariadb, phpmyadmin, cloudflared
    nano .env                       # đổi mật khẩu DB
    sudo chown -R 1000:1000 nodered-data
    docker compose up -d
    docker compose ps
- Node-RED:   http://localhost:1880
- phpMyAdmin: http://localhost:8080 (user root, mật khẩu trong .env)

Cloudflared (cần domain): tạo tunnel ở Cloudflare Zero Trust → Networks → Tunnels,
dán token vào CF_TUNNEL_TOKEN trong .env, Public Hostname trỏ về http://nginx:80, rồi:
    docker compose --profile tunnel up -d

## 4. Nginx chạy 2 website với 2 domain
Hai file nginx/conf.d/site1.conf và site2.conf, phân biệt bằng server_name.
Test cục bộ: thêm vào file hosts `127.0.0.1 site1.local site2.local`
(Windows: C:\Windows\System32\drivers\etc\hosts) rồi mở http://site1.local và http://site2.local.
Dùng domain thật: sửa server_name rồi `docker compose restart nginx`.
