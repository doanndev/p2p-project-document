# Đặc tả API Authentication Module

## Base URL
```
https://44ef1db0ff39.ngrok-free.app/api/v1
```

## Response Format
Tất cả API đều sử dụng format response thống nhất:

### Success Response
```json
{
  "status": "success",
  "successType": "LOGIN_SUCCESS",
  "message": "Login successful",
  "data": { ... },
  "httpStatus": 200,
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/auth/login",
  "requestId": "uuid"
}
```

### Error Response
```json
{
  "success": false,
  "message": "Error message",
  "error": {
    "type": "UNAUTHORIZED",
    "details": { ... },
    "validationErrors": [ ... ]
  },
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/auth/login",
  "requestId": "uuid"
}
```

---

## 1. Đăng ký tài khoản

### POST `/auth/register`

**Mô tả:** Đăng ký tài khoản mới với email và password

**Request Body:**
```json
{
  "uname": "john_doe",
  "uemail": "john@example.com",
  "uphone": "+84123456789",
  "upassword": "password123",
  "ufullname": "John Doe",
  "ubirthday": "1990-01-01",
  "usex": "male",
  "uref": "REF123"
}
```

**Validation Rules:**
- `uname`: 3-16 ký tự
- `uemail`: Email hợp lệ
- `uphone`: 10-15 số, format quốc tế
- `upassword`: 6-32 ký tự
- `ufullname`: 2-32 ký tự
- `ubirthday`: Date string (optional)
- `usex`: enum (male, female, other)
- `uref`: Mã giới thiệu (optional)

**Response:**
```json
{
  "status": "success",
  "successType": "CREATED",
  "message": "A verification code has been sent to your email.",
  "data": "john@example.com",
  "httpStatus": 201
}
```

**Error Cases:**
- `400`: Validation error
- `409`: Email/username đã tồn tại
- `500`: Lỗi server

---

## 2. Xác thực email

### POST `/auth/verify`

**Mô tả:** Xác thực mã code được gửi qua email

**Request Body:**
```json
{
  "verificationCode": "123456",
  "codeType": "ACTIVE_EMAIL",
  "email": "john@example.com"
}
```

**Validation Rules:**
- `verificationCode`: 6 số
- `codeType`: enum (ACTIVE_EMAIL, TELE_LOGIN, RESET_PASSWORD, CHANGE_BANK, WITHDRAW)
- `email`: Email hợp lệ (hoặc username, phone, telegram)

**Response:**
```json
{
  "status": "success",
  "successType": "VERIFIED",
  "message": "Code verified successfully",
  "data": {
    "success": true,
    "message": "email-verification code verified successfully",
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  },
  "httpStatus": 200
}
```

**Error Cases:**
- `400`: Mã code không hợp lệ
- `404`: User không tồn tại
- `410`: Mã code đã hết hạn

---

## 3. Đăng nhập

### POST `/auth/login`

**Mô tả:** Đăng nhập bằng email/username và password

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

**Hoặc:**
```json
{
  "username": "john_doe",
  "password": "password123"
}
```

**Validation Rules:**
- `email` hoặc `username`: Bắt buộc một trong hai
- `password`: 6-32 ký tự

**Response:**
```json
{
  "status": "success",
  "successType": "LOGIN_SUCCESS",
  "message": "Login successful",
  "data": {
    "user": {
      "uid": 1,
      "uname": "john_doe",
      "uemail": "john@example.com",
      "ufullname": "John Doe",
      "u_active_email": true,
      "u_active_ggauth": false
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  },
  "httpStatus": 200
}
```

**Error Cases:**
- `400`: Thiếu thông tin đăng nhập
- `401`: Thông tin đăng nhập không đúng
- `403`: Tài khoản chưa xác thực email
- `423`: Tài khoản bị khóa

---

## 4. Đăng xuất

### GET `/auth/logout`

**Mô tả:** Đăng xuất và vô hiệu hóa token

**Headers:**
```
Cookie: accessToken=xxx; refreshToken=xxx
```

**Response:**
```json
{
  "status": "success",
  "successType": "LOGOUT_SUCCESS",
  "message": "Logout successful",
  "data": {
    "success": true,
    "message": "Logout successful",
    "tokens": {
      "accessToken": "expired_token",
      "refreshToken": "expired_token"
    }
  },
  "httpStatus": 200
}
```

**Error Cases:**
- `401`: Không có token trong cookie

---

