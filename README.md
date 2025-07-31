# API DOCUMENTATION - WebHub Backend

## Tổng quan
WebHub Backend là REST API được xây dựng bằng Spring Boot, cung cấp các dịch vụ quản lý blog, project, event, user và file storage.

**Base URL**: `http://localhost:8080`

## Authentication & Authorization

### JWT Authentication
Hệ thống sử dụng JWT (JSON Web Token) cho authentication:
- **Access Token**: Thời hạn 2 giờ
- **Refresh Token**: Thời hạn 7 ngày
- **Header Format**: `Authorization: Bearer <token>`

### User Roles
- **ROOT**: Quyền cao nhất, có thể thực hiện mọi thao tác
- **ADMIN**: Quản trị viên, có thể quản lý blog, project, event, user
- **MEMBER**: Thành viên thường, chỉ có thể xem và cập nhật profile

---

## 1. AUTHENTICATION ENDPOINTS
**Base Path**: `/api/v1/auth`

### 1.1 Đăng nhập
```http
POST /api/v1/auth/login
```

**Mô tả**: Xác thực người dùng và trả về access token + refresh token

**Request Body**:
```json
{
  "username": "string",
  "password": "string"
}
```

**Response** (200 OK):
```json
{
  "access_token": "string",
  "refresh_token": "string"
}
```

**Error** (401 Unauthorized):
```json
{
  "message": "Thông tin đăng nhập không hợp lệ"
}
```

### 1.2 Làm mới token
```http
POST /api/v1/auth/refresh
```

**Mô tả**: Lấy access token mới từ refresh token

**Request Body**:
```json
{
  "refresh_token": "string"
}
```

**Response** (200 OK):
```json
{
  "access_token": "string",
  "refresh_token": "string"
}
```

### 1.3 Quên mật khẩu
```http
POST /api/v1/auth/forgot-password?email={email}
```

**Mô tả**: Gửi email khôi phục mật khẩu

**Parameters**:
- `email` (query, required): Email người dùng

**Response** (201 Created): Void

### 1.4 Xác minh tài khoản
```http
GET /api/v1/auth/verify/{token}
```

**Mô tả**: Xác minh email người dùng qua token

**Parameters**:
- `token` (path, required): Token xác minh

**Response** (200 OK): Void

### 1.5 Reset mật khẩu
```http
POST /api/v1/auth/reset-password
```

**Mô tả**: Đặt lại mật khẩu mới từ mã xác minh

**Request Body**:
```json
{
  "token": "string",
  "newPassword": "string"
}
```

**Response** (200 OK):
```json
"Reset password successfully"
```

### 1.6 Đổi mật khẩu
```http
POST /api/v1/auth/change-password
```
**Authentication**: Required

**Mô tả**: Người dùng đổi mật khẩu trong khi đang đăng nhập

**Request Body**:
```json
{
  "oldPassword": "string",
  "newPassword": "string"
}
```

**Response** (200 OK):
```json
"Change password successful"
```

### 1.7 Đăng xuất
```http
POST /api/v1/auth/logout
```
**Authentication**: Required

**Mô tả**: Đăng xuất và thu hồi refresh token

**Response** (200 OK):
```json
"Log out successfully"
```

---

## 2. USER MANAGEMENT ENDPOINTS
**Base Path**: `/api/v1`

### 2.1 Tìm kiếm người dùng
```http
GET /api/v1/users
```
**Roles**: ADMIN, ROOT

**Mô tả**: Trả về danh sách người dùng phù hợp với tiêu chí tìm kiếm

**Query Parameters**:
- `name` (optional): Tên người dùng
- `email` (optional): Email
- `role` (optional): UserRole (ROOT, ADMIN, MEMBER)
- `status` (optional): Boolean trạng thái active

**Response** (200 OK):
```json
[
  {
    "id": "long",
    "fullName": "string",
    "email": "string",
    "role": "ROOT|ADMIN|MEMBER",
    "isActive": "boolean",
    "avatarUrl": "string"
  }
]
```

### 2.2 Tạo người dùng mới
```http
POST /api/v1/users
```
**Roles**: ADMIN, ROOT

**Request Body**:
```json
{
  "fullName": "string",
  "email": "string",
  "password": "string",
  "role": "ROOT|ADMIN|MEMBER"
}
```

**Response** (201 Created):
```json
{
  "id": "long",
  "message": "Tạo người dùng thành công"
}
```

### 2.3 Cập nhật thông tin người dùng
```http
PATCH /api/v1/users/{id}
```
**Roles**: ADMIN, ROOT

**Path Parameters**:
- `id` (required): ID người dùng

**Request Body**:
```json
{
  "fullName": "string",
  "role": "ROOT|ADMIN|MEMBER"
}
```

**Response** (200 OK):
```json
{
  "id": "long",
  "fullName": "string",
  "email": "string",
  "role": "ROOT|ADMIN|MEMBER",
  "isActive": "boolean"
}
```

### 2.4 Cập nhật người dùng (multipart)
```http
PATCH /api/v1/users/{id}/multipart
```
**Roles**: ADMIN, ROOT
**Content-Type**: multipart/form-data

**Form Parameters**:
- `full_name` (optional): string
- `role` (optional): UserRole
- `avatar` (optional): MultipartFile

### 2.5 Cập nhật trạng thái người dùng
```http
PATCH /api/v1/users/{id}/status
```
**Roles**: ADMIN, ROOT

**Request Body**:
```json
{
  "isActive": "boolean"
}
```

### 2.6 Xóa người dùng
```http
DELETE /api/v1/users/{id}
```
**Roles**: ROOT

**Request Body**:
```json
{
  "confirmDelete": true
}
```

### 2.7 Lấy thông tin người dùng hiện tại
```http
GET /api/v1/users/me
```
**Authentication**: Required

**Response** (200 OK):
```json
{
  "id": "long",
  "fullName": "string",
  "email": "string",
  "role": "ROOT|ADMIN|MEMBER",
  "isActive": "boolean",
  "avatarUrl": "string"
}
```

### 2.8 Cập nhật profile
```http
PATCH /api/v1/users/me
```
**Authentication**: Required

**Request Body**:
```json
{
  "fullName": "string"
}
```

### 2.9 Cập nhật avatar
```http
PATCH /api/v1/users/me/avatar
```
**Authentication**: Required
**Content-Type**: multipart/form-data

**Form Parameters**:
- `avatar` (optional): MultipartFile

---

## 3. BLOG ENDPOINTS
**Base Path**: `/api`

### 3.1 PUBLIC ENDPOINTS

#### 3.1.1 Lấy tất cả bài viết blog đã xuất bản
```http
GET /api/blog-posts
```

**Query Parameters**:
- `page` (optional): Số trang (default: 0)
- `size` (optional): Kích thước trang (default: 10)
- `search` (optional): Từ khóa tìm kiếm
- `tag` (optional): Tag filter

**Response** (200 OK):
```json
{
  "data": [
    {
      "id": "long",
      "title": "string",
      "content": "string",
      "thumbnailUrl": "string",
      "slug": "string",
      "publishedAt": "datetime",
      "tags": ["string"]
    }
  ],
  "pagination": {
    "totalItems": "long",
    "totalPages": "integer",
    "currentPage": "integer",
    "nextPage": "integer",
    "prevPage": "integer"
  }
}
```

#### 3.1.2 Lấy các bài viết blog nổi bật
```http
GET /api/blog-posts/featured
```

**Response** (200 OK):
```json
[
  {
    "id": "long",
    "title": "string",
    "thumbnailUrl": "string",
    "slug": "string"
  }
]
```

#### 3.1.3 Lấy chi tiết bài viết theo slug
```http
GET /api/blog-posts/{slug}
```

**Path Parameters**:
- `slug` (required): Slug của bài viết

**Response** (200 OK):
```json
{
  "id": "long",
  "title": "string",
  "content": "string",
  "thumbnailUrl": "string",
  "slug": "string",
  "publishedAt": "datetime",
  "author": {
    "id": "long",
    "fullName": "string"
  },
  "tags": ["string"]
}
```

### 3.2 ADMIN ENDPOINTS

#### 3.2.1 Lấy tất cả bài viết cho admin
```http
GET /api/admin/blog-posts
```
**Roles**: ADMIN, ROOT

**Query Parameters**:
- `page`, `size`: Pagination
- `search` (optional): Từ khóa tìm kiếm
- `status` (optional): BlogStatus (DRAFT, PUBLISHED, ARCHIVED)
- `author_id` (optional): ID tác giả

#### 3.2.2 Tạo bài viết blog mới
```http
POST /api/admin/blog-posts?user_id={user_id}
```
**Roles**: ADMIN, ROOT

**Query Parameters**:
- `user_id` (required): ID người tạo

**Request Body**:
```json
{
  "title": "string",
  "content": "string",
  "thumbnailUrl": "string",
  "status": "DRAFT|PUBLISHED|ARCHIVED",
  "isPinned": "boolean",
  "tagIds": ["long"]
}
```

**Response** (201 Created):
```json
{
  "id": "long",
  "title": "string",
  "slug": "string",
  "status": "string",
  "message": "Tạo bài viết thành công"
}
```

#### 3.2.3 Cập nhật bài viết
```http
PUT /api/admin/blog-posts/{id}
```
**Roles**: ADMIN, ROOT

#### 3.2.4 Xóa bài viết
```http
DELETE /api/admin/blog-posts/{id}
```
**Roles**: ADMIN, ROOT

#### 3.2.5 Thay đổi trạng thái bài viết
```http
PATCH /api/admin/blog-posts/{id}/status
```
**Roles**: ADMIN, ROOT

**Request Body**:
```json
{
  "status": "DRAFT|PUBLISHED|ARCHIVED"
}
```

#### 3.2.6 Ghim/bỏ ghim bài viết
```http
PATCH /api/admin/blog-posts/{id}/pin
```
**Roles**: ADMIN, ROOT

**Request Body**:
```json
{
  "isPinned": "boolean"
}
```

#### 3.2.7 Lấy blog theo ID để edit
```http
GET /api/admin/blog-posts/{id}
```
**Roles**: ADMIN, ROOT

#### 3.2.8 Utility endpoints
```http
GET /api/blog-posts/recent?limit={limit}
GET /api/admin/blog-posts/count
GET /api/admin/blog-posts/count/status/{status}
GET /api/admin/blog-posts/exists/{id}
```
**Roles**: ADMIN, ROOT

---

## 4. PROJECT ENDPOINTS
**Base Path**: `/api/v1/projects`

### 4.1 Tạo project
```http
POST /api/v1/projects
```
**Roles**: ADMIN, ROOT

**Request Body**:
```json
{
  "title": "string",
  "description": "string",
  "type": "PROJECT_TYPE",
  "status": "PROJECT_STATUS",
  "thumbnailUrl": "string",
  "tagIds": ["long"]
}
```

### 4.2 Lấy tất cả projects
```http
GET /api/v1/projects
```

**Query Parameters**:
- `page` (default: 0): Số trang
- `size` (default: 10): Kích thước trang

**Response** (200 OK):
```json
{
  "content": [
    {
      "id": "long",
      "title": "string",
      "description": "string",
      "type": "string",
      "status": "string",
      "thumbnailUrl": "string",
      "isFeatured": "boolean"
    }
  ],
  "totalElements": "long",
  "totalPages": "integer",
  "currentPage": "integer"
}
```

### 4.3 Lấy project theo ID
```http
GET /api/v1/projects/{id}
```

### 4.4 Lấy projects theo tags
```http
POST /api/v1/projects/tags
```

**Request Body**:
```json
["long"]
```

### 4.5 Lấy tags của project
```http
GET /api/v1/projects/{projectId}/tags
```

### 4.6 Filter projects
```http
POST /api/v1/projects/filter
```

**Request Body**:
```json
{
  "title": "string",
  "type": "PROJECT_TYPE",
  "status": "PROJECT_STATUS",
  "tagIds": ["long"]
}
```

### 4.7 Lấy projects theo type
```http
GET /api/v1/projects/type?type={type}&page={page}&size={size}
```

### 4.8 Cập nhật project
```http
PUT /api/v1/projects/{id}
```
**Roles**: ADMIN, ROOT

### 4.9 Cập nhật trạng thái project
```http
PUT /api/v1/projects/{id}/status
```
**Roles**: ADMIN, ROOT

