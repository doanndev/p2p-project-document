# Đặc tả API Authentication Module

## Base URL
```
https://dp7vlq3z-8000.asse.devtunnels.ms/api/v1
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
  "path": "/auth/login"
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
  "path": "/auth/login"
}
```

---

## 1. Đăng nhập

## 2. Google OAuth

### 2.1. Lấy Google OAuth URL

### GET `/auth/google/auth-url`

**Mô tả:** Tạo URL để đăng nhập Google OAuth

**Request:**
- Method: `GET`
- Headers: Không cần

**Response:**
```json
{
    "status": "success",
    "successType": "retrieved",
    "message": "Google OAuth URL generated successfully",
    "data": {
        "authUrl": "https://accounts.google.com/o/oauth2/v2/auth?access_type=offline&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.profile&state=login-google&prompt=consent&response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_REDIRECT_URI"
    },
    "timestamp": "2025-09-15T03:23:58.885Z"
}
```

**Error Response:**
```json
{
    "status": "error",
    "successType": "failed",
    "message": "Failed to generate Google OAuth URL",
    "error": {
        "type": "INTERNAL_SERVER_ERROR",
        "details": {
            "originalError": "Error message"
        }
    },
    "timestamp": "2025-09-15T03:23:58.885Z"
}
```

### 2.2. Google OAuth Callback

### GET `/auth/google/callback`

**Mô tả:** Xử lý callback từ Google OAuth sau khi user đăng nhập

**Request:**
- Method: `GET`
- Query Parameters:
  - `code` (string, required): Authorization code từ Google
  - `state` (string, required): State parameter để bảo mật

**Response:**
```json
{
    "status": "success",
    "successType": "authenticated",
    "message": "Google login successful",
    "data": {
        "user": {
            "uid": 123,
            "uname": "john_doe",
            "uemail": "john.doe@gmail.com",
            "ufulllname": "John Doe",
            "uavatar": "https://lh3.googleusercontent.com/...",
            "ustatus": "active"
        },
        "isNewUser": false
    },
    "httpStatus": 200,
    "timestamp": "2025-09-15T03:23:58.885Z"
}
```

**Error Response:**
```json
{
    "status": "error",
    "successType": "failed",
    "message": "Authorization code has expired or already been used",
    "error": {
        "type": "UNAUTHORIZED",
        "details": {
            "originalError": "invalid_grant",
            "suggestion": "Please try logging in again"
        }
    },
    "httpStatus": 401,
    "timestamp": "2025-09-15T03:23:58.885Z"
}
```

---

## 6. Telegram Authentication

### 6.1. Xác thực Telegram

Telegram Bot Login

**Mô tả:** Đăng nhập thông qua Telegram Bot. User cần gửi `/start` cho bot để nhận login URL.

### POST `/auth/telegram/verify`

**Mô tả:** Xác thực mã code từ Telegram

**Body Parameters:**
- `telegram_id`: ID Telegram của user
- `code`: Mã xác thực 6 số

**Response:**
```json
{
    "success": true,
    "message": "Telegram verification successful",
    "timestamp": "2025-09-15T08:31:00.355Z"
}
```

**Error Cases:**
- `400`: Thiếu telegram_id hoặc code
- `404`: User không tồn tại
- `410`: Mã code đã hết hạn

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


### Refresh Token
- **Thời gian hết hạn:** 7 ngày

### Cookie Settings
- **HttpOnly:** true
- **Secure:** true (production)
- **SameSite:** 'strict'