## 5. Google OAuth

### 5.1. Lấy Google OAuth URL

### POST `/auth/google/auth-url`

**Mô tả:** Tạo URL để đăng nhập Google

**Request Body:**
```json
{
  "state": "login-email"
}
```

**Response:**
```json
{
  "status": "success",
  "successType": "RETRIEVED",
  "message": "Google OAuth URL generated successfully",
  "data": {
    "authUrl": "https://accounts.google.com/oauth/authorize?..."
  },
  "httpStatus": 200
}
```

### 5.2. Đăng nhập Google

### POST `/auth/google/login`

**Mô tả:** Đăng nhập bằng Google authorization code

**Request Body:**
```json
{
  "code": "4/0AX4XfWh...",
  "ref_code": "REF123"
}
```

**Response:**
```json
{
  "status": "success",
  "successType": "LOGIN_SUCCESS",
  "message": "Login successful",
  "data": {
    "status": true,
    "message": "Login successful",
    "user": {
      "uid": 1,
      "email": "john@gmail.com",
      "fullName": "John Doe",
      "isActive": true
    },
    "wallet": {
      "umw_id": 1,
      "address": "0x1234..."
    },
    "isNewUser": false,
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  },
  "httpStatus": 200
}
```

### 5.3. Google Callback

### GET `/auth/google/callback`

**Mô tả:** Callback URL từ Google OAuth

**Query Parameters:**
- `code`: Authorization code từ Google
- `state`: State parameter

**Response:** Tương tự như POST `/auth/google/login`

---

## 6. Telegram Authentication

### 6.1. Xác thực Telegram

### GET `/auth/telegram/verify`

**Mô tả:** Xác thực mã code từ Telegram

**Query Parameters:**
- `telegram_id`: ID Telegram của user
- `code`: Mã xác thực 6 số

**Response:**
```json
{
  "status": "success",
  "successType": "LOGIN_SUCCESS",
  "message": "Telegram verification successful",
  "data": {
    "uid": 1,
    "uname": "john_doe",
    "uemail": "john@example.com",
    "ufullname": "John Doe",
    "u_active_email": true,
    "u_active_ggauth": true
  },
  "httpStatus": 200
}
```

**Error Cases:**
- `400`: Thiếu telegram_id hoặc code
- `404`: User không tồn tại
- `410`: Mã code đã hết hạn

---

## 7. Các loại Code và thời gian hết hạn

| Code Type | Mô tả | Thời gian hết hạn | Kênh gửi |
|-----------|-------|-------------------|----------|
| `ACTIVE_EMAIL` | Xác thực email | 15 phút | Email |
| `TELE_LOGIN` | Đăng nhập Telegram | 10 phút | Telegram |
| `RESET_PASSWORD` | Đặt lại mật khẩu | 30 phút | Email |
| `CHANGE_BANK` | Thay đổi ngân hàng | 5 phút | SMS |
| `WITHDRAW` | Rút tiền | 5 phút | SMS |

---

## 8. JWT Token

### Access Token
- **Thời gian hết hạn:** 15 phút
- **Payload:** `{ sub: userId, username, email, type: 'access' }`
- **Secret:** `JWT_SECRET`

### Refresh Token
- **Thời gian hết hạn:** 7 ngày
- **Payload:** `{ sub: userId, type: 'refresh' }`
- **Secret:** `JWT_REFRESH_SECRET`

### Cookie Settings
- **HttpOnly:** true
- **Secure:** true (production)
- **SameSite:** 'strict'

---

## 9. Error Types

| Error Type | HTTP Status | Mô tả |
|------------|-------------|-------|
| `BAD_REQUEST` | 400 | Dữ liệu đầu vào không hợp lệ |
| `UNAUTHORIZED` | 401 | Chưa đăng nhập hoặc token không hợp lệ |
| `FORBIDDEN` | 403 | Không có quyền truy cập |
| `NOT_FOUND` | 404 | Không tìm thấy tài nguyên |
| `CONFLICT` | 409 | Xung đột dữ liệu (email đã tồn tại) |
| `VALIDATION_ERROR` | 422 | Lỗi validation |
| `ACCOUNT_SUSPENDED` | 423 | Tài khoản bị khóa |
| `TWO_FACTOR_REQUIRED` | 424 | Cần xác thực 2 bước |
| `VERIFICATION_FAILED` | 410 | Mã xác thực không hợp lệ |
| `INVALID_CREDENTIALS` | 401 | Thông tin đăng nhập không đúng |
| `INTERNAL_SERVER_ERROR` | 500 | Lỗi server |

