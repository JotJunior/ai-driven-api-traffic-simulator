# API Endpoint Mapping for Traffic Simulator

## Overview

This document provides a comprehensive mapping of all API endpoints in the User Service Domain for traffic simulation purposes. Each endpoint includes detailed information about parameters, authentication requirements, response formats, error scenarios, and realistic usage patterns.

## Base URL
- HTTP: `http://localhost:9501`
- HTTPS: `https://api.fotus.com.br/user-service`

## Authentication

All endpoints (except health checks) require Bearer token authentication:
```
Authorization: Bearer {jwt_token}
```

## 1. Authentication Endpoints

### 1.1 Login
- **Method**: `POST`
- **Path**: `/api/v1/auth/login`
- **Authentication**: None required
- **Content-Type**: `application/json`

**Request Body**:
```json
{
  "email": "user@fotus.com.br",
  "password": "senha123@",
  "client_id": "fotus-client",
  "client_secret": "client-secret-123"
}
```

**Success Response (200)**:
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
  "refresh_token": "def502004a1b2c3d4e5f6789...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Error Scenarios**:
- `401`: Invalid credentials
- `400`: Missing email/password
- `500`: Server error

**Usage Patterns**:
- Frequency: High (daily login sessions)
- Peak times: 8-10 AM, 2-4 PM
- Typical response time: 200-500ms
- Cache TTL: No caching (security)

### 1.2 Logout
- **Method**: `POST`
- **Path**: `/api/v1/auth/logout`
- **Authentication**: Bearer token required

**Headers**:
```
Authorization: Bearer {jwt_token}
```

**Success Response (200)**:
```json
{
  "message": "Logout realizado com sucesso"
}
```

**Error Scenarios**:
- `401`: Invalid/missing token
- `500`: Token revocation failed

**Usage Patterns**:
- Frequency: Medium
- Response time: 100-300ms

### 1.3 Refresh Token
- **Method**: `POST`
- **Path**: `/api/v1/auth/refresh`
- **Authentication**: None required

**Request Body**:
```json
{
  "refresh_token": "def502004a1b2c3d4e5f6789...",
  "client_id": "fotus-client",
  "client_secret": "client-secret-123"
}
```

**Success Response (200)**:
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
  "refresh_token": "def502004a1b2c3d4e5f6789...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Error Scenarios**:
- `401`: Invalid refresh token
- `400`: Missing refresh token
- `500`: Server error

**Usage Patterns**:
- Frequency: High (automatic refresh)
- Response time: 150-400ms

### 1.4 User Info (Me)
- **Method**: `POST`
- **Path**: `/api/v1/auth/me`
- **Authentication**: Bearer token required

**Success Response (200)**:
```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@fotus.com.br",
  "scopes": ["read", "write"],
  "expires_at": "2024-12-31 23:59:59"
}
```

**Error Scenarios**:
- `401`: Invalid/expired token
- `500`: Server error

**Usage Patterns**:
- Frequency: Very high (every API call validation)
- Response time: 50-150ms
- Cache TTL: 5 minutes

## 2. User Management Endpoints

### 2.1 List Users
- **Method**: `GET`
- **Path**: `/api/v1/users`
- **Authentication**: Bearer token required

**Query Parameters**:
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20)
- `search` (string): Search term
- `is_active` (boolean): Filter by active status
- `is_deleted` (boolean): Filter by deleted status

**Example Request**:
```
GET /api/v1/users?page=1&limit=20&search=joao&is_active=true
```

**Success Response (200)**:
```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "joao.silva",
      "email": "joao.silva@fotus.com.br",
      "name": "João Silva Santos",
      "is_active": true,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 150,
    "page": 1,
    "limit": 20,
    "pages": 8
  }
}
```

**Error Scenarios**:
- `401`: Unauthorized
- `400`: Invalid parameters
- `500`: Server error

**Usage Patterns**:
- Frequency: High (admin dashboards)
- Response time: 200-800ms
- Cache TTL: 2 minutes

### 2.2 Search Users
- **Method**: `GET`
- **Path**: `/api/v1/users/search`
- **Authentication**: Bearer token required

**Query Parameters**:
- `q` (string, required): Search term
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20)

**Example Request**:
```
GET /api/v1/users/search?q=joão&page=1&limit=10
```

**Success Response (200)**:
```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "joao.silva",
      "email": "joao.silva@fotus.com.br",
      "name": "João Silva Santos",
      "is_active": true
    }
  ],
  "pagination": {
    "total": 5,
    "page": 1,
    "limit": 10,
    "pages": 1
  }
}
```

**Error Scenarios**:
- `400`: Missing search term
- `401`: Unauthorized
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 300-1000ms (search complexity)
- Cache TTL: 30 seconds

### 2.3 Get User by ID
- **Method**: `GET`
- **Path**: `/api/v1/users/{id}`
- **Authentication**: Bearer token required

**Path Parameters**:
- `id` (string, required): User UUID

**Example Request**:
```
GET /api/v1/users/550e8400-e29b-41d4-a716-446655440000
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@fotus.com.br",
  "name": "João Silva Santos",
  "phone_number": "+5511999887766",
  "document_number": "12345678901",
  "is_active": true,
  "is_deleted": false,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-20T15:45:00Z",
  "roles": [
    {
      "id": "role-id-1",
      "name": "user",
      "display_name": "User"
    }
  ]
}
```

**Error Scenarios**:
- `404`: User not found
- `401`: Unauthorized
- `400`: Invalid UUID format
- `500`: Server error

**Usage Patterns**:
- Frequency: Very high
- Response time: 100-300ms
- Cache TTL: 5 minutes

### 2.4 Get User by Email
- **Method**: `GET`
- **Path**: `/api/v1/users/email/{email}`
- **Authentication**: Bearer token required

**Path Parameters**:
- `email` (string, required): User email address

**Example Request**:
```
GET /api/v1/users/email/joao.silva@fotus.com.br
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@fotus.com.br",
  "name": "João Silva Santos",
  "is_active": true
}
```

**Error Scenarios**:
- `404`: User not found
- `401`: Unauthorized
- `400`: Invalid email format
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 150-400ms
- Cache TTL: 10 minutes

### 2.5 Get User by Username
- **Method**: `GET`
- **Path**: `/api/v1/users/username/{username}`
- **Authentication**: Bearer token required

**Path Parameters**:
- `username` (string, required): Username

**Example Request**:
```
GET /api/v1/users/username/joao.silva
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@fotus.com.br",
  "name": "João Silva Santos",
  "is_active": true
}
```

**Error Scenarios**:
- `404`: User not found
- `401`: Unauthorized
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 150-400ms
- Cache TTL: 10 minutes

### 2.6 Create User
- **Method**: `POST`
- **Path**: `/api/v1/users`
- **Authentication**: Bearer token required
- **Content-Type**: `application/json`

**Request Body**:
```json
{
  "username": "joao.silva",
  "email": "joao.silva@fotus.com.br",
  "password": "senha123@",
  "name": "João Silva Santos",
  "phone_number": "+5511999887766",
  "document_number": "12345678901"
}
```

**Success Response (201)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@fotus.com.br",
  "name": "João Silva Santos",
  "phone_number": "+5511999887766",
  "document_number": "12345678901",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `409`: User already exists (email/username conflict)
- `500`: Server error

**Usage Patterns**:
- Frequency: Low-Medium (user registration)
- Response time: 500-1200ms (password hashing)
- Cache invalidation: User lists, search results

### 2.7 Update User
- **Method**: `PUT`
- **Path**: `/api/v1/users/{id}`
- **Authentication**: Bearer token required
- **Content-Type**: `application/json`

**Path Parameters**:
- `id` (string, required): User UUID

**Request Body** (all fields optional):
```json
{
  "username": "joao.silva.updated",
  "email": "joao.silva.updated@fotus.com.br",
  "name": "João Silva Santos Updated",
  "phone_number": "+5511999887777",
  "document_number": "12345678902"
}
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva.updated",
  "email": "joao.silva.updated@fotus.com.br",
  "name": "João Silva Santos Updated",
  "phone_number": "+5511999887777",
  "document_number": "12345678902",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `404`: User not found
- `409`: Email/username conflict
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 300-700ms
- Cache invalidation: User details, lists

### 2.8 Delete User (Soft Delete)
- **Method**: `DELETE`
- **Path**: `/api/v1/users/{id}`
- **Authentication**: Bearer token required

**Path Parameters**:
- `id` (string, required): User UUID

**Success Response (200)**:
```json
{
  "message": "Usuário removido com sucesso"
}
```

**Error Scenarios**:
- `404`: User not found
- `401`: Unauthorized
- `409`: Cannot delete user with active sessions
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 200-500ms
- Cache invalidation: User lists, search results

### 2.9 Activate User
- **Method**: `PUT`
- **Path**: `/api/v1/users/{id}/activate`
- **Authentication**: Bearer token required

**Path Parameters**:
- `id` (string, required): User UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@fotus.com.br",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: User not found
- `401`: Unauthorized
- `400`: User already active
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 200-400ms
- Cache invalidation: User details

### 2.10 Deactivate User
- **Method**: `PUT`
- **Path**: `/api/v1/users/{id}/deactivate`
- **Authentication**: Bearer token required

**Path Parameters**:
- `id` (string, required): User UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@fotus.com.br",
  "is_active": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: User not found
- `401`: Unauthorized
- `400`: User already inactive
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 200-400ms
- Cache invalidation: User details

### 2.11 Change Password
- **Method**: `PUT`
- **Path**: `/api/v1/users/{id}/change-password`
- **Authentication**: Bearer token required
- **Content-Type**: `application/json`

**Path Parameters**:
- `id` (string, required): User UUID

**Request Body**:
```json
{
  "oldPassword": "senha123@",
  "newPassword": "novaSenha456#"
}
```

**Success Response (200)**:
```json
{
  "message": "Senha alterada com sucesso"
}
```

**Error Scenarios**:
- `400`: Invalid current password
- `401`: Unauthorized
- `404`: User not found
- `422`: Password validation failed
- `500`: Server error

**Usage Patterns**:
- Frequency: Low-Medium
- Response time: 800-1500ms (password hashing)
- Security: Rate limiting recommended

### 2.12 Reset Password (Admin)
- **Method**: `PUT`
- **Path**: `/api/v1/users/{id}/reset-password`
- **Authentication**: Bearer token required (Admin role)
- **Content-Type**: `application/json`

**Path Parameters**:
- `id` (string, required): User UUID

**Request Body**:
```json
{
  "newPassword": "novaSenha456#",
  "resetToken": "admin-reset"
}
```

**Success Response (200)**:
```json
{
  "message": "Senha resetada com sucesso"
}
```

**Error Scenarios**:
- `400`: Invalid new password
- `401`: Unauthorized
- `403`: Insufficient privileges
- `404`: User not found
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 800-1500ms
- Security: Admin-only access

## 3. User Profile Management Endpoints

### 3.1 Create User Profile
- **Method**: `POST`
- **Path**: `/api/v1/users/{userId}/profile`
- **Authentication**: Bearer token required
- **Content-Type**: `application/json`

**Path Parameters**:
- `userId` (string, required): User UUID

**Request Body**:
```json
{
  "avatar": "https://example.com/avatar.jpg",
  "bio": "Desenvolvedor apaixonado por tecnologia",
  "website": "https://johndoe.com",
  "location": "São Paulo, Brasil",
  "company": "Fotus Distribuidora Solar",
  "job_title": "Desenvolvedor Senior",
  "birth_date": "1990-05-15",
  "gender": "male",
  "timezone": "America/Sao_Paulo",
  "language": "pt_BR",
  "is_public": true
}
```

**Success Response (201)**:
```json
{
  "id": "profile-id-123",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "avatar": "https://example.com/avatar.jpg",
  "bio": "Desenvolvedor apaixonado por tecnologia",
  "website": "https://johndoe.com",
  "location": "São Paulo, Brasil",
  "company": "Fotus Distribuidora Solar",
  "job_title": "Desenvolvedor Senior",
  "birth_date": "1990-05-15",
  "gender": "male",
  "timezone": "America/Sao_Paulo",
  "language": "pt_BR",
  "is_public": true,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `404`: User not found
- `409`: Profile already exists for user
- `500`: Server error

**Usage Patterns**:
- Frequency: Low (one-time creation)
- Response time: 300-600ms
- Cache TTL: 30 minutes

### 3.2 Get User Profile
- **Method**: `GET`
- **Path**: `/api/v1/users/{userId}/profile`
- **Authentication**: Bearer token required

**Path Parameters**:
- `userId` (string, required): User UUID

**Success Response (200)**:
```json
{
  "id": "profile-id-123",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "avatar": "https://example.com/avatar.jpg",
  "bio": "Desenvolvedor apaixonado por tecnologia",
  "website": "https://johndoe.com",
  "location": "São Paulo, Brasil",
  "company": "Fotus Distribuidora Solar",
  "job_title": "Desenvolvedor Senior",
  "birth_date": "1990-05-15",
  "gender": "male",
  "timezone": "America/Sao_Paulo",
  "language": "pt_BR",
  "is_public": true,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: Profile not found
- `401`: Unauthorized
- `403`: Profile is private and user has no access
- `500`: Server error

**Usage Patterns**:
- Frequency: High
- Response time: 100-300ms
- Cache TTL: 30 minutes

### 3.3 Update User Profile
- **Method**: `PUT`
- **Path**: `/api/v1/users/{userId}/profile`
- **Authentication**: Bearer token required
- **Content-Type**: `application/json`

**Path Parameters**:
- `userId` (string, required): User UUID

**Request Body** (all fields optional):
```json
{
  "avatar": "https://example.com/new-avatar.jpg",
  "bio": "Updated bio",
  "location": "Rio de Janeiro, Brasil",
  "is_public": false
}
```

**Success Response (200)**:
```json
{
  "id": "profile-id-123",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "avatar": "https://example.com/new-avatar.jpg",
  "bio": "Updated bio",
  "location": "Rio de Janeiro, Brasil",
  "is_public": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `404`: Profile not found
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 200-500ms
- Cache invalidation: Profile data

### 3.4 Delete User Profile
- **Method**: `DELETE`
- **Path**: `/api/v1/users/{userId}/profile`
- **Authentication**: Bearer token required

**Path Parameters**:
- `userId` (string, required): User UUID

**Success Response (204)**:
```
No content
```

**Error Scenarios**:
- `404`: Profile not found
- `401`: Unauthorized
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 200-400ms
- Cache invalidation: Profile data

## 4. Role Management Endpoints

### 4.1 Create Role
- **Method**: `POST`
- **Path**: `/api/v1/roles`
- **Authentication**: Bearer token required (Admin)
- **Content-Type**: `application/json`

**Request Body**:
```json
{
  "name": "admin",
  "display_name": "Administrador",
  "description": "Role com acesso total ao sistema"
}
```

**Success Response (201)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "admin",
  "display_name": "Administrador",
  "description": "Role com acesso total ao sistema",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `403`: Insufficient privileges
- `409`: Role already exists
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low (admin setup)
- Response time: 200-400ms
- Cache invalidation: Role lists

### 4.2 List Roles
- **Method**: `GET`
- **Path**: `/api/v1/roles`
- **Authentication**: Bearer token required

**Query Parameters**:
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20)
- `search` (string): Search term
- `is_active` (boolean): Filter by active status

**Example Request**:
```
GET /api/v1/roles?page=1&limit=10&is_active=true
```

**Success Response (200)**:
```json
{
  "items": [
    {
      "id": "role-id-1",
      "name": "admin",
      "display_name": "Administrador",
      "description": "Role com acesso total",
      "is_active": true,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 5,
    "page": 1,
    "limit": 10,
    "pages": 1
  }
}
```

**Error Scenarios**:
- `401`: Unauthorized
- `400`: Invalid parameters
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 150-400ms
- Cache TTL: 15 minutes

### 4.3 Get Role by ID
- **Method**: `GET`
- **Path**: `/api/v1/roles/{id}`
- **Authentication**: Bearer token required

**Path Parameters**:
- `id` (string, required): Role UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "admin",
  "display_name": "Administrador",
  "description": "Role com acesso total ao sistema",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: Role not found
- `401`: Unauthorized
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 100-250ms
- Cache TTL: 30 minutes

### 4.4 Update Role
- **Method**: `PUT`
- **Path**: `/api/v1/roles/{id}`
- **Authentication**: Bearer token required (Admin)
- **Content-Type**: `application/json`

**Path Parameters**:
- `id` (string, required): Role UUID

**Request Body** (all fields optional):
```json
{
  "name": "super-admin",
  "display_name": "Super Administrador",
  "description": "Role com acesso completo e privilégios especiais"
}
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "super-admin",
  "display_name": "Super Administrador",
  "description": "Role com acesso completo e privilégios especiais",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `403`: Insufficient privileges
- `404`: Role not found
- `409`: Role name conflict
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 200-400ms
- Cache invalidation: Role data

### 4.5 Delete Role
- **Method**: `DELETE`
- **Path**: `/api/v1/roles/{id}`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `id` (string, required): Role UUID

**Success Response (204)**:
```
No content
```

**Error Scenarios**:
- `404`: Role not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `409`: Role has dependencies (assigned to users)
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 200-500ms
- Cache invalidation: Role lists

### 4.6 Activate Role
- **Method**: `PUT`
- **Path**: `/api/v1/roles/{id}/activate`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `id` (string, required): Role UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "admin",
  "display_name": "Administrador",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: Role not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `400`: Role already active
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 150-300ms
- Cache invalidation: Role data

### 4.7 Deactivate Role
- **Method**: `PUT`
- **Path**: `/api/v1/roles/{id}/deactivate`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `id` (string, required): Role UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "admin",
  "display_name": "Administrador",
  "is_active": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: Role not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `400`: Role already inactive
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 150-300ms
- Cache invalidation: Role data

## 5. Permission Management Endpoints

### 5.1 Create Permission
- **Method**: `POST`
- **Path**: `/api/v1/permissions`
- **Authentication**: Bearer token required (Admin)
- **Content-Type**: `application/json`

**Request Body**:
```json
{
  "name": "users.create",
  "display_name": "Criar Usuários",
  "resource": "users",
  "action": "create",
  "description": "Permite criar novos usuários no sistema"
}
```

**Success Response (201)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "users.create",
  "display_name": "Criar Usuários",
  "resource": "users",
  "action": "create",
  "description": "Permite criar novos usuários no sistema",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `403`: Insufficient privileges
- `409`: Permission already exists
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low (system setup)
- Response time: 200-400ms
- Cache invalidation: Permission lists

### 5.2 List Permissions
- **Method**: `GET`
- **Path**: `/api/v1/permissions`
- **Authentication**: Bearer token required

**Query Parameters**:
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20)
- `search` (string): Search term
- `is_active` (boolean): Filter by active status
- `resource` (string): Filter by resource type

**Example Request**:
```
GET /api/v1/permissions?page=1&limit=10&resource=users&is_active=true
```

**Success Response (200)**:
```json
{
  "items": [
    {
      "id": "perm-id-1",
      "name": "users.create",
      "display_name": "Criar Usuários",
      "resource": "users",
      "action": "create",
      "is_active": true
    }
  ],
  "pagination": {
    "total": 25,
    "page": 1,
    "limit": 10,
    "pages": 3
  }
}
```

**Error Scenarios**:
- `401`: Unauthorized
- `400`: Invalid parameters
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 150-400ms
- Cache TTL: 30 minutes

### 5.3 Get Permission by ID
- **Method**: `GET`
- **Path**: `/api/v1/permissions/{id}`
- **Authentication**: Bearer token required

**Path Parameters**:
- `id` (string, required): Permission UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "users.create",
  "display_name": "Criar Usuários",
  "resource": "users",
  "action": "create",
  "description": "Permite criar novos usuários no sistema",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: Permission not found
- `401`: Unauthorized
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 100-250ms
- Cache TTL: 30 minutes

### 5.4 Update Permission
- **Method**: `PUT`
- **Path**: `/api/v1/permissions/{id}`
- **Authentication**: Bearer token required (Admin)
- **Content-Type**: `application/json`

**Path Parameters**:
- `id` (string, required): Permission UUID

**Request Body** (all fields optional):
```json
{
  "name": "users.create.advanced",
  "display_name": "Criar Usuários Avançado",
  "description": "Permite criar usuários com permissões especiais"
}
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "users.create.advanced",
  "display_name": "Criar Usuários Avançado",
  "description": "Permite criar usuários com permissões especiais",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid data/validation errors
- `401`: Unauthorized
- `403`: Insufficient privileges
- `404`: Permission not found
- `409`: Permission name conflict
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 200-400ms
- Cache invalidation: Permission data

### 5.5 Delete Permission
- **Method**: `DELETE`
- **Path**: `/api/v1/permissions/{id}`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `id` (string, required): Permission UUID

**Success Response (200)**:
```json
{
  "message": "Permissão removida com sucesso"
}
```

**Error Scenarios**:
- `404`: Permission not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `409`: Permission has dependencies
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 200-500ms
- Cache invalidation: Permission lists

### 5.6 Activate Permission
- **Method**: `PUT`
- **Path**: `/api/v1/permissions/{id}/activate`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `id` (string, required): Permission UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "users.create",
  "display_name": "Criar Usuários",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: Permission not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `400`: Permission already active
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 150-300ms
- Cache invalidation: Permission data

### 5.7 Deactivate Permission
- **Method**: `PUT`
- **Path**: `/api/v1/permissions/{id}/deactivate`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `id` (string, required): Permission UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "users.create",
  "display_name": "Criar Usuários",
  "is_active": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: Permission not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `400`: Permission already inactive
- `500`: Server error

**Usage Patterns**:
- Frequency: Very low
- Response time: 150-300ms
- Cache invalidation: Permission data

## 6. User Role Management Endpoints

### 6.1 List User Roles
- **Method**: `GET`
- **Path**: `/api/v1/users/{userId}/roles`
- **Authentication**: Bearer token required

**Path Parameters**:
- `userId` (string, required): User UUID

**Success Response (200)**:
```json
{
  "items": [
    {
      "id": "role-id-1",
      "name": "admin",
      "display_name": "Administrador",
      "description": "Role com acesso total"
    },
    {
      "id": "role-id-2",
      "name": "user",
      "display_name": "Usuário",
      "description": "Role básica de usuário"
    }
  ],
  "pagination": {
    "total": 2,
    "page": 1,
    "limit": 2,
    "pages": 1
  }
}
```

**Error Scenarios**:
- `404`: User not found
- `401`: Unauthorized
- `500`: Server error

**Usage Patterns**:
- Frequency: Medium
- Response time: 150-350ms
- Cache TTL: 10 minutes

### 6.2 Assign Role to User
- **Method**: `POST`
- **Path**: `/api/v1/users/{userId}/roles/{roleId}`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `userId` (string, required): User UUID
- `roleId` (string, required): Role UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@fotus.com.br",
  "roles": [
    {
      "id": "role-id-1",
      "name": "admin",
      "display_name": "Administrador"
    }
  ],
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: User or role not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `409`: Role already assigned to user
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 200-500ms
- Cache invalidation: User roles, user details

### 6.3 Revoke Role from User
- **Method**: `DELETE`
- **Path**: `/api/v1/users/{userId}/roles/{roleId}`
- **Authentication**: Bearer token required (Admin)

**Path Parameters**:
- `userId` (string, required): User UUID
- `roleId` (string, required): Role UUID

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@fotus.com.br",
  "roles": [],
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `404`: User or role not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `400`: Role not assigned to user
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 200-500ms
- Cache invalidation: User roles, user details

### 6.4 Bulk Assign Roles
- **Method**: `POST`
- **Path**: `/api/v1/users/{userId}/roles/bulk-assign`
- **Authentication**: Bearer token required (Admin)
- **Content-Type**: `application/json`

**Path Parameters**:
- `userId` (string, required): User UUID

**Request Body**:
```json
{
  "roleIds": ["role-id-1", "role-id-2", "role-id-3"]
}
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@fotus.com.br",
  "roles": [
    {
      "id": "role-id-1",
      "name": "admin",
      "display_name": "Administrador"
    },
    {
      "id": "role-id-2",
      "name": "user",
      "display_name": "Usuário"
    },
    {
      "id": "role-id-3",
      "name": "manager",
      "display_name": "Gerente"
    }
  ],
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid role IDs or empty array
- `404`: User not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 300-800ms (multiple operations)
- Cache invalidation: User roles, user details

### 6.5 Bulk Revoke Roles
- **Method**: `POST`
- **Path**: `/api/v1/users/{userId}/roles/bulk-revoke`
- **Authentication**: Bearer token required (Admin)
- **Content-Type**: `application/json`

**Path Parameters**:
- `userId` (string, required): User UUID

**Request Body**:
```json
{
  "roleIds": ["role-id-1", "role-id-2"]
}
```

**Success Response (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@fotus.com.br",
  "roles": [
    {
      "id": "role-id-3",
      "name": "manager",
      "display_name": "Gerente"
    }
  ],
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `400`: Invalid role IDs or empty array
- `404`: User not found
- `401`: Unauthorized
- `403`: Insufficient privileges
- `500`: Server error

**Usage Patterns**:
- Frequency: Low
- Response time: 300-800ms (multiple operations)
- Cache invalidation: User roles, user details

## 7. System Endpoints

### 7.1 Health Check (Simple)
- **Method**: `GET`
- **Path**: `/`
- **Authentication**: None required

**Success Response (200)**:
```json
{
  "status": "ok",
  "service": "user-svc-domain",
  "timestamp": "2024-01-20T15:45:00Z"
}
```

**Error Scenarios**:
- `500`: Service unavailable

**Usage Patterns**:
- Frequency: Very high (load balancer health checks)
- Response time: 10-50ms
- Cache: No caching

### 7.2 Health Check (Detailed)
- **Method**: `GET`
- **Path**: `/health`
- **Authentication**: None required

**Success Response (200)**:
```json
{
  "status": "healthy",
  "checks": {
    "database": "ok",
    "cache": "ok"
  },
  "timestamp": "2024-01-20T15:45:00Z",
  "uptime": 3600,
  "version": "1.0.0"
}
```

**Error Scenarios**:
- `503`: Service degraded (some checks failing)
- `500`: Service unavailable

**Usage Patterns**:
- Frequency: High (monitoring systems)
- Response time: 20-100ms
- Cache: No caching

### 7.3 Metrics
- **Method**: `GET`
- **Path**: `/metrics`
- **Authentication**: None required (internal use)
- **Content-Type**: `text/plain`

**Success Response (200)**:
```
# HELP http_requests_total The total number of HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",status="200"} 1024
http_requests_total{method="POST",status="201"} 256
http_requests_total{method="POST",status="400"} 12

# HELP http_request_duration_seconds The HTTP request latency
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.1"} 789
http_request_duration_seconds_bucket{le="0.5"} 1200
http_request_duration_seconds_bucket{le="1.0"} 1350
http_request_duration_seconds_bucket{le="+Inf"} 1400

# HELP user_service_users_total Total number of users in the system
# TYPE user_service_users_total gauge
user_service_users_total 1500

# HELP user_service_active_sessions_total Active user sessions
# TYPE user_service_active_sessions_total gauge
user_service_active_sessions_total 45
```

**Error Scenarios**:
- `500`: Metrics collection failed

**Usage Patterns**:
- Frequency: High (Prometheus scraping every 30s)
- Response time: 50-200ms
- Cache: No caching (real-time metrics)

## 8. Traffic Simulation Recommendations

### 8.1 Realistic Usage Patterns

**Peak Hours**:
- Business hours: 8 AM - 6 PM (Brazil timezone)
- Lunch break spike: 12 PM - 2 PM
- Weekend: 50% reduced traffic

**Endpoint Frequency Distribution**:
- Authentication (`/auth/*`): 20% of total requests
- User lookup (`GET /users/*`): 40% of total requests
- User modifications (`POST/PUT /users/*`): 15% of total requests
- Profile operations: 10% of total requests
- Role/Permission operations: 5% of total requests
- System endpoints: 10% of total requests

### 8.2 Error Simulation Scenarios

**Network Errors**:
- Connection timeouts: 2% of requests
- Network latency spikes: 5% of requests
- Connection drops: 1% of requests

**Application Errors**:
- Authentication failures: 8% of login attempts
- Validation errors: 12% of creation/update requests
- Not found errors: 15% of lookup requests
- Permission denied: 5% of protected resource access

**Server Errors**:
- Database connection issues: 0.5% of requests
- Cache unavailable: 1% of requests
- Service overload: 0.2% of requests during peak times

### 8.3 Load Testing Scenarios

**Scenario 1: Normal Business Load**
- 100 concurrent users
- 10 requests per user per minute
- Mixed endpoint usage (following frequency distribution)
- 95% success rate expected

**Scenario 2: Peak Load**
- 500 concurrent users
- 20 requests per user per minute
- Heavy focus on authentication and user lookup
- 90% success rate expected

**Scenario 3: Stress Test**
- 1000+ concurrent users
- 50 requests per user per minute
- Monitor system degradation gracefully
- Acceptable degraded performance thresholds

### 8.4 Cache Simulation

**Cache Hit Ratios** (target values):
- User lookups: 80%
- Role/Permission data: 95%
- Profile data: 70%
- Authentication data: 60%

**Cache Invalidation Triggers**:
- User updates: Clear user-specific caches
- Role changes: Clear role-related caches
- Permission changes: Clear permission caches
- System-wide clear: Every 6 hours

### 8.5 Security Testing

**Rate Limiting**:
- Login attempts: 5 per minute per IP
- Password changes: 3 per hour per user
- API calls: 100 per minute per authenticated user

**Authentication Testing**:
- Valid token scenarios: 85%
- Expired token scenarios: 10%
- Invalid token scenarios: 5%

### 8.6 Data Consistency Testing

**Concurrent Operations**:
- Multiple users updating same resource
- Role assignments during user updates
- Cache consistency during high-frequency updates
- Transaction rollback scenarios

This comprehensive API endpoint mapping provides all the necessary information for implementing a robust traffic simulator that can generate realistic load patterns, test various error scenarios, and validate the system's performance under different conditions.