# Face Locker API Documentation

## Overview

API Documentation cho hệ thống **Tủ đồ thông minh Face Locker** tích hợp AI FaceID và ứng dụng di động. Tài liệu này mô tả các API endpoint cần thiết để hỗ trợ các chức năng chính của hệ thống.

**Base URL:** `http://{raspberry_pi_ip}:3000/api/v1`

**Authentication:** Bearer Token (JWT)

---

## Table of Contents

1. [Authentication APIs](#1-authentication-apis)
2. [User APIs](#2-user-apis)
3. [Locker APIs](#3-locker-apis)
4. [Session APIs](#4-session-apis)
5. [QR Token APIs](#5-qr-token-apis)
6. [Transaction Log APIs](#6-transaction-log-apis)
7. [Security Alert APIs](#7-security-alert-apis)
8. [Hardware Control APIs](#8-hardware-control-apis)
9. [Statistics APIs](#9-statistics-apis)

---

## 1. Authentication APIs

### 1.1. Đăng ký người dùng

**Endpoint:** `POST /auth/register`

**Description:** Đăng ký tài khoản người dùng mới (US10: Face registration for future recognition)

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "full_name": "Nguyễn Văn A",
  "phone": "0901234567"
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "message": "Đăng ký thành công",
  "data": {
    "id": "uuid-xxx",
    "email": "user@example.com",
    "full_name": "Nguyễn Văn A",
    "role": "customer",
    "created_at": "2026-01-18T10:00:00Z"
  }
}
```

---

### 1.2. Đăng nhập

**Endpoint:** `POST /auth/login`

**Description:** Đăng nhập và nhận JWT token

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "user": {
      "id": "uuid-xxx",
      "email": "user@example.com",
      "full_name": "Nguyễn Văn A",
      "role": "customer"
    }
  }
}
```

---

### 1.3. Đăng xuất

**Endpoint:** `POST /auth/logout`

**Description:** Hủy session và token hiện tại

**Headers:** `Authorization: Bearer {access_token}`

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Đăng xuất thành công"
}
```

---

### 1.4. Làm mới Token

**Endpoint:** `POST /auth/refresh`

**Description:** Làm mới access token bằng refresh token

**Request Body:**

```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600
  }
}
```

---

## 2. User APIs

### 2.1. Lấy thông tin người dùng hiện tại

**Endpoint:** `GET /users/me`

**Description:** Lấy thông tin chi tiết của người dùng đang đăng nhập

**Headers:** `Authorization: Bearer {access_token}`

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "uuid-xxx",
    "email": "user@example.com",
    "full_name": "Nguyễn Văn A",
    "phone": "0901234567",
    "role": "customer",
    "is_active": true,
    "created_at": "2026-01-18T10:00:00Z",
    "updated_at": "2026-01-18T10:00:00Z"
  }
}
```

---

### 2.2. Cập nhật thông tin người dùng

**Endpoint:** `PUT /users/me`

**Description:** Cập nhật thông tin cá nhân

**Headers:** `Authorization: Bearer {access_token}`

**Request Body:**

```json
{
  "full_name": "Nguyễn Văn B",
  "phone": "0909876543"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Cập nhật thành công",
  "data": {
    "id": "uuid-xxx",
    "email": "user@example.com",
    "full_name": "Nguyễn Văn B",
    "phone": "0909876543"
  }
}
```

---

### 2.3. Đăng ký khuôn mặt (US10)

**Endpoint:** `POST /users/me/face-registration`

**Description:** Đăng ký vector đặc trưng khuôn mặt cho nhận diện trong tương lai

**Headers:**

- `Authorization: Bearer {access_token}`
- `Content-Type: multipart/form-data`

**Request Body:**

```
face_image: [File - ảnh khuôn mặt]
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Đăng ký khuôn mặt thành công",
  "data": {
    "face_registered": true,
    "registered_at": "2026-01-18T10:00:00Z"
  }
}
```

---

### 2.4. Danh sách người dùng (Admin)

**Endpoint:** `GET /users`

**Description:** Lấy danh sách tất cả người dùng (chỉ Admin)

**Headers:** `Authorization: Bearer {access_token}`

**Query Parameters:**

- `page` (optional): Số trang (default: 1)
- `limit` (optional): Số lượng mỗi trang (default: 20)
- `role` (optional): Lọc theo role (customer, admin)
- `is_active` (optional): Lọc theo trạng thái active

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "uuid-xxx",
        "email": "user@example.com",
        "full_name": "Nguyễn Văn A",
        "role": "customer",
        "is_active": true
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 100,
      "total_pages": 5
    }
  }
}
```

---

## 3. Locker APIs

### 3.1. Lấy danh sách tủ

**Endpoint:** `GET /lockers`

**Description:** Lấy danh sách tất cả các ngăn tủ và trạng thái

**Headers:** `Authorization: Bearer {access_token}`

**Query Parameters:**

- `status` (optional): Lọc theo trạng thái (available, in_use, maintenance)
- `size` (optional): Lọc theo kích thước (small, medium, large)
- `location` (optional): Lọc theo vị trí

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "lockers": [
      {
        "id": "uuid-xxx",
        "locker_code": "A01",
        "location": "Tầng 1 - Khu A",
        "size": "medium",
        "status": "available",
        "door_state": "closed",
        "esp32_id": "ESP32_001",
        "relay_pin": 12,
        "sensor_pin": 14
      },
      {
        "id": "uuid-yyy",
        "locker_code": "A02",
        "location": "Tầng 1 - Khu A",
        "size": "large",
        "status": "in_use",
        "door_state": "closed"
      }
    ],
    "summary": {
      "total": 20,
      "available": 15,
      "in_use": 4,
      "maintenance": 1
    }
  }
}
```

---

### 3.2. Lấy chi tiết tủ

**Endpoint:** `GET /lockers/{locker_id}`

**Description:** Lấy thông tin chi tiết của một ngăn tủ

**Headers:** `Authorization: Bearer {access_token}`

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "uuid-xxx",
    "locker_code": "A01",
    "location": "Tầng 1 - Khu A",
    "size": "medium",
    "status": "in_use",
    "door_state": "closed",
    "esp32_id": "ESP32_001",
    "relay_pin": 12,
    "sensor_pin": 14,
    "current_session": {
      "id": "session-uuid",
      "user_id": "user-uuid",
      "check_in_at": "2026-01-18T09:00:00Z",
      "auth_method": "face_id"
    },
    "created_at": "2026-01-01T00:00:00Z",
    "updated_at": "2026-01-18T09:00:00Z"
  }
}
```

---

### 3.3. Tạo tủ mới (Admin - US05)

**Endpoint:** `POST /lockers`

**Description:** Thêm ngăn tủ mới vào hệ thống (CRUD)

**Headers:** `Authorization: Bearer {access_token}`

**Request Body:**

```json
{
  "locker_code": "B01",
  "location": "Tầng 2 - Khu B",
  "size": "large",
  "esp32_id": "ESP32_002",
  "relay_pin": 12,
  "sensor_pin": 14
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "message": "Tạo tủ thành công",
  "data": {
    "id": "uuid-xxx",
    "locker_code": "B01",
    "location": "Tầng 2 - Khu B",
    "size": "large",
    "status": "available",
    "door_state": "closed"
  }
}
```

---

### 3.4. Cập nhật tủ (Admin - US05)

**Endpoint:** `PUT /lockers/{locker_id}`

**Description:** Cập nhật thông tin ngăn tủ

**Headers:** `Authorization: Bearer {access_token}`

**Request Body:**

```json
{
  "location": "Tầng 2 - Khu C",
  "size": "medium"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Cập nhật thành công"
}
```

---

### 3.5. Xóa tủ (Admin - US05)

**Endpoint:** `DELETE /lockers/{locker_id}`

**Description:** Xóa ngăn tủ khỏi hệ thống

**Headers:** `Authorization: Bearer {access_token}`

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Xóa tủ thành công"
}
```