---

## 10. Testing Examples

### Test Case 1: Đăng ký và xác thực email
```bash
# 1. Đăng ký
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "uname": "testuser",
    "uemail": "test@example.com",
    "uphone": "+84123456789",
    "upassword": "password123",
    "ufullname": "Test User",
    "usex": "male"
  }'

# 2. Xác thực email (sử dụng code từ email)
curl -X POST http://localhost:3000/auth/verify \
  -H "Content-Type: application/json" \
  -d '{
    "verificationCode": "123456",
    "codeType": "ACTIVE_EMAIL",
    "email": "test@example.com"
  }'
```

### Test Case 2: Đăng nhập
```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

### Test Case 3: Google OAuth
```bash
# 1. Lấy auth URL
curl -X POST http://localhost:3000/auth/google/auth-url \
  -H "Content-Type: application/json" \
  -d '{"state": "login-email"}'

# 2. Đăng nhập với code
curl -X POST http://localhost:3000/auth/google/login \
  -H "Content-Type: application/json" \
  -d '{"code": "4/0AX4XfWh..."}'
```

### Test Case 4: Telegram
```bash
# Xác thực Telegram
curl -X GET "http://localhost:3000/auth/telegram/verify?telegram_id=123456789&code=123456"
```

---

## 11. Frontend Integration

### Cookie Management
```javascript
// Sau khi đăng nhập thành công, tokens được set vào cookies
// Frontend cần gửi cookies trong mọi request

// Axios example
axios.defaults.withCredentials = true;

// Fetch example
fetch('/api/protected-route', {
  credentials: 'include'
});
```

### Error Handling
```javascript
// Xử lý error response
if (response.data.error) {
  switch (response.data.error.type) {
    case 'TWO_FACTOR_REQUIRED':
      // Redirect to verification page
      break;
    case 'INVALID_CREDENTIALS':
      // Show login error
      break;
    case 'ACCOUNT_SUSPENDED':
      // Show account suspended message
      break;
  }
}
```

---

## 12. Security Notes

1. **Password:** Được hash bằng bcrypt với salt rounds = 10
2. **JWT:** Sử dụng secret keys riêng biệt cho access và refresh token
3. **Cookies:** HttpOnly, Secure, SameSite strict
4. **Rate Limiting:** Nên implement rate limiting cho các API đăng nhập
5. **CORS:** Cấu hình CORS phù hợp cho production
6. **HTTPS:** Bắt buộc sử dụng HTTPS trong production

---

## 13. Database Schema

### User Table
```sql
CREATE TABLE users (
  uid SERIAL PRIMARY KEY,
  uname VARCHAR(16) UNIQUE NOT NULL,
  uemail VARCHAR(255) UNIQUE NOT NULL,
  uphone VARCHAR(15),
  upassword VARCHAR(255) NOT NULL,
  ufulllname VARCHAR(32) NOT NULL,
  ubirthday DATE,
  usex VARCHAR(10),
  uavatar VARCHAR(255),
  utelegram VARCHAR(50),
  uggauth VARCHAR(255),
  u_active_email BOOLEAN DEFAULT FALSE,
  u_active_ggauth BOOLEAN DEFAULT FALSE,
  uref VARCHAR(32),
  uverify BOOLEAN DEFAULT FALSE,
  ustatus VARCHAR(20) DEFAULT 'active',
  created_at TIMESTAMP DEFAULT NOW()
);
```

### User Code Table
```sql
CREATE TABLE user_codes (
  uc_id SERIAL PRIMARY KEY,
  uc_value VARCHAR(6) NOT NULL,
  uc_type VARCHAR(20) NOT NULL,
  uc_place VARCHAR(20) NOT NULL,
  uc_code_time TIMESTAMP NOT NULL,
  uc_life BOOLEAN DEFAULT TRUE,
  uc_user_id INTEGER REFERENCES users(uid)
);
```

### User Main Wallet Table
```sql
CREATE TABLE user_main_wallets (
  umw_id SERIAL PRIMARY KEY,
  umw_user_id INTEGER UNIQUE REFERENCES users(uid),
  umw_path_hd_wallet INTEGER NOT NULL,
  umw_address VARCHAR(255) NOT NULL,
  umw_create_at TIMESTAMP DEFAULT NOW()
);
```
