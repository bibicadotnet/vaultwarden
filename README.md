# Vaultwarden with Cloudflare Tunnels for Oracle Away FREE
## 1. Cài đặt VPS
Tạo 1 VPS Oracle tại US, gói miễn phí cấu hình thấp nhất VM.Standard.E2.1.Micro chạy Ubuntu 22.04, 1 GB RAM

Login với quyền root

Cập nhập OS
```
sudo apt update && sudo apt upgrade -y && sudo reboot
```
Bật BBR, tắt firewall, tạo 4G RAM ảo, cài đặt docker, docker-compose, Rclone và bypass Oracle
```
sudo wget https://go.bibica.net/oracle-docker -O oracle-docker.sh && sudo chmod +x oracle-docker.sh && sudo ./oracle-docker.sh
```
## 2. Cài đặt Vaultwarden
Tạo thư mục lưu trữ và tạo file cấu hình
```
mkdir ./vaultwarden
cd ./vaultwarden
sudo wget https://raw.githubusercontent.com/bibicadotnet/vaultwarden/main/docker-compose.yml
```
### Thêm push Bitwarden
Để lấy Id và Key, truy cập vào [Bitwarden](https://bitwarden.com/host/) điền email, data region chọn US, nó sẽ ra 2 giá trị `INSTALLATION ID` và `INSTALLATION KEY`

Thay thế vào bên trong file `docker-compose.yml`

### Cập nhập email thông báo
Nên dùng 1 dịch vụ SMTP nào đó như [smtp2go](https://www.smtp2go.com/) để gửi email thông báo, giúp kiểm soát, quản lý việc login tài khoản tốt hơn

Chỉnh sửa lại các giá trị bên trong `docker-compose.yml` cho phù hợp rồi khởi chạy docker
```
docker-compose up -d --build --remove-orphans --force-recreate
```
### Cấu Hình Cloudflare Tunnels
Tạo tài khoản [Cloudflare Zero Trust](https://one.dash.cloudflare.com/) -> Access -> Tunnel -> Create a tunnel -> đặt 1 tên cho dễ nhớ (Vaultwarden) -> Install and run a connector

![alt text](https://raw.githubusercontent.com/bibicadotnet/vaultwarden/main/2024-01-21_03-39-50.png)

Tạo Public hostnames `sub.domain.com` tới `localhost:1254`

Lúc này bạn có thể truy cập vào `sub.domain.com` để tạo tài khoản và login vào sử dụng
## 3. Backup và restore
### Backup
Đơn giản, hiệu quả và miễn phí thì mình vẫn ưu tiên sử dụng [Rclone để tạo backup trên Google Drive](https://bibica.net/cau-hinh-backup-webinoly-rclone-cloudflare-r2-va-google-drive/)

Tạo 1 file bash để backup tương tự như sau
```
nano /root/vaultwarden/backup.sh
```
Bên trong điền vào
```
tar -cvf vaultwarden.tar /root/vaultwarden
rclone sync vaultwarden.tar google-drive:_vaultwarden --progress
rm vaultwarden.tar
```
Ctrl+O -> Enter -> Ctrl+X để save và exit

Phân quyền
```
chmod +x /root/vaultwarden/backup.sh
```
Tạo cron để tự chạy 1 phút 1 lần
```
crontab -l > vaultwarden_cron
echo "* * * * * /root/vaultwarden/backup.sh" >> vaultwarden_cron
crontab vaultwarden_cron
```
### Restore
Muốn restore trên một VPS mới hoàn toàn, thì các bước như sau

#### Cập nhập OS và cài đặt docker tương tự như ban đầu

Copy cấu hình cũ Rclone

Thường mình hay chép file cấu hình này ở 1 URL nào đó, sau đó wget thẳng vào VPS mới, hoặc cẩn thận hơn, có thể down file cấu hình Rclone về máy, sau đó load trực tiếp lên VPS mới, theo đường dẫn `/root/.config/rclone/rclone.conf`

Copy bản backup từ Google Drive xuống
```
rclone copyto google-drive:_vaultwarden/vaultwarden.tar vaultwarden.tar --progress
```
Giải nén file backup
```
tar -xvf vaultwarden.tar
```
Điều chỉnh lại đường dẫn cho phù hợp
```
mv /root/root/* /root
rm -r /root/root
```
Cài đặt lại Vaultwarden
```
cd vaultwarden
docker-compose up -d --build --remove-orphans --force-recreate
```
Cấu hình lại cron
```
chmod +x /root/vaultwarden/backup.sh
crontab -l > vaultwarden_cron
echo "* * * * * /root/vaultwarden/backup.sh" >> vaultwarden_cron
crontab vaultwarden_cron
```
### Cấu hình lại Cloudflare Tunnels
1. Xóa DNS records cũ sub.domain.com trên Cloudflare
2. Xóa Tunnels cũ sub.domain.com trên Cloudflare
3. Tạo Tunnels mới trên VPS mới


