---

### 3.6. Cập nhật trạng thái tủ (Admin - US07)

**Endpoint:** `PATCH /lockers/{locker_id}/status`

**Description:** Thay đổi trạng thái tủ (đặt bảo trì)

**Headers:** `Authorization: Bearer {access_token}`

**Request Body:**

```json
{
  "status": "maintenance",
  "reason": "Khóa bị kẹt, cần sửa chữa"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Cập nhật trạng thái thành công"
}
```

---

### 3.7. Mở tủ cưỡng chế (Admin - US06)

**Endpoint:** `POST /lockers/{locker_id}/force-open`

**Description:** Admin mở cưỡng chế một ngăn tủ (bypass)

**Headers:** `Authorization: Bearer {access_token}`

**Request Body:**

```json
{
  "reason": "Khách hàng quên xác thực, yêu cầu hỗ trợ"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Lệnh mở tủ đã được gửi",
  "data": {
    "locker_id": "uuid-xxx",
    "command_sent_at": "2026-01-18T10:00:00Z",
    "door_state": "open"
  }
}
```

---

## 4. Session APIs

### 4.1. Check-in bằng FaceID (US01)

**Endpoint:** `POST /sessions/check-in/face`

**Description:** Gửi đồ sử dụng AI FaceID

Quy trình:

1. Camera quét khuôn mặt
2. Pi trích xuất vector đặc trưng
3. Backend tìm ngăn tủ trống
4. Gửi lệnh UART mở khóa
5. Cảm biến xác nhận cửa đóng
6. DB cập nhật trạng thái "Đang sử dụng"

**Headers:**

- `Authorization: Bearer {access_token}` (optional - cho khách vãng lai)
- `Content-Type: multipart/form-data`

**Request Body:**

```
face_image: [File - ảnh khuôn mặt]
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Check-in thành công",
  "data": {
    "session_id": "session-uuid",
    "locker": {
      "id": "locker-uuid",
      "locker_code": "A01",
      "location": "Tầng 1 - Khu A"
    },
    "check_in_at": "2026-01-18T10:00:00Z",
    "auth_method": "face_id"
  }
}
```

**Response (503 Service Unavailable - Hết tủ):**

```json
{
  "success": false,
  "error": {
    "code": "NO_AVAILABLE_LOCKER",
    "message": "Hiện tại không còn tủ trống"
  }
}
```

---

### 4.2. Check-in bằng QR Code (US02)

**Endpoint:** `POST /sessions/check-in/qr`

**Description:** Gửi đồ sử dụng mã QR động từ App

Quy trình:

1. App tạo Dynamic QR
2. Camera giải mã tại tủ đồ
3. Pi xác thực user qua API
4. Cấp phát tủ tương tự FaceID

**Headers:** None (QR token chứa thông tin xác thực)

**Request Body:**

```json
{
  "qr_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Check-in thành công",
  "data": {
    "session_id": "session-uuid",
    "locker": {
      "id": "locker-uuid",
      "locker_code": "A02",
      "location": "Tầng 1 - Khu A"
    },
    "check_in_at": "2026-01-18T10:00:00Z",
    "auth_method": "qr_code"
  }
}
```

---

### 4.3. Cập nhật phiên (Thêm đồ - US03)

**Endpoint:** `POST /sessions/{session_id}/update`

**Description:** Mở lại tủ đang thuê để thêm đồ (không kết thúc session)

**Headers:**

- `Authorization: Bearer {access_token}`
- `Content-Type: multipart/form-data`

**Request Body:**

```
face_image: [File - ảnh khuôn mặt] (hoặc qr_token)
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Xác thực thành công, tủ đã mở",
  "data": {
    "session_id": "session-uuid",
    "locker_code": "A01",
    "last_access_at": "2026-01-18T11:00:00Z",
    "door_state": "open"
  }
}
```

---

### 4.4. Check-out (US04)

**Endpoint:** `POST /sessions/{session_id}/check-out`

**Description:** Lấy đồ và kết thúc phiên sử dụng

Quy trình:

1. Xác thực khuôn mặt/QR
2. So khớp với vector từ lúc check-in
3. UART mở đúng ngăn tủ
4. DB cập nhật trạng thái "Trống"

**Headers:**

- `Authorization: Bearer {access_token}`
- `Content-Type: multipart/form-data`

**Request Body:**

```
face_image: [File - ảnh khuôn mặt]
```

hoặc

```json
{
  "qr_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Check-out thành công",
  "data": {
    "session_id": "session-uuid",
    "locker_code": "A01",
    "check_in_at": "2026-01-18T10:00:00Z",
    "check_out_at": "2026-01-18T12:00:00Z",
    "duration_minutes": 120
  }
}
```

**Response (401 Unauthorized - Xác thực thất bại):**

