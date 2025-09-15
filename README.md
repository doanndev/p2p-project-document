# Đặc tả API Authentication Module

## Base URL
```
https://36e0a8b6e38d.ngrok-free.app/api/v1
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
    "successType": "retrieved",
    "message": "Google OAuth URL generated successfully",
    "data": {
        "authUrl": "https://accounts.google.com/o/oauth2/v2/auth?access_type=offline&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.profile&state=login-email&prompt=consent&response_type=code&client_id=214941487373-5u9pre3lr06nndoal6tfiphp3tb08pr6.apps.googleusercontent.com&redirect_uri=https%3A%2F%2F36e0a8b6e38d.ngrok-free.app%2Fapi%2Fv1%2Fauth%2Fgoogle%2Fcallback"
    },
    "httpStatus": 200,
    "timestamp": "2025-09-15T03:23:58.885Z"
}
```

### 5.3. Google Callback

### GET `/auth/google/callback`

**Mô tả:** Callback URL từ Google OAuth

**Query Parameters:**
- `code`: Authorization code từ Google
- `state`: State parameter

**Response:**
```json
{
  "status": "success",
  "successType": "login_success",
  "message": "Login successful",
  "data": {
    "status": true,
    "message": "Login successful",
    "user": {
      "uid": 71,
      "email": "doann.dev@gmail.com",
      "fullName": "Cornelius Waters",
      "isActive": true
    },
    "wallet": {
      "umw_id": 10,
      "address": "CZpGjxh8tkjuCCMiUeHchawmufBUivK33FUeZ5Wrsn7u"
    },
    "isNewUser": false
  },
  "httpStatus": 200,
  "timestamp": "2025-09-15T03:19:41.333Z"
}
```

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