### 4.10 Cập nhật type project
```http
PUT /api/v1/projects/{id}/type
```
**Roles**: ADMIN, ROOT

### 4.11 Cập nhật featured status
```http
PUT /api/v1/projects/{id}/isFeatured
```
**Roles**: ADMIN, ROOT

### 4.12 Xóa project
```http
DELETE /api/v1/projects/{id}
```
**Roles**: ADMIN, ROOT

---

## 5. EVENT ENDPOINTS

### 5.1 Tạo event
```http
POST /api/v1/events
```
**Roles**: ADMIN

**Request Body**:
```json
{
  "title": "string",
  "description": "string",
  "location": "string",
  "startDate": "datetime",
  "endDate": "datetime",
  "status": "EVENT_STATUS"
}
```

**Response** (201 Created):
```json
{
  "message": "Tạo thành công"
}
```

### 5.2 Tìm kiếm events (User)
```http
GET /api/v1/user/events
```

**Query Parameters**:
- `title` (optional): Tên event
- `location` (optional): Địa điểm
- `status` (optional): EventStatus

**Response** (200 OK):
```json
[
  {
    "id": "long",
    "title": "string",
    "description": "string",
    "location": "string",
    "startDate": "datetime",
    "endDate": "datetime",
    "status": "string"
  }
]
```

---

## 6. FILE STORAGE ENDPOINTS
**Base Path**: `/api/v1/files`

### 6.1 Upload file
```http
POST /api/v1/files/statics/{type}
```
**Content-Type**: multipart/form-data

**Path Parameters**:
- `type` (required): Loại storage (avatars, documents, media, temp, thumbnails)

**Form Parameters**:
- `file` (required): MultipartFile

**Response** (200 OK):
```json
{
  "message": "Upload successful",
  "downloadUrl": "string"
}
```

### 6.2 Download file
```http
GET /api/v1/files/statics/{type}/{filename}
```

**Path Parameters**:
- `type` (required): Loại storage
- `filename` (required): Tên file

**Response**: File binary data

### 6.3 Xóa file
```http
DELETE /api/v1/files/{type}/{filename}
```

### 6.4 Liệt kê files
```http
GET /api/v1/files/statics/{type}
```

**Response** (200 OK):
```json
["string"]
```

---

## 7. FOLDER ENDPOINTS
**Base Path**: `/api/v1/folders`

### 7.1 Lấy thông tin folder
```http
GET /api/v1/folders/{folderId}
```

**Path Parameters**:
- `folderId` (required): ID của folder

**Response** (200 OK):
```json
{
  "id": "long",
  "name": "string",
  "contents": ["object"]
}
```

---

## Error Handling

### Cấu trúc Error Response
```json
{
  "statusCode": "integer",
  "message": "string",
  "error": "string",
  "timestamp": "datetime"
}
```

### Common HTTP Status Codes
- **200 OK**: Thành công
- **201 Created**: Tạo thành công
- **400 Bad Request**: Dữ liệu không hợp lệ
- **401 Unauthorized**: Chưa xác thực
- **403 Forbidden**: Không có quyền truy cập
- **404 Not Found**: Không tìm thấy tài nguyên
- **500 Internal Server Error**: Lỗi server

---

## Security Notes

1. **CORS**: Được cấu hình cho phép cross-origin requests
2. **CSRF**: Đã được disable do sử dụng JWT
3. **Session**: Stateless (không sử dụng session)
4. **JWT Secret**: Cấu hình trong application.properties
5. **OAuth2**: Tích hợp với GitLab OAuth

### Security Headers
- `Authorization: Bearer <access_token>`
- Content-Type phù hợp với từng endpoint

---

## Configuration

### Database
- **Type**: PostgreSQL
- **Host**: postgres:5432
- **Database**: rtic_db
- **Username**: rtic

### File Upload
- **Max File Size**: 10MB
- **Max Request Size**: 10MB
- **Storage Paths**:
  - avatars: uploads/avatars
  - documents: uploads/documents
  - media: uploads/media
  - temp: uploads/temp
  - thumbnails: uploads/thumbnails

### JWT Configuration
- **Access Token Validity**: 2 hours
- **Refresh Token Validity**: 7 days 