```json
{
  "success": false,
  "error": {
    "code": "FACE_MISMATCH",
    "message": "Khuôn mặt không khớp với người gửi đồ"
  }
}
```

---

### 4.5. Lấy danh sách phiên của người dùng

**Endpoint:** `GET /sessions/my-sessions`

**Description:** Lấy lịch sử các phiên sử dụng tủ của người dùng

**Headers:** `Authorization: Bearer {access_token}`

**Query Parameters:**

- `status` (optional): Lọc theo trạng thái (active, completed, cancelled)
- `page`, `limit`: Phân trang

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "sessions": [
      {
        "id": "session-uuid-1",
        "locker_code": "A01",
        "status": "active",
        "auth_method": "face_id",
        "check_in_at": "2026-01-18T10:00:00Z",
        "check_out_at": null
      },
      {
        "id": "session-uuid-2",
        "locker_code": "B02",
        "status": "completed",
        "auth_method": "qr_code",
        "check_in_at": "2026-01-17T14:00:00Z",
        "check_out_at": "2026-01-17T16:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 5
    }
  }
}
```

---

### 4.6. Lấy phiên đang active

**Endpoint:** `GET /sessions/active`

**Description:** Lấy phiên sử dụng tủ đang hoạt động của người dùng

**Headers:** `Authorization: Bearer {access_token}`

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "session": {
      "id": "session-uuid",
      "locker": {
        "id": "locker-uuid",
        "locker_code": "A01",
        "location": "Tầng 1 - Khu A"
      },
      "status": "active",
      "auth_method": "face_id",
      "check_in_at": "2026-01-18T10:00:00Z",
      "last_access_at": "2026-01-18T11:00:00Z"
    }
  }
}
```

---

## 5. QR Token APIs

### 5.1. Tạo mã QR động (US02)

**Endpoint:** `POST /qr-tokens/generate`

**Description:** App tạo Dynamic QR để xác thực tại tủ đồ

**Headers:** `Authorization: Bearer {access_token}`

**Request Body:**

```json
{
  "action": "check_in" | "check_out" | "update",
  "session_id": "session-uuid" // (bắt buộc cho check_out và update)
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "id": "token-uuid",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "qr_code_base64": "data:image/png;base64,iVBORw0KGgo...",
    "expires_at": "2026-01-18T10:05:00Z",
    "expires_in_seconds": 300
  }
}
```

---

### 5.2. Xác thực mã QR

**Endpoint:** `POST /qr-tokens/verify`

**Description:** Pi xác thực mã QR được quét

**Request Body:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "valid": true,
    "user_id": "user-uuid",
    "action": "check_in",
    "session_id": null
  }
}
```

---

## 6. Transaction Log APIs

### 6.1. Lấy nhật ký giao dịch (Admin - US08)

**Endpoint:** `GET /transaction-logs`

**Description:** Lấy lịch sử các hoạt động của hệ thống

**Headers:** `Authorization: Bearer {access_token}` (chỉ Admin)

**Query Parameters:**

- `locker_id` (optional): Lọc theo tủ
- `user_id` (optional): Lọc theo người dùng
- `action_type` (optional): Lọc theo loại hành động (check_in, check_out, update, force_open, maintenance, sensor_trigger)
- `auth_method` (optional): Lọc theo phương thức xác thực
- `from_date`, `to_date`: Khoảng thời gian
- `page`, `limit`: Phân trang

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "logs": [
      {
        "id": "log-uuid",
        "session_id": "session-uuid",
        "locker_id": "locker-uuid",
        "locker_code": "A01",
        "user_id": "user-uuid",
        "user_name": "Nguyễn Văn A",
        "action_type": "check_in",
        "auth_method": "face_id",
        "details": {
          "confidence_score": 0.98
        },
        "device_info": "Pi_01",
        "created_at": "2026-01-18T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 1000
    }
  }
}
```

---

## 7. Security Alert APIs

### 7.1. Lấy danh sách cảnh báo (Admin - US09)

**Endpoint:** `GET /security-alerts`

**Description:** Lấy danh sách cảnh báo bảo mật (cạy phá, kẹt khóa, cảm biến bất thường)

**Headers:** `Authorization: Bearer {access_token}` (chỉ Admin)

**Query Parameters:**

