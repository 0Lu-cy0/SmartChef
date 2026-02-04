# SmartChef API Documentation

## Base URL
```
https://smartchef-1-ipw0.onrender.com
```

## API Version
**v1**

Base path: `/v1`

---

## 📋 Mục Lục
1. [Authentication](#authentication)
2. [Users](#users)
3. [Schemas](#schemas)
4. [Error Responses](#error-responses)

---

## 🔐 Authentication

### Đăng ký tài khoản mới
**POST** `/v1/auth/register`

Đăng ký tài khoản người dùng mới.

**Request Body:**
```json
{
  "name": "Nguyen Van A",
  "email": "nguyenvana@example.com",
  "password": "password123"
}
```

**Requirements:**
- Password: Tối thiểu 8 ký tự, bao gồm ít nhất 1 chữ và 1 số
- Email: Phải unique trong hệ thống

**Response (201 Created):**
```json
{
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "name": "Nguyen Van A",
    "email": "nguyenvana@example.com",
    "role": "user",
    "isEmailVerified": false
  },
  "tokens": {
    "access": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires": "2026-01-21T10:30:00.000Z"
    },
    "refresh": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires": "2026-02-20T10:00:00.000Z"
    }
  }
}
```

---

### Đăng nhập
**POST** `/v1/auth/login`

Đăng nhập vào hệ thống.

**Request Body:**
```json
{
  "email": "nguyenvana@example.com",
  "password": "password123"
}
```

**Response (200 OK):**
```json
{
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "name": "Nguyen Van A",
    "email": "nguyenvana@example.com",
    "role": "user",
    "isEmailVerified": false
  },
  "tokens": {
    "access": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires": "2026-01-21T10:30:00.000Z"
    },
    "refresh": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires": "2026-02-20T10:00:00.000Z"
    }
  }
}
```

**Error Response (401):**
```json
{
  "code": 401,
  "message": "Invalid email or password"
}
```

---

### Đăng xuất
**POST** `/v1/auth/logout`

Đăng xuất khỏi hệ thống.

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (204 No Content)**

---

### Làm mới token
**POST** `/v1/auth/refresh-tokens`

Làm mới access token bằng refresh token.

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**
```json
{
  "access": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires": "2026-01-21T10:30:00.000Z"
  },
  "refresh": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires": "2026-02-20T10:00:00.000Z"
  }
}
```

---

### Quên mật khẩu
**POST** `/v1/auth/forgot-password`

Gửi email reset mật khẩu đến người dùng.

**Request Body:**
```json
{
  "email": "nguyenvana@example.com"
}
```

**Response (204 No Content)**

---

### Reset mật khẩu
**POST** `/v1/auth/reset-password?token={reset_token}`

Reset mật khẩu bằng token nhận được qua email.

**Query Parameters:**
- `token` (required): Token reset mật khẩu

**Request Body:**
```json
{
  "password": "newPassword123"
}
```

**Response (204 No Content)**

---

### Gửi email xác thực
**POST** `/v1/auth/send-verification-email`

Gửi email xác thực đến người dùng.

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (204 No Content)**

---

### Xác thực email
**POST** `/v1/auth/verify-email?token={verify_token}`

Xác thực email bằng token nhận được qua email.

**Query Parameters:**
- `token` (required): Token xác thực email

**Response (204 No Content)**

---

## 👥 Users

### Tạo người dùng mới
**POST** `/v1/users`

Tạo người dùng mới. Chỉ admin mới có quyền thực hiện.

**Headers:**
```
Authorization: Bearer {access_token}
```

**Required Role:** Admin (`manageUsers` permission)

**Request Body:**
```json
{
  "name": "Tran Thi B",
  "email": "tranthib@example.com",
  "password": "password123",
  "role": "user"
}
```

**Roles:**
- `user` - Người dùng thường
- `admin` - Quản trị viên

**Response (201 Created):**
```json
{
  "id": "507f1f77bcf86cd799439012",
  "name": "Tran Thi B",
  "email": "tranthib@example.com",
  "role": "user",
  "isEmailVerified": false,
  "createdAt": "2026-01-21T10:00:00.000Z",
  "updatedAt": "2026-01-21T10:00:00.000Z"
}
```

---

### Lấy danh sách người dùng
**GET** `/v1/users`

Lấy danh sách tất cả người dùng với phân trang và filter. Chỉ admin mới có quyền thực hiện.

**Headers:**
```
Authorization: Bearer {access_token}
```

**Required Role:** Admin (`getUsers` permission)

**Query Parameters:**
- `name` (optional): Filter theo tên
- `role` (optional): Filter theo role (user/admin)
- `sortBy` (optional): Sắp xếp theo trường (vd: `name:asc`, `createdAt:desc`)
- `projectBy` (optional): Chọn/ẩn trường (vd: `password:hide`, `email:include`)
- `limit` (optional, default: 10): Số lượng bản ghi mỗi trang
- `page` (optional, default: 1): Số trang

**Example:**
```
GET /v1/users?role=user&sortBy=createdAt:desc&limit=20&page=1
```

**Response (200 OK):**
```json
{
  "results": [
    {
      "id": "507f1f77bcf86cd799439011",
      "name": "Nguyen Van A",
      "email": "nguyenvana@example.com",
      "role": "user",
      "isEmailVerified": false,
      "createdAt": "2026-01-21T10:00:00.000Z",
      "updatedAt": "2026-01-21T10:00:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 1
}
```

---

### Lấy thông tin người dùng
**GET** `/v1/users/{userId}`

Lấy thông tin một người dùng cụ thể.
- Người dùng chỉ có thể lấy thông tin của chính mình
- Admin có thể lấy thông tin của bất kỳ ai

**Headers:**
```
Authorization: Bearer {access_token}
```

**Path Parameters:**
- `userId` (required): ID của người dùng

**Response (200 OK):**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Nguyen Van A",
  "email": "nguyenvana@example.com",
  "role": "user",
  "isEmailVerified": false,
  "createdAt": "2026-01-21T10:00:00.000Z",
  "updatedAt": "2026-01-21T10:00:00.000Z"
}
```

---

### Cập nhật thông tin người dùng
**PATCH** `/v1/users/{userId}`

Cập nhật thông tin người dùng.
- Người dùng chỉ có thể cập nhật thông tin của chính mình
- Admin có thể cập nhật thông tin của bất kỳ ai

**Headers:**
```
Authorization: Bearer {access_token}
```

**Path Parameters:**
- `userId` (required): ID của người dùng

**Request Body:**
```json
{
  "name": "Nguyen Van A Updated",
  "email": "newemail@example.com",
  "password": "newPassword123"
}
```

**Response (200 OK):**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Nguyen Van A Updated",
  "email": "newemail@example.com",
  "role": "user",
  "isEmailVerified": false,
  "createdAt": "2026-01-21T10:00:00.000Z",
  "updatedAt": "2026-01-21T10:30:00.000Z"
}
```

---

### Xóa người dùng
**DELETE** `/v1/users/{userId}`

Xóa người dùng.
- Người dùng chỉ có thể xóa tài khoản của chính mình
- Admin có thể xóa tài khoản của bất kỳ ai

**Headers:**
```
Authorization: Bearer {access_token}
```

**Path Parameters:**
- `userId` (required): ID của người dùng

**Response (200 OK)**

---

## 📊 Schemas

### User Object
```json
{
  "id": "string",
  "name": "string",
  "email": "string (email format)",
  "role": "user | admin",
  "isEmailVerified": "boolean",
  "createdAt": "string (ISO 8601 datetime)",
  "updatedAt": "string (ISO 8601 datetime)"
}
```

### Token Object
```json
{
  "token": "string (JWT)",
  "expires": "string (ISO 8601 datetime)"
}
```

### Auth Tokens
```json
{
  "access": {
    "token": "string",
    "expires": "string"
  },
  "refresh": {
    "token": "string",
    "expires": "string"
  }
}
```

---

## ❌ Error Responses

### 400 - Bad Request
```json
{
  "code": 400,
  "message": "Validation error details"
}
```

### 401 - Unauthorized
```json
{
  "code": 401,
  "message": "Please authenticate"
}
```

### 403 - Forbidden
```json
{
  "code": 403,
  "message": "Forbidden"
}
```

### 404 - Not Found
```json
{
  "code": 404,
  "message": "Not found"
}
```

### 409 - Duplicate Email
```json
{
  "code": 409,
  "message": "Email already taken"
}
```

### 500 - Internal Server Error
```json
{
  "code": 500,
  "message": "Internal server error"
}
```

---

## 🔑 Authentication

Hầu hết các endpoints yêu cầu authentication. Sử dụng Bearer token trong header:

```
Authorization: Bearer {access_token}
```

Access token có thời gian sống ngắn. Khi hết hạn, sử dụng refresh token để lấy access token mới thông qua endpoint `/v1/auth/refresh-tokens`.

---

## 📝 Notes

1. **Password Requirements:** Mật khẩu phải có ít nhất 8 ký tự, bao gồm ít nhất 1 chữ cái và 1 số
2. **Email Verification:** Sau khi đăng ký, người dùng cần xác thực email để có đầy đủ quyền truy cập
3. **Rate Limiting:** API có rate limiting để bảo vệ khỏi abuse
4. **Pagination:** Các endpoints trả về danh sách đều hỗ trợ pagination với `limit` và `page`
5. **Sorting:** Sử dụng format `field:asc` hoặc `field:desc` cho sortBy parameter

---

## 🌐 Testing API

### Using cURL
```bash
# Register
curl -X POST https://smartchef-1-ipw0.onrender.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","password":"password123"}'

# Login
curl -X POST https://smartchef-1-ipw0.onrender.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Get Users (requires admin token)
curl -X GET https://smartchef-1-ipw0.onrender.com/v1/users \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Using Postman
Import base URL: `https://smartchef-1-ipw0.onrender.com`

Thêm environment variable:
- `baseUrl`: https://smartchef-1-ipw0.onrender.com
- `accessToken`: (sẽ được set sau khi login)

---

## 📞 Support

Nếu có vấn đề hoặc câu hỏi, vui lòng liên hệ team phát triển.

**Last Updated:** January 21, 2026
