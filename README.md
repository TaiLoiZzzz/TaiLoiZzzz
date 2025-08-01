# API DOCUMENTATION - WebHub Backend

## Tổng quan
WebHub Backend là REST API được xây dựng bằng Spring Boot, cung cấp các dịch vụ quản lý blog, project, event, user, document và file storage.

**Base URL**: `http://localhost:8080`

## Authentication & Authorization

### JWT Authentication
Hệ thống sử dụng JWT (JSON Web Token) cho authentication:
- **Access Token**: Thời hạn 2 giờ
- **Refresh Token**: Thời hạn 7 ngày
- **Header Format**: `Authorization: Bearer <token>`

### User Roles
- **ROOT**: Quyền cao nhất, có thể thực hiện mọi thao tác.
- **ADMIN**: Quản trị viên, có thể quản lý blog, project, event, user, document.
- **MEMBER**: Thành viên thường, có thể xem và cập nhật profile, truy cập tài nguyên được chia sẻ.

---

## 1. AUTHENTICATION ENDPOINTS
**Base Path**: `/api/v1/auth`

### 1.1 Đăng nhập
```http
POST /api/v1/auth/login
```
**Mô tả**: Xác thực người dùng và trả về access token + refresh token.

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

**Errors**:
- **401 Unauthorized**: `{"message": "Thông tin đăng nhập không hợp lệ"}`
- **403 Forbidden**: `{"message": "Tài khoản hiện bị khóa"}`

### 1.2 Làm mới token
```http
POST /api/v1/auth/refresh
```
**Mô tả**: Lấy access token mới từ refresh token.

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
**Mô tả**: Gửi email khôi phục mật khẩu.

**Parameters**:
- `email` (query, required): Email người dùng.

**Response** (200 OK): Void

### 1.4 Xác minh tài khoản
```http
GET /api/v1/auth/verify/{token}
```
**Mô tả**: Xác minh email người dùng qua token được gửi trong email.

**Parameters**:
- `token` (path, required): Token xác minh.

**Response** (200 OK): Void

### 1.5 Reset mật khẩu
```http
POST /api/v1/auth/reset-password
```
**Mô tả**: Đặt lại mật khẩu mới từ token đã xác minh.

**Request Body**:
```json
{
  "token": "string",
  "newPassword": "string"
}
```

**Response** (200 OK): `{"message": "Reset password successfully"}`

### 1.6 Đổi mật khẩu
```http
POST /api/v1/auth/change-password
```
**Authentication**: Required

**Mô tả**: Người dùng đổi mật khẩu khi đang đăng nhập.

**Request Body**:
```json
{
  "oldPassword": "string",
  "newPassword": "string"
}
```

**Response** (200 OK): `{"message": "Change password successful"}`

### 1.7 Đăng xuất
```http
POST /api/v1/auth/logout
```
**Authentication**: Required

**Mô tả**: Đăng xuất và thu hồi refresh token.

**Response** (200 OK): `{"message": "Log out successfully"}`

---

## 2. USER MANAGEMENT ENDPOINTS
**Base Path**: `/api/v1/users`

### 2.1 Tìm kiếm người dùng
```http
GET /api/v1/users
```
**Roles**: ADMIN, ROOT

**Mô tả**: Trả về danh sách người dùng phù hợp với tiêu chí tìm kiếm.

**Query Parameters**:
- `name` (optional): Tên người dùng.
- `email` (optional): Email.
- `role` (optional): UserRole (ROOT, ADMIN, MEMBER).
- `status` (optional): Boolean trạng thái active.

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
- `id` (required): ID người dùng.

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
- `avatar` (required): MultipartFile

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