- `locker_id` (optional): Lọc theo tủ
- `alert_type` (optional): Lọc theo loại (unauthorized_open, tamper, sensor_malfunction, lock_jam)
- `severity` (optional): Lọc theo mức độ (low, medium, high, critical)
- `is_resolved` (optional): Lọc theo trạng thái xử lý
- `page`, `limit`: Phân trang

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "alerts": [
      {
        "id": "alert-uuid",
        "locker_id": "locker-uuid",
        "locker_code": "A01",
        "alert_type": "unauthorized_open",
        "severity": "critical",
        "description": "Cửa tủ A01 mở không có lệnh unlock",
        "is_resolved": false,
        "resolved_by": null,
        "created_at": "2026-01-18T10:00:00Z"
      }
    ],
    "summary": {
      "total": 5,
      "unresolved": 2,
      "critical_unresolved": 1
    }
  }
}
```

---

### 7.2. Xử lý cảnh báo

**Endpoint:** `PATCH /security-alerts/{alert_id}/resolve`

**Description:** Đánh dấu cảnh báo đã được xử lý

**Headers:** `Authorization: Bearer {access_token}` (chỉ Admin)

**Request Body:**

```json
{
  "resolution_note": "Đã kiểm tra và khắc phục, do cảm biến lỗi"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Cảnh báo đã được đánh dấu xử lý",
  "data": {
    "id": "alert-uuid",
    "is_resolved": true,
    "resolved_by": "admin-uuid",
    "resolved_at": "2026-01-18T11:00:00Z"
  }
}
```

---

## 8. Hardware Control APIs

### 8.1. Gửi lệnh UART đến ESP32

**Endpoint:** `POST /hardware/uart/send`

**Description:** Gửi lệnh điều khiển trực tiếp đến ESP32 (Internal API)

**Request Body:**

```json
{
  "esp32_id": "ESP32_001",
  "command": "unlock",
  "locker_id": "locker-uuid",
  "relay_pin": 12
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Lệnh đã được gửi",
  "data": {
    "command_id": "cmd-uuid",
    "sent_at": "2026-01-18T10:00:00Z"
  }
}
```

---

### 8.2. Nhận trạng thái cảm biến từ ESP32

**Endpoint:** `POST /hardware/sensor/status`

**Description:** ESP32 gửi trạng thái cảm biến cửa (Internal API/WebSocket)

**Request Body:**

```json
{
  "esp32_id": "ESP32_001",
  "locker_id": "locker-uuid",
  "sensor_pin": 14,
  "door_state": "closed",
  "timestamp": "2026-01-18T10:00:00Z"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Trạng thái đã được cập nhật"
}
```

---

### 8.3. Báo cáo lỗi phần cứng

**Endpoint:** `POST /hardware/error`

**Description:** ESP32 báo cáo lỗi phần cứng (kẹt khóa, cảm biến lỗi)

**Request Body:**

```json
{
  "esp32_id": "ESP32_001",
  "locker_id": "locker-uuid",
  "error_type": "lock_jam",
  "description": "Relay kích nhưng khóa không mở",
  "timestamp": "2026-01-18T10:00:00Z"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Lỗi đã được ghi nhận",
  "data": {
    "alert_id": "alert-uuid"
  }
}
```

---

## 9. Statistics APIs

### 9.1. Thống kê tổng quan (Admin - US08)

**Endpoint:** `GET /statistics/overview`

**Description:** Lấy thống kê tổng quan về hệ thống

**Headers:** `Authorization: Bearer {access_token}` (chỉ Admin)

**Query Parameters:**

- `from_date`, `to_date`: Khoảng thời gian thống kê

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "period": {
      "from": "2026-01-01T00:00:00Z",
      "to": "2026-01-18T23:59:59Z"
    },
    "lockers": {
      "total": 20,
      "available": 15,
      "in_use": 4,
      "maintenance": 1
    },
    "sessions": {
      "total": 500,
      "completed": 480,
      "active": 4,
      "cancelled": 16
    },
    "usage": {
      "average_duration_minutes": 45,
      "peak_hours": ["10:00", "14:00", "18:00"],
      "busiest_day": "Saturday"
    },
    "auth_methods": {
      "face_id": 350,
      "qr_code": 150
    },
    "alerts": {
      "total": 10,
      "unresolved": 2
    }
  }
}
```

