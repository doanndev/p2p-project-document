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
  "usex": "other",
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
