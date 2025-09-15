# Đặc tả API Authentication Module

## Base URL
```
https://dp7vlq3z-8000.asse.devtunnels.ms/api/v1
```

## 1. Đăng nhập Google

### Endpoint
```
POST /auth/login/google
```

### Request Body
```json
{
  "code": "4%2F0AVGzR1CoEBpQO6uFh5jtD-fOwovJk_li9f6F7jT6I6pA79mnyq5Rf7aXwnFWAhDmbuexKw",
  "path": "login-google",
  "ref_code": "optional_referral_code"
}
```

| Field | Type | Required | Mô tả |
|-------|------|----------|-------|
| `code` | string | Yes | Mã authorization từ Google OAuth |
| `path` | string | No | Đường dẫn redirect URI |
| `ref_code` | string | No | Mã giới thiệu |

### Response Success (200)
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "uname": "john_doe",
      "uemail": "john@example.com",
      "ufulllname": "John Doe",
      "uavatar": "https://lh3.googleusercontent.com/...",
      "ubirthday": "1990-01-01",
      "usex": "male"
    },
    "wallet": {
      "umw_id": 1,
      "address": "0x1234abcd...",
      "created_at": "2024-01-01T00:00:00.000Z"
    },
    "isNewUser": true
  },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Response Error (400/500)
```json
{
  "success": false,
  "message": "Google login failed",
  "error": {
    "type": "BAD_REQUEST",
    "details": {
      "originalError": "Invalid state parameter"
    }
  }
}
```

**Cookies được set:** `accessToken`, `refreshToken`

---

## 2. Xác thực Telegram

### Endpoint
```
POST /auth/telegram/verify
```

### Request Body
```json
{
  "telegram_id": "123456789",
  "code": "123456"
}
```

| Field | Type | Required | Mô tả |
|-------|------|----------|-------|
| `telegram_id` | string | Yes | ID người dùng Telegram |
| `code` | string | Yes | Mã xác thực 6 số |

### Response Success (200)
```json
{
  "success": true,
  "message": "Telegram verification successful",
  "data": {
    "user": {
      "uname": "john_doe",
      "uemail": "john@example.com",
      "ufulllname": "John Doe",
      "uavatar": "https://t.me/i/userpic/...",
      "ubirthday": "1990-01-01",
      "usex": "male"
    },
    "wallet": {
      "umw_id": 1,
      "address": "0x1234abcd...",
      "created_at": "2024-01-01T00:00:00.000Z"
    }
  },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Response Error (400)
```json
{
  "success": false,
  "message": "Telegram ID and verification code are required",
  "error": {
    "type": "BAD_REQUEST",
    "details": {
      "provided": {
        "telegramId": "123456789",
        "code": "123456"
      }
    }
  }
}
```

**Cookies được set:** `accessToken`, `refreshToken`

---

## 3. Lấy thông tin profile

### Endpoint
```
GET /auth/me
```

### Request Headers
```
Cookie: accessToken=your_access_token; refreshToken=your_refresh_token
```

### Response Success (200)
```json
{
  "success": true,
  "message": "Profile retrieved successfully",
  "data": {
    "user": {
      "uname": "john_doe",
      "uemail": "john@example.com",
      "ufulllname": "John Doe",
      "uavatar": "https://example.com/avatar.jpg",
      "ubirthday": "1990-01-01",
      "usex": "male"
    },
    "mainWallet": {
      "umw_id": 1,
      "address": "0x1234abcd...",
      "created_at": "2024-01-01T00:00:00.000Z"
    },
    "importWallets": [
      {
        "uic_id": 1,
        "uic_name": "My Import Wallet",
        "address": "0x5678efgh...",
        "created_at": "2024-01-01T00:00:00.000Z"
      }
    ]
  },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Response Error (401)
```json
{
  "success": false,
  "message": "Both access token and refresh token are invalid",
  "error": {
    "type": "UNAUTHORIZED",
    "details": {
      "reason": "Session expired, please login again"
    }
  }
}
```

**Ghi chú:** API tự động refresh token nếu access token hết hạn

---

## 4. Đăng xuất

### Endpoint
```
GET /auth/logout
```

### Request Headers
```
Cookie: accessToken=your_access_token; refreshToken=your_refresh_token
```

### Response Success (200)
```json
{
  "success": true,
  "message": "Logout successful",
  "data": {
    "message": "User logged out successfully"
  },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Response Error (401)
```json
{
  "success": false,
  "message": "No access token found in cookies",
  "error": {
    "type": "UNAUTHORIZED",
    "details": {
      "reason": "User not logged in"
    }
  }
}
```

**Cookies bị xóa:** `accessToken`, `refreshToken`

---

## Các loại lỗi thường gặp

- `UNAUTHORIZED`: Token không hợp lệ hoặc thiếu
- `BAD_REQUEST`: Tham số request không đúng
- `USER_NOT_FOUND`: Không tìm thấy user
- `VERIFICATION_FAILED`: Xác thực mã thất bại
- `INTERNAL_SERVER_ERROR`: Lỗi server

## 8. JWT Token

### Access Token
- **Thời gian hết hạn:** 15 phút


### Refresh Token
- **Thời gian hết hạn:** 7 ngày

### Cookie Settings
- **HttpOnly:** true
- **Secure:** true (production)
- **SameSite:** 'strict'

