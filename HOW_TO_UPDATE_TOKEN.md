# Hướng dẫn cập nhật Firebase AppCheck Token

## Vấn đề
Lỗi "API not initialized. Check server logs." xảy ra khi Firebase AppCheck token hết hạn.

Token này thường có thời hạn ngắn (vài giờ đến vài ngày) và cần được cập nhật định kỳ.

## Cách lấy token mới

### Phương pháp 1: Dùng Proxyman/Charles Proxy (macOS)

1. **Cài đặt Proxyman** (hoặc Charles Proxy)
   - Download từ: https://proxyman.io/
   - Cài đặt SSL certificate để intercept HTTPS

2. **Cấu hình iPhone**
   - Kết nối iPhone và Mac cùng mạng WiFi
   - Trong iPhone Settings > Wi-Fi > Configure Proxy > Manual
   - Nhập IP của Mac và port (mặc định 9090)
   - Cài đặt SSL certificate trên iPhone (Settings > General > VPN & Device Management)

3. **Chạy Locket app**
   - Mở app Locket trên iPhone
   - Đăng nhập hoặc thực hiện bất kỳ action nào

4. **Lấy token từ request**
   - Trong Proxyman, tìm request đến `api.locketcamera.com` hoặc `www.googleapis.com`
   - Xem header `X-Firebase-AppCheck`
   - Copy toàn bộ giá trị token (rất dài, khoảng 1000+ ký tự)

### Phương pháp 2: Dùng HTTP Catcher (iOS - không cần máy tính)

1. **Cài HTTP Catcher** từ App Store
2. **Bật VPN** trong app
3. **Mở Locket app** và thực hiện action
4. **Xem captured requests** trong HTTP Catcher
5. **Tìm header** `X-Firebase-AppCheck`
6. **Copy token**

### Phương pháp 3: Dùng mitmproxy (Linux/Windows/macOS)

```bash
# Cài đặt
pip install mitmproxy

# Chạy
mitmweb

# Cấu hình proxy trên iPhone (IP:8080)
# Cài cert từ mitm.it
# Mở Locket app
# Xem requests trong web interface (http://127.0.0.1:8081)
```

## Cách cập nhật token

### Bước 1: Copy token vừa lấy được

Token sẽ có dạng:
```
eyJraWQiOiJNbjVDS1EiLCJ0eXAiOiJKV1QiLCJhbGc....[rất dài]....X6Xx
```

### Bước 2: Mở file `.env`

```bash
nano .env
# hoặc
code .env
```

### Bước 3: Thay thế token cũ

Tìm dòng:
```
FIREBASE_APPCHECK_TOKEN=eyJraWQ...
```

Thay bằng token mới:
```
FIREBASE_APPCHECK_TOKEN=<token_moi_cua_ban>
```

### Bước 4: Restart server

```bash
# Dừng server (Ctrl+C)
# Chạy lại
python app.py
```

## Kiểm tra

Sau khi restart:
- Mở browser đến http://localhost:5000
- Thử unlock Locket Gold cho một username
- Nếu vẫn lỗi "API not initialized", kiểm tra terminal logs để xem lỗi cụ thể

## Lưu ý

- Token thường hết hạn sau **1-7 ngày**
- Nên cập nhật token mỗi khi gặp lỗi authentication
- **KHÔNG** commit token vào Git (đã có trong .gitignore)
- Token này chỉ cho phép truy cập API Locket, không phải thông tin nhạy cảm khác

## Token hiện tại đã hết hạn

Token trong file hiện tại có `exp: 1722240670` (timestamp) = **29/07/2024 10:24 AM UTC**

Bạn **BẮT BUỘC** phải lấy token mới để hệ thống hoạt động.

## Debug

Nếu không chắc token có đúng không:

```python
import jwt
import json

token = "paste_token_here"
try:
    # Decode (không verify signature)
    decoded = jwt.decode(token, options={"verify_signature": False})
    print(json.dumps(decoded, indent=2))
    print(f"\nExpires at: {decoded['exp']}")
except Exception as e:
    print(f"Invalid token: {e}")
```

Kiểm tra `exp` (expiration) phải **lớn hơn** thời gian hiện tại (timestamp hiện tại: ~1770000000+)
