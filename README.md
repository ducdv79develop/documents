# 🛡️ Hướng Dẫn Theo Dõi & Chặn DDoS Trên VPS

## 📌 1. Kiểm tra lượng kết nối đến server

### 🔹 Kiểm tra số lượng kết nối từ mỗi IP:
```sh
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -20
