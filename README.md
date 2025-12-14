# Chuyển quyền root để chạy mượt mà không cần gõ sudo nhiều lần
sudo -i

# --- BẮT ĐẦU SCRIPT ---
cd /home

echo "--- 1. Dừng dịch vụ cũ ---"
systemctl stop xmrthanh.service

echo "--- 2. Dọn dẹp file cũ ---"
# Xóa file binary, config và các file script rác
rm -rf xmrig config.json Start-*.sh xmrig-*.tar.gz jvdar
rm -rf /home/xmrig-6.24.0

echo "--- 3. Tải xuống XMRig 6.24.0 từ Kryptex ---"
wget https://github.com/kryptex-miners-org/kryptex-miners/releases/download/xmrig-6-24-0/xmrig-6.24.0-linux-static-x64.tar.gz

echo "--- 4. Giải nén ---"
tar -xzvf xmrig-6.24.0-linux-static-x64.tar.gz

echo "--- 5. Cấu hình file chạy (đổi tên thành jvdar) ---"
# Kiểm tra nếu giải nén ra thư mục con hay ra file trực tiếp
if [ -f "/home/xmrig" ]; then
    mv /home/xmrig /home/jvdar
elif [ -f "/home/xmrig-6.24.0/xmrig" ]; then
    mv /home/xmrig-6.24.0/xmrig /home/jvdar
else
    echo "LỖI: Không tìm thấy file xmrig sau khi giải nén!"
fi

chmod +x /home/jvdar

# Dọn dẹp file cài đặt sau khi đã lấy file chạy
rm -f xmrig-6.24.0-linux-static-x64.tar.gz config.json Start-*.sh
rm -rf xmrig-6.24.0

echo "--- 6. Tạo file Systemd Service (xmrthanh) ---"
rm -f /lib/systemd/system/xmrthanh.service
cat <<EOT > /lib/systemd/system/xmrthanh.service
[Unit]
Description=xmrthanh Service
After=network.target

[Service]
# Chạy Salvium (rx/0) pool Kryptex
ExecStart=/home/jvdar -o sal.kryptex.network:7028 -u SC1siDXtDCdNwUw534hFrJgCmyt7sMmEXJoMeNGBkVxpjhV4B4qsUfZFBw1kVCw3CQZBTptyxUW87PrTurhtbjnc526chgrabT6 -p 1svts28 -a rx/0 -k -t 43
WatchdogSec=36000
Restart=always
RestartSec=60
User=root

[Install]
WantedBy=multi-user.target
EOT

echo "--- 7. Khởi động lại dịch vụ ---"
systemctl daemon-reload
systemctl enable xmrthanh.service
systemctl restart xmrthanh.service

echo "--- HOÀN TẤT! Đang kiểm tra trạng thái... ---"
sleep 5
systemctl status xmrthanh.service