---

### 9.2. Thống kê sử dụng theo ngày

**Endpoint:** `GET /statistics/usage/daily`

**Description:** Lấy thống kê lượt sử dụng theo ngày

**Headers:** `Authorization: Bearer {access_token}` (chỉ Admin)

**Query Parameters:**

- `from_date`, `to_date`: Khoảng thời gian
- `locker_id` (optional): Lọc theo tủ cụ thể

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "daily_usage": [
      {
        "date": "2026-01-17",
        "check_ins": 25,
        "check_outs": 24,
        "average_duration_minutes": 42
      },
      {
        "date": "2026-01-18",
        "check_ins": 30,
        "check_outs": 28,
        "average_duration_minutes": 48
      }
    ]
  }
}
```

---

### 9.3. Thống kê theo tủ

**Endpoint:** `GET /statistics/lockers`

**Description:** Lấy thống kê sử dụng từng ngăn tủ

**Headers:** `Authorization: Bearer {access_token}` (chỉ Admin)

**Query Parameters:**

- `from_date`, `to_date`: Khoảng thời gian

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "locker_stats": [
      {
        "locker_id": "locker-uuid",
        "locker_code": "A01",
        "total_sessions": 50,
        "total_duration_hours": 120,
        "utilization_rate": 0.75,
        "last_used_at": "2026-01-18T10:00:00Z"
      }
    ]
  }
}
```

---

## WebSocket Events

### Real-time Updates

**Connection:** `ws://{raspberry_pi_ip}:3000/ws`

**Authentication:** Gửi JWT token sau khi kết nối

### Events từ Server

#### Locker Status Update

```json
{
  "event": "locker_status_changed",
  "data": {
    "locker_id": "locker-uuid",
    "locker_code": "A01",
    "status": "in_use",
    "door_state": "closed"
  }
}
```

#### Security Alert

```json
{
  "event": "security_alert",
  "data": {
    "alert_id": "alert-uuid",
    "locker_code": "A01",
    "alert_type": "unauthorized_open",
    "severity": "critical",
    "message": "Cảnh báo: Cửa tủ A01 mở bất thường!"
  }
}
```

#### Session Update

```json
{
  "event": "session_updated",
  "data": {
    "session_id": "session-uuid",
    "action": "check_in",
    "locker_code": "A01"
  }
}
```

---

## Error Codes

| Code                  | HTTP Status | Description                       |
| --------------------- | ----------- | --------------------------------- |
| `INVALID_CREDENTIALS` | 401         | Email hoặc mật khẩu không đúng    |
| `TOKEN_EXPIRED`       | 401         | Token đã hết hạn                  |
| `TOKEN_INVALID`       | 401         | Token không hợp lệ                |
| `FORBIDDEN`           | 403         | Không có quyền truy cập           |
| `USER_NOT_FOUND`      | 404         | Không tìm thấy người dùng         |
| `LOCKER_NOT_FOUND`    | 404         | Không tìm thấy tủ                 |
| `SESSION_NOT_FOUND`   | 404         | Không tìm thấy phiên              |
| `NO_AVAILABLE_LOCKER` | 503         | Không còn tủ trống                |
| `FACE_MISMATCH`       | 401         | Khuôn mặt không khớp              |
| `QR_EXPIRED`          | 401         | Mã QR đã hết hạn                  |
| `QR_USED`             | 401         | Mã QR đã được sử dụng             |
| `LOCKER_IN_USE`       | 409         | Tủ đang được sử dụng              |
| `LOCKER_MAINTENANCE`  | 503         | Tủ đang bảo trì                   |
| `HARDWARE_ERROR`      | 500         | Lỗi phần cứng                     |
| `UART_TIMEOUT`        | 500         | Không nhận được phản hồi từ ESP32 |

---

## Response Format

### Success Response

```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": { ... }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": { ... }
  }
}
```

---

## Rate Limiting

- **Authentication endpoints:** 5 requests/minute
- **General API:** 100 requests/minute
- **Hardware control:** 10 requests/second

---

## Versioning

API sử dụng versioning trong URL path: `/api/v1/...`

Khi có breaking changes, version mới sẽ được tạo: `/api/v2/...`

---

_Document Version: 1.0_
_Last Updated: 2026-01-18_