**Response** (200 OK): (Cấu trúc phân trang)
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
  "pagination": { ... }
}
```

#### 3.1.2 Lấy các bài viết blog nổi bật
```http
GET /api/blog-posts/featured
```

#### 3.1.3 Lấy chi tiết bài viết theo slug
```http
GET /api/blog-posts/{slug}
```

#### 3.1.4 Lấy các bài viết gần đây
```http
GET /api/blog-posts/recent?limit={limit}
```

### 3.2 ADMIN ENDPOINTS
**Base Path**: `/api/admin/blog-posts`
**Roles**: ADMIN, ROOT

#### 3.2.1 Lấy tất cả bài viết cho admin
```http
GET /api/admin/blog-posts
```
**Query Parameters**:
- `page`, `size`: Pagination
- `search` (optional): Từ khóa tìm kiếm
- `status` (optional): BlogStatus (DRAFT, PUBLISHED, ARCHIVED)
- `author_id` (optional): ID tác giả

#### 3.2.2 Tạo bài viết blog mới
```http
POST /api/admin/blog-posts?user_id={user_id}
```
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

#### 3.2.3 Cập nhật bài viết
```http
PUT /api/admin/blog-posts/{id}
```
**Request Body**: (Tương tự mục 3.2.2)

#### 3.2.4 Xóa bài viết
```http
DELETE /api/admin/blog-posts/{id}
```

#### 3.2.5 Thay đổi trạng thái bài viết
```http
PATCH /api/admin/blog-posts/{id}/status
```
**Request Body**: `{"status": "DRAFT|PUBLISHED|ARCHIVED"}`

#### 3.2.6 Ghim/bỏ ghim bài viết
```http
PATCH /api/admin/blog-posts/{id}/pin
```
**Request Body**: `{"isPinned": "boolean"}`

#### 3.2.7 Lấy blog theo ID để edit
```http
GET /api/admin/blog-posts/{id}
```

#### 3.2.8 Utility endpoints
```http
GET /api/admin/blog-posts/count
GET /api/admin/blog-posts/count/status/{status}
GET /api/admin/blog-posts/exists/{id}
```

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
- `page` (default: 0), `size` (default: 10): Phân trang.

### 4.3 Lấy project theo ID
```http
GET /api/v1/projects/{id}
```

### 4.4 Cập nhật project
```http
PUT /api/v1/projects/{id}
```
**Roles**: ADMIN, ROOT
**Request Body**: (Tương tự mục 4.1)

### 4.5 Cập nhật trạng thái project
```http
PUT /api/v1/projects/{id}/status
```
**Roles**: ADMIN, ROOT
**Request Body**: `{"status": "PROJECT_STATUS"}`

### 4.6 Cập nhật type project
```http
PUT /api/v1/projects/{id}/type
```
**Roles**: ADMIN, ROOT
**Request Body**: `{"type": "PROJECT_TYPE"}`

### 4.7 Cập nhật featured status
```http
PUT /api/v1/projects/{id}/isFeatured
```
**Roles**: ADMIN, ROOT
**Request Body**: `{"isFeatured": "boolean"}`

### 4.8 Xóa project
```http
DELETE /api/v1/projects/{id}
```
**Roles**: ADMIN, ROOT

### 4.9 Lấy projects theo tags
```http
POST /api/v1/projects/tags
```
**Request Body**: `["long"]` (Mảng các ID của tag)

### 4.10 Filter projects
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

### 4.11 Lấy tags của một project
```http
GET /api/v1/projects/{projectId}/tags
```

---

## 5. EVENT ENDPOINTS
**Base Path**: `/api/v1/events`

### 5.1 Lấy danh sách sự kiện
```http
GET /api/v1/events
```
**Authentication**: Optional (Guest có thể xem)

**Query Parameters**:
- `title` (optional): Tên event.
- `location` (optional): Địa điểm.
- `status` (optional): EventStatus.

**Response** (200 OK): Mảng các đối tượng Event.

### 5.2 Lấy chi tiết sự kiện
```http
GET /api/v1/events/{id}
```
**Authentication**: Optional

### 5.3 Tạo sự kiện mới
```http
POST /api/v1/events
```
**Roles**: ADMIN, ROOT

**Request Body**:
```json
{
  "title": "string",
  "description": "string",
  "location": "string",
  "startDate": "datetime",
  "endDate": "datetime",
  "status": "EVENT_STATUS",
  "isPublic": "boolean",
  "isFeatured": "boolean"
}
```
**Response** (201 Created): `{"message": "Tạo thành công"}`

### 5.4 Cập nhật sự kiện
```http
PUT /api/v1/events/{id}
```
**Roles**: ADMIN, ROOT
**Request Body**: (Tương tự mục 5.3)
**Response** (200 OK): `{"message": "Cập nhật thành công"}`

### 5.5 Xóa sự kiện
```http
DELETE /api/v1/events/{id}
```
**Roles**: ADMIN, ROOT
**Response** (200 OK): `{"message": "Xóa thành công"}`

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
**Response** (200 OK): `["string"]`

---

## 7. FOLDER & DOCUMENT ENDPOINTS
**Base Path**: `/api/v1`

### 7.1 Lấy danh sách thư mục
```http
GET /api/v1/folders
```
**Roles**: MEMBER, ADMIN, ROOT
**Query Parameters**:
- `parent_id` (optional, long): ID của thư mục cha. Nếu không có, lấy thư mục gốc.

### 7.2 Tạo thư mục mới
```http
POST /api/v1/folders
```
**Roles**: ADMIN, ROOT
**Request Body**: `{"name": "string", "parent_folder_id": "long"}`

### 7.3 Cập nhật thư mục
```http
PATCH /api/v1/folders/{id}
```
**Roles**: ADMIN, ROOT
**Request Body**: `{"name": "string", "parent_folder_id": "long"}`

### 7.4 Xóa thư mục
```http
DELETE /api/v1/folders/{id}
```
**Roles**: ADMIN, ROOT
**Description**: Xóa vĩnh viễn thư mục và toàn bộ nội dung bên trong.

### 7.5 Lấy danh sách tài liệu
```http
GET /api/v1/documents
```
**Roles**: MEMBER, ADMIN, ROOT
**Query Parameters**:
- `q` (optional): Từ khóa tìm kiếm.
- `tags` (optional): Chuỗi ID các tag, cách nhau bởi dấu phẩy (vd: "1,5,12").
- `folder_id` (optional): Lọc tài liệu trong một thư mục cụ thể.
- `page`, `size`: Pagination.

### 7.6 Xem và Tải tài liệu
```http
GET /api/v1/documents/{id}
```
**Roles**: MEMBER, ADMIN, ROOT
**Query Parameters**:
- `download` (optional, boolean): Nếu `true`, trả về file stream để tải xuống. Mặc định là `false` (trả về thông tin chi tiết dạng JSON).

### 7.7 Tạo tài liệu (Upload)
```http
POST /api/v1/documents
```
**Roles**: MEMBER, ADMIN, ROOT
**Content-Type**: multipart/form-data
**Form Parameters**:
- `title` (required, string)
- `description` (optional, string)
- `file` (conditional, MultipartFile)
- `source_type` (optional, string, default: INTERNAL_UPLOAD)
- `source_location` (conditional, string, for EXTERNAL_LINK)
- `is_public` (optional, boolean, default: false)
- `status` (optional, string, default: DRAFT)
- `tags` (optional, string, comma-separated IDs)
- `folder_id` (optional, long)

### 7.8 Cập nhật tài liệu
```http
PATCH /api/v1/documents/{id}
```
**Roles**: MEMBER (chỉ tài liệu của mình), ADMIN, ROOT (mọi tài liệu)
**Content-Type**: multipart/form-data
**Form Parameters**: Tương tự mục 7.7 (các trường đều là tùy chọn).

### 7.9 Xóa tài liệu
```http
DELETE /api/v1/documents/{id}
```
**Roles**: MEMBER (chỉ tài liệu của mình), ADMIN, ROOT (mọi tài liệu)
**Query Parameters**:
- `force` (optional, boolean): Nếu `true`, xóa vĩnh viễn. Mặc định là xóa mềm. (Chỉ ADMIN/ROOT có quyền dùng `force=true`).

---

## 8. Error Handling

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
- **204 No Content**: Thành công nhưng không có nội dung trả về (ví dụ: xóa)
- **400 Bad Request**: Dữ liệu không hợp lệ
- **401 Unauthorized**: Chưa xác thực
- **403 Forbidden**: Không có quyền truy cập
- **404 Not Found**: Không tìm thấy tài nguyên
- **500 Internal Server Error**: Lỗi server

---

## 9. Security & Configuration

### Security
- **CORS**: Được cấu hình cho phép cross-origin requests.
- **CSRF**: Đã được disable do sử dụng JWT.
- **Session**: Stateless (không sử dụng session).
- **OAuth2**: Tích hợp với GitLab OAuth.

### Configuration
- **Database**: PostgreSQL (`postgres:5432`, db: `rtic_db`, user: `rtic`)
- **File Upload**: Max size 10MB.
- **Storage Paths**: `uploads/avatars`, `uploads/documents`, ...
- **JWT**: Access token 2 giờ, Refresh token 7 ngày.