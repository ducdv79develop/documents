🛡️ Hướng Dẫn Theo Dõi & Chặn DDoS Trên VPS
📌 1. Kiểm tra lượng kết nối đến server
🔹 Kiểm tra số lượng kết nối từ mỗi IP:
sh
Copy
Edit
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -20
➡️ Hiển thị top 20 IP có nhiều kết nối nhất.

🔹 Kiểm tra số lượng kết nối SYN (nghi vấn SYN Flood):
sh
Copy
Edit
netstat -ntu | grep SYN_RECV | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -10
➡️ Nếu có nhiều SYN_RECV, có thể là tấn công SYN Flood.

🔹 Kiểm tra cổng nào đang bị tấn công nhiều nhất:
sh
Copy
Edit
netstat -ant | awk '{print $4}' | cut -d: -f2 | sort | uniq -c | sort -nr | head -10
➡️ Nếu cổng 80 (HTTP) hoặc 443 (HTTPS) có quá nhiều kết nối, có thể là HTTP Flood.

📌 2. Theo dõi lưu lượng mạng theo thời gian thực
🔹 Dùng iftop để theo dõi băng thông:
sh
Copy
Edit
iftop -n -P
📌 Nếu chưa có iftop, cài đặt bằng:

sh
Copy
Edit
sudo apt install iftop  # Ubuntu/Debian
sudo yum install iftop  # CentOS/RHEL
➡️ Hiển thị IP nào đang chiếm nhiều băng thông nhất.

🔹 Dùng tcpdump để kiểm tra gói tin từ IP đáng ngờ:
sh
Copy
Edit
tcpdump -i eth0 src host 192.168.1.100
📌 Thay eth0 bằng tên interface mạng của bạn (ip a để kiểm tra).

📌 3. Chặn IP tấn công bằng Firewall
🔹 Chặn một IP cụ thể:
sh
Copy
Edit
sudo iptables -A INPUT -s 192.168.1.100 -j DROP
➡️ Chặn tất cả kết nối từ IP 192.168.1.100.

🔹 Tự động chặn IP có nhiều kết nối bất thường:
sh
Copy
Edit
sudo iptables -A INPUT -p tcp --syn -m connlimit --connlimit-above 100 -j DROP
➡️ Chặn IP có hơn 100 kết nối cùng lúc.

🔹 Chặn nhiều IP trong danh sách:
sh
Copy
Edit
for ip in $(cat ddos-ips.txt); do sudo iptables -A INPUT -s $ip -j DROP; done
➡️ Chặn nhiều IP từ file ddos-ips.txt.

🔹 Chặn toàn bộ TCP Flood:
sh
Copy
Edit
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 20/s --limit-burst 100 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
➡️ Giới hạn tối đa 20 request/giây trên cổng 80.

🔹 Chặn UDP Flood:
sh
Copy
Edit
sudo iptables -A INPUT -p udp -m limit --limit 10/s -j ACCEPT
sudo iptables -A INPUT -p udp -j DROP
➡️ Giới hạn UDP để tránh UDP Flood.

📌 4. Xóa luật chặn nếu cần
🔹 Xóa luật chặn một IP cụ thể:
sh
Copy
Edit
sudo iptables -D INPUT -s 192.168.1.100 -j DROP
🔹 Xóa tất cả luật chặn:
sh
Copy
Edit
sudo iptables -F
🔹 Kiểm tra danh sách IP bị chặn:
sh
Copy
Edit
sudo iptables -L -v -n
📌 5. Bảo vệ nâng cao
Dùng Cloudflare để bảo vệ website khỏi DDoS HTTP Flood.
Sử dụng AWS WAF để lọc request bất thường.
Giới hạn số lần đăng nhập sai trong ứng dụng của bạn.
📌 Nếu cần tự động hóa, bạn có thể viết script cronjob để kiểm tra và chặn IP theo thời gian thực.
