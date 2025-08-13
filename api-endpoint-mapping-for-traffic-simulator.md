# Mapeamento de Endpoints da API para Simulador de Tráfego

## Visão Geral

Este documento fornece um mapeamento abrangente de todos os endpoints da API no Domínio do Serviço de Usuários para fins de simulação de tráfego. Cada endpoint inclui informações detalhadas sobre parâmetros, requisitos de autenticação, formatos de resposta, cenários de erro e padrões de uso realistas.

## URL Base
- HTTP: `http://localhost:9501`
- HTTPS: `https://api.empresa.com.br/user-service`

## Autenticação

Todos os endpoints (exceto verificações de saúde) requerem autenticação por Bearer token:
```
Authorization: Bearer {jwt_token}
```

## 1. Endpoints de Autenticação

### 1.1 Login
- **Método**: `POST`
- **Caminho**: `/api/v1/auth/login`
- **Autenticação**: Não requerida
- **Content-Type**: `application/json`

**Corpo da Requisição**:
```json
{
  "email": "user@empresa.com.br",
  "password": "senha123@",
  "client_id": "empresa-client",
  "client_secret": "client-secret-123"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
  "refresh_token": "def502004a1b2c3d4e5f6789...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Cenários de Erro**:
- `401`: Credenciais inválidas
- `400`: Email/senha ausentes
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Alta (sessões de login diário)
- Horários de pico: 8-10h, 14-16h
- Tempo de resposta típico: 200-500ms
- TTL de Cache: Sem cache (segurança)

### 1.2 Logout
- **Método**: `POST`
- **Caminho**: `/api/v1/auth/logout`
- **Autenticação**: Bearer token requerido

**Headers**:
```
Authorization: Bearer {jwt_token}
```

**Resposta de Sucesso (200)**:
```json
{
  "message": "Logout realizado com sucesso"
}
```

**Cenários de Erro**:
- `401`: Token inválido/ausente
- `500`: Falha na revogação do token

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 100-300ms

### 1.3 Refresh Token
- **Método**: `POST`
- **Caminho**: `/api/v1/auth/refresh`
- **Autenticação**: Não requerida

**Corpo da Requisição**:
```json
{
  "refresh_token": "def502004a1b2c3d4e5f6789...",
  "client_id": "empresa-client",
  "client_secret": "client-secret-123"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
  "refresh_token": "def502004a1b2c3d4e5f6789...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Cenários de Erro**:
- `401`: Refresh token inválido
- `400`: Refresh token ausente
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Alta (refresh automático)
- Tempo de resposta: 150-400ms

### 1.4 Informações do Usuário (Me)
- **Método**: `POST`
- **Caminho**: `/api/v1/auth/me`
- **Autenticação**: Bearer token requerido

**Resposta de Sucesso (200)**:
```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@empresa.com.br",
  "scopes": ["read", "write"],
  "expires_at": "2024-12-31 23:59:59"
}
```

**Cenários de Erro**:
- `401`: Token inválido/expirado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito alta (validação a cada chamada da API)
- Tempo de resposta: 50-150ms
- TTL de Cache: 5 minutos

## 2. Endpoints de Gerenciamento de Usuários

### 2.1 Listar Usuários
- **Método**: `GET`
- **Caminho**: `/api/v1/users`
- **Autenticação**: Bearer token requerido

**Parâmetros de Query**:
- `page` (inteiro): Número da página (padrão: 1)
- `limit` (inteiro): Itens por página (padrão: 20)
- `search` (string): Termo de busca
- `is_active` (boolean): Filtrar por status ativo
- `is_deleted` (boolean): Filtrar por status excluído

**Exemplo de Requisição**:
```
GET /api/v1/users?page=1&limit=20&search=joao&is_active=true
```

**Resposta de Sucesso (200)**:
```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "joao.silva",
      "email": "joao.silva@empresa.com.br",
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

**Cenários de Erro**:
- `401`: Não autorizado
- `400`: Parâmetros inválidos
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Alta (painéis administrativos)
- Tempo de resposta: 200-800ms
- TTL de Cache: 2 minutos

### 2.2 Buscar Usuários
- **Método**: `GET`
- **Caminho**: `/api/v1/users/search`
- **Autenticação**: Bearer token requerido

**Parâmetros de Query**:
- `q` (string, obrigatório): Termo de busca
- `page` (inteiro): Número da página (padrão: 1)
- `limit` (inteiro): Itens por página (padrão: 20)

**Exemplo de Requisição**:
```
GET /api/v1/users/search?q=joão&page=1&limit=10
```

**Resposta de Sucesso (200)**:
```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "joao.silva",
      "email": "joao.silva@empresa.com.br",
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

**Cenários de Erro**:
- `400`: Termo de busca ausente
- `401`: Não autorizado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 300-1000ms (complexidade da busca)
- TTL de Cache: 30 segundos

### 2.3 Obter Usuário por ID
- **Método**: `GET`
- **Caminho**: `/api/v1/users/{id}`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID do usuário

**Exemplo de Requisição**:
```
GET /api/v1/users/550e8400-e29b-41d4-a716-446655440000
```

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@empresa.com.br",
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
      "display_name": "Usuário"
    }
  ]
}
```

**Cenários de Erro**:
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `400`: Formato UUID inválido
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito alta
- Tempo de resposta: 100-300ms
- TTL de Cache: 5 minutos

### 2.4 Obter Usuário por Email
- **Método**: `GET`
- **Caminho**: `/api/v1/users/email/{email}`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `email` (string, obrigatório): Endereço de email do usuário

**Exemplo de Requisição**:
```
GET /api/v1/users/email/joao.silva@empresa.com.br
```

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@empresa.com.br",
  "name": "João Silva Santos",
  "is_active": true
}
```

**Cenários de Erro**:
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `400`: Formato de email inválido
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 150-400ms
- TTL de Cache: 10 minutos

### 2.5 Obter Usuário por Nome de Usuário
- **Método**: `GET`
- **Caminho**: `/api/v1/users/username/{username}`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `username` (string, obrigatório): Nome de usuário

**Exemplo de Requisição**:
```
GET /api/v1/users/username/joao.silva
```

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@empresa.com.br",
  "name": "João Silva Santos",
  "is_active": true
}
```

**Cenários de Erro**:
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 150-400ms
- TTL de Cache: 10 minutos

### 2.6 Criar Usuário
- **Método**: `POST`
- **Caminho**: `/api/v1/users`
- **Autenticação**: Bearer token requerido
- **Content-Type**: `application/json`

**Corpo da Requisição**:
```json
{
  "username": "joao.silva",
  "email": "joao.silva@empresa.com.br",
  "password": "senha123@",
  "name": "João Silva Santos",
  "phone_number": "+5511999887766",
  "document_number": "12345678901"
}
```

**Resposta de Sucesso (201)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@empresa.com.br",
  "name": "João Silva Santos",
  "phone_number": "+5511999887766",
  "document_number": "12345678901",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `409`: Usuário já existe (conflito email/username)
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa-Média (registro de usuários)
- Tempo de resposta: 500-1200ms (hashing de senha)
- Invalidação de cache: Listas de usuários, resultados de busca

### 2.7 Atualizar Usuário
- **Método**: `PUT`
- **Caminho**: `/api/v1/users/{id}`
- **Autenticação**: Bearer token requerido
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID do usuário

**Corpo da Requisição** (todos os campos opcionais):
```json
{
  "username": "joao.silva.atualizado",
  "email": "joao.silva.atualizado@empresa.com.br",
  "name": "João Silva Santos Atualizado",
  "phone_number": "+5511999887777",
  "document_number": "12345678902"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva.atualizado",
  "email": "joao.silva.atualizado@empresa.com.br",
  "name": "João Silva Santos Atualizado",
  "phone_number": "+5511999887777",
  "document_number": "12345678902",
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `404`: Usuário não encontrado
- `409`: Conflito email/username
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 300-700ms
- Invalidação de cache: Detalhes do usuário, listas

### 2.8 Excluir Usuário (Exclusão Lógica)
- **Método**: `DELETE`
- **Caminho**: `/api/v1/users/{id}`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID do usuário

**Resposta de Sucesso (200)**:
```json
{
  "message": "Usuário removido com sucesso"
}
```

**Cenários de Erro**:
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `409`: Não é possível excluir usuário com sessões ativas
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 200-500ms
- Invalidação de cache: Listas de usuários, resultados de busca

### 2.9 Ativar Usuário
- **Método**: `PUT`
- **Caminho**: `/api/v1/users/{id}/activate`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID do usuário

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@empresa.com.br",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `400`: Usuário já está ativo
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 200-400ms
- Invalidação de cache: Detalhes do usuário

### 2.10 Desativar Usuário
- **Método**: `PUT`
- **Caminho**: `/api/v1/users/{id}/deactivate`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID do usuário

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "joao.silva",
  "email": "joao.silva@empresa.com.br",
  "is_active": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `400`: Usuário já está inativo
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 200-400ms
- Invalidação de cache: Detalhes do usuário

### 2.11 Alterar Senha
- **Método**: `PUT`
- **Caminho**: `/api/v1/users/{id}/change-password`
- **Autenticação**: Bearer token requerido
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID do usuário

**Corpo da Requisição**:
```json
{
  "oldPassword": "senha123@",
  "newPassword": "novaSenha456#"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "message": "Senha alterada com sucesso"
}
```

**Cenários de Erro**:
- `400`: Senha atual inválida
- `401`: Não autorizado
- `404`: Usuário não encontrado
- `422`: Validação da senha falhou
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa-Média
- Tempo de resposta: 800-1500ms (hashing de senha)
- Segurança: Rate limiting recomendado

### 2.12 Resetar Senha (Admin)
- **Método**: `PUT`
- **Caminho**: `/api/v1/users/{id}/reset-password`
- **Autenticação**: Bearer token requerido (Role Admin)
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID do usuário

**Corpo da Requisição**:
```json
{
  "newPassword": "novaSenha456#",
  "resetToken": "admin-reset"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "message": "Senha resetada com sucesso"
}
```

**Cenários de Erro**:
- `400`: Nova senha inválida
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `404`: Usuário não encontrado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 800-1500ms
- Segurança: Acesso somente para admin

## 3. Endpoints de Gerenciamento de Perfil de Usuário

### 3.1 Criar Perfil de Usuário
- **Método**: `POST`
- **Caminho**: `/api/v1/users/{userId}/profile`
- **Autenticação**: Bearer token requerido
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário

**Corpo da Requisição**:
```json
{
  "avatar": "https://example.com/avatar.jpg",
  "bio": "Desenvolvedor apaixonado por tecnologia",
  "website": "https://johndoe.com",
  "location": "São Paulo, Brasil",
  "company": "Empresa Distribuidora Solar",
  "job_title": "Desenvolvedor Senior",
  "birth_date": "1990-05-15",
  "gender": "male",
  "timezone": "America/Sao_Paulo",
  "language": "pt_BR",
  "is_public": true
}
```

**Resposta de Sucesso (201)**:
```json
{
  "id": "profile-id-123",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "avatar": "https://example.com/avatar.jpg",
  "bio": "Desenvolvedor apaixonado por tecnologia",
  "website": "https://johndoe.com",
  "location": "São Paulo, Brasil",
  "company": "Empresa Distribuidora Solar",
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

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `404`: Usuário não encontrado
- `409`: Perfil já existe para o usuário
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa (criação única)
- Tempo de resposta: 300-600ms
- TTL de Cache: 30 minutos

### 3.2 Obter Perfil de Usuário
- **Método**: `GET`
- **Caminho**: `/api/v1/users/{userId}/profile`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário

**Resposta de Sucesso (200)**:
```json
{
  "id": "profile-id-123",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "avatar": "https://example.com/avatar.jpg",
  "bio": "Desenvolvedor apaixonado por tecnologia",
  "website": "https://johndoe.com",
  "location": "São Paulo, Brasil",
  "company": "Empresa Distribuidora Solar",
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

**Cenários de Erro**:
- `404`: Perfil não encontrado
- `401`: Não autorizado
- `403`: Perfil é privado e usuário não tem acesso
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Alta
- Tempo de resposta: 100-300ms
- TTL de Cache: 30 minutos

### 3.3 Atualizar Perfil de Usuário
- **Método**: `PUT`
- **Caminho**: `/api/v1/users/{userId}/profile`
- **Autenticação**: Bearer token requerido
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário

**Corpo da Requisição** (todos os campos opcionais):
```json
{
  "avatar": "https://example.com/new-avatar.jpg",
  "bio": "Bio atualizada",
  "location": "Rio de Janeiro, Brasil",
  "is_public": false
}
```

**Resposta de Sucesso (200)**:
```json
{
  "id": "profile-id-123",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "avatar": "https://example.com/new-avatar.jpg",
  "bio": "Bio atualizada",
  "location": "Rio de Janeiro, Brasil",
  "is_public": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `404`: Perfil não encontrado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 200-500ms
- Invalidação de cache: Dados do perfil

### 3.4 Excluir Perfil de Usuário
- **Método**: `DELETE`
- **Caminho**: `/api/v1/users/{userId}/profile`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário

**Resposta de Sucesso (204)**:
```
Sem conteúdo
```

**Cenários de Erro**:
- `404`: Perfil não encontrado
- `401`: Não autorizado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 200-400ms
- Invalidação de cache: Dados do perfil

## 4. Endpoints de Gerenciamento de Roles

### 4.1 Criar Role
- **Método**: `POST`
- **Caminho**: `/api/v1/roles`
- **Autenticação**: Bearer token requerido (Admin)
- **Content-Type**: `application/json`

**Corpo da Requisição**:
```json
{
  "name": "admin",
  "display_name": "Administrador",
  "description": "Role com acesso total ao sistema"
}
```

**Resposta de Sucesso (201)**:
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

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `409`: Role já existe
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa (configuração admin)
- Tempo de resposta: 200-400ms
- Invalidação de cache: Listas de roles

### 4.2 Listar Roles
- **Método**: `GET`
- **Caminho**: `/api/v1/roles`
- **Autenticação**: Bearer token requerido

**Parâmetros de Query**:
- `page` (inteiro): Número da página (padrão: 1)
- `limit` (inteiro): Itens por página (padrão: 20)
- `search` (string): Termo de busca
- `is_active` (boolean): Filtrar por status ativo

**Exemplo de Requisição**:
```
GET /api/v1/roles?page=1&limit=10&is_active=true
```

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `401`: Não autorizado
- `400`: Parâmetros inválidos
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 150-400ms
- TTL de Cache: 15 minutos

### 4.3 Obter Role por ID
- **Método**: `GET`
- **Caminho**: `/api/v1/roles/{id}`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da role

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `404`: Role não encontrada
- `401`: Não autorizado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 100-250ms
- TTL de Cache: 30 minutos

### 4.4 Atualizar Role
- **Método**: `PUT`
- **Caminho**: `/api/v1/roles/{id}`
- **Autenticação**: Bearer token requerido (Admin)
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da role

**Corpo da Requisição** (todos os campos opcionais):
```json
{
  "name": "super-admin",
  "display_name": "Super Administrador",
  "description": "Role com acesso completo e privilégios especiais"
}
```

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `404`: Role não encontrada
- `409`: Conflito de nome da role
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 200-400ms
- Invalidação de cache: Dados da role

### 4.5 Excluir Role
- **Método**: `DELETE`
- **Caminho**: `/api/v1/roles/{id}`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da role

**Resposta de Sucesso (204)**:
```
Sem conteúdo
```

**Cenários de Erro**:
- `404`: Role não encontrada
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `409`: Role possui dependências (atribuída a usuários)
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 200-500ms
- Invalidação de cache: Listas de roles

### 4.6 Ativar Role
- **Método**: `PUT`
- **Caminho**: `/api/v1/roles/{id}/activate`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da role

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "admin",
  "display_name": "Administrador",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `404`: Role não encontrada
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `400`: Role já está ativa
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 150-300ms
- Invalidação de cache: Dados da role

### 4.7 Desativar Role
- **Método**: `PUT`
- **Caminho**: `/api/v1/roles/{id}/deactivate`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da role

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "admin",
  "display_name": "Administrador",
  "is_active": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `404`: Role não encontrada
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `400`: Role já está inativa
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 150-300ms
- Invalidação de cache: Dados da role

## 5. Endpoints de Gerenciamento de Permissões

### 5.1 Criar Permissão
- **Método**: `POST`
- **Caminho**: `/api/v1/permissions`
- **Autenticação**: Bearer token requerido (Admin)
- **Content-Type**: `application/json`

**Corpo da Requisição**:
```json
{
  "name": "users.create",
  "display_name": "Criar Usuários",
  "resource": "users",
  "action": "create",
  "description": "Permite criar novos usuários no sistema"
}
```

**Resposta de Sucesso (201)**:
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

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `409`: Permissão já existe
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa (configuração do sistema)
- Tempo de resposta: 200-400ms
- Invalidação de cache: Listas de permissões

### 5.2 Listar Permissões
- **Método**: `GET`
- **Caminho**: `/api/v1/permissions`
- **Autenticação**: Bearer token requerido

**Parâmetros de Query**:
- `page` (inteiro): Número da página (padrão: 1)
- `limit` (inteiro): Itens por página (padrão: 20)
- `search` (string): Termo de busca
- `is_active` (boolean): Filtrar por status ativo
- `resource` (string): Filtrar por tipo de recurso

**Exemplo de Requisição**:
```
GET /api/v1/permissions?page=1&limit=10&resource=users&is_active=true
```

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `401`: Não autorizado
- `400`: Parâmetros inválidos
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 150-400ms
- TTL de Cache: 30 minutos

### 5.3 Obter Permissão por ID
- **Método**: `GET`
- **Caminho**: `/api/v1/permissions/{id}`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da permissão

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `404`: Permissão não encontrada
- `401`: Não autorizado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 100-250ms
- TTL de Cache: 30 minutos

### 5.4 Atualizar Permissão
- **Método**: `PUT`
- **Caminho**: `/api/v1/permissions/{id}`
- **Autenticação**: Bearer token requerido (Admin)
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da permissão

**Corpo da Requisição** (todos os campos opcionais):
```json
{
  "name": "users.create.advanced",
  "display_name": "Criar Usuários Avançado",
  "description": "Permite criar usuários com permissões especiais"
}
```

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `400`: Dados inválidos/erros de validação
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `404`: Permissão não encontrada
- `409`: Conflito de nome da permissão
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 200-400ms
- Invalidação de cache: Dados da permissão

### 5.5 Excluir Permissão
- **Método**: `DELETE`
- **Caminho**: `/api/v1/permissions/{id}`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da permissão

**Resposta de Sucesso (200)**:
```json
{
  "message": "Permissão removida com sucesso"
}
```

**Cenários de Erro**:
- `404`: Permissão não encontrada
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `409`: Permissão possui dependências
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 200-500ms
- Invalidação de cache: Listas de permissões

### 5.6 Ativar Permissão
- **Método**: `PUT`
- **Caminho**: `/api/v1/permissions/{id}/activate`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da permissão

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "users.create",
  "display_name": "Criar Usuários",
  "is_active": true,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `404`: Permissão não encontrada
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `400`: Permissão já está ativa
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 150-300ms
- Invalidação de cache: Dados da permissão

### 5.7 Desativar Permissão
- **Método**: `PUT`
- **Caminho**: `/api/v1/permissions/{id}/deactivate`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `id` (string, obrigatório): UUID da permissão

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "users.create",
  "display_name": "Criar Usuários",
  "is_active": false,
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `404`: Permissão não encontrada
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `400`: Permissão já está inativa
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Muito baixa
- Tempo de resposta: 150-300ms
- Invalidação de cache: Dados da permissão

## 6. Endpoints de Gerenciamento de Roles de Usuário

### 6.1 Listar Roles do Usuário
- **Método**: `GET`
- **Caminho**: `/api/v1/users/{userId}/roles`
- **Autenticação**: Bearer token requerido

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Média
- Tempo de resposta: 150-350ms
- TTL de Cache: 10 minutos

### 6.2 Atribuir Role ao Usuário
- **Método**: `POST`
- **Caminho**: `/api/v1/users/{userId}/roles/{roleId}`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário
- `roleId` (string, obrigatório): UUID da role

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@empresa.com.br",
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

**Cenários de Erro**:
- `404`: Usuário ou role não encontrado
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `409`: Role já atribuída ao usuário
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 200-500ms
- Invalidação de cache: Roles do usuário, detalhes do usuário

### 6.3 Revogar Role do Usuário
- **Método**: `DELETE`
- **Caminho**: `/api/v1/users/{userId}/roles/{roleId}`
- **Autenticação**: Bearer token requerido (Admin)

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário
- `roleId` (string, obrigatório): UUID da role

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@empresa.com.br",
  "roles": [],
  "updated_at": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `404`: Usuário ou role não encontrado
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `400`: Role não atribuída ao usuário
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 200-500ms
- Invalidação de cache: Roles do usuário, detalhes do usuário

### 6.4 Atribuir Roles em Lote
- **Método**: `POST`
- **Caminho**: `/api/v1/users/{userId}/roles/bulk-assign`
- **Autenticação**: Bearer token requerido (Admin)
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário

**Corpo da Requisição**:
```json
{
  "roleIds": ["role-id-1", "role-id-2", "role-id-3"]
}
```

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@empresa.com.br",
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

**Cenários de Erro**:
- `400`: IDs de roles inválidos ou array vazio
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 300-800ms (múltiplas operações)
- Invalidação de cache: Roles do usuário, detalhes do usuário

### 6.5 Revogar Roles em Lote
- **Método**: `POST`
- **Caminho**: `/api/v1/users/{userId}/roles/bulk-revoke`
- **Autenticação**: Bearer token requerido (Admin)
- **Content-Type**: `application/json`

**Parâmetros do Caminho**:
- `userId` (string, obrigatório): UUID do usuário

**Corpo da Requisição**:
```json
{
  "roleIds": ["role-id-1", "role-id-2"]
}
```

**Resposta de Sucesso (200)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@empresa.com.br",
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

**Cenários de Erro**:
- `400`: IDs de roles inválidos ou array vazio
- `404`: Usuário não encontrado
- `401`: Não autorizado
- `403`: Privilégios insuficientes
- `500`: Erro de servidor

**Padrões de Uso**:
- Frequência: Baixa
- Tempo de resposta: 300-800ms (múltiplas operações)
- Invalidação de cache: Roles do usuário, detalhes do usuário

## 7. Endpoints do Sistema

### 7.1 Verificação de Saúde (Simples)
- **Método**: `GET`
- **Caminho**: `/`
- **Autenticação**: Não requerida

**Resposta de Sucesso (200)**:
```json
{
  "status": "ok",
  "service": "user-svc-domain",
  "timestamp": "2024-01-20T15:45:00Z"
}
```

**Cenários de Erro**:
- `500`: Serviço indisponível

**Padrões de Uso**:
- Frequência: Muito alta (verificações de saúde do load balancer)
- Tempo de resposta: 10-50ms
- Cache: Sem cache

### 7.2 Verificação de Saúde (Detalhada)
- **Método**: `GET`
- **Caminho**: `/health`
- **Autenticação**: Não requerida

**Resposta de Sucesso (200)**:
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

**Cenários de Erro**:
- `503`: Serviço degradado (algumas verificações falharam)
- `500`: Serviço indisponível

**Padrões de Uso**:
- Frequência: Alta (sistemas de monitoramento)
- Tempo de resposta: 20-100ms
- Cache: Sem cache

### 7.3 Métricas
- **Método**: `GET`
- **Caminho**: `/metrics`
- **Autenticação**: Não requerida (uso interno)
- **Content-Type**: `text/plain`

**Resposta de Sucesso (200)**:
```
# HELP http_requests_total O número total de requisições HTTP
# TYPE http_requests_total counter
http_requests_total{method="GET",status="200"} 1024
http_requests_total{method="POST",status="201"} 256
http_requests_total{method="POST",status="400"} 12

# HELP http_request_duration_seconds A latência de requisições HTTP
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.1"} 789
http_request_duration_seconds_bucket{le="0.5"} 1200
http_request_duration_seconds_bucket{le="1.0"} 1350
http_request_duration_seconds_bucket{le="+Inf"} 1400

# HELP user_service_users_total Número total de usuários no sistema
# TYPE user_service_users_total gauge
user_service_users_total 1500

# HELP user_service_active_sessions_total Sessões ativas de usuário
# TYPE user_service_active_sessions_total gauge
user_service_active_sessions_total 45
```

**Cenários de Erro**:
- `500`: Falha na coleta de métricas

**Padrões de Uso**:
- Frequência: Alta (scraping Prometheus a cada 30s)
- Tempo de resposta: 50-200ms
- Cache: Sem cache (métricas em tempo real)

## 8. Recomendações para Simulação de Tráfego

### 8.1 Padrões de Uso Realistas

**Horários de Pico**:
- Horário comercial: 8h - 18h (fuso horário Brasil)
- Pico do almoço: 12h - 14h
- Fim de semana: 50% de tráfego reduzido

**Distribuição de Frequência de Endpoints**:
- Autenticação (`/auth/*`): 20% do total de requisições
- Consultas de usuário (`GET /users/*`): 40% do total de requisições
- Modificações de usuário (`POST/PUT /users/*`): 15% do total de requisições
- Operações de perfil: 10% do total de requisições
- Operações de role/permissão: 5% do total de requisições
- Endpoints do sistema: 10% do total de requisições

### 8.2 Cenários de Simulação de Erro

**Erros de Rede**:
- Timeouts de conexão: 2% das requisições
- Picos de latência de rede: 5% das requisições
- Quedas de conexão: 1% das requisições

**Erros de Aplicação**:
- Falhas de autenticação: 8% das tentativas de login
- Erros de validação: 12% das requisições de criação/atualização
- Erros de não encontrado: 15% das requisições de consulta
- Permissão negada: 5% do acesso a recursos protegidos

**Erros de Servidor**:
- Problemas de conexão com banco de dados: 0,5% das requisições
- Cache indisponível: 1% das requisições
- Sobrecarga de serviço: 0,2% das requisições durante horários de pico

### 8.3 Cenários de Teste de Carga

**Cenário 1: Carga Normal de Negócio**
- 100 usuários concorrentes
- 10 requisições por usuário por minuto
- Uso misto de endpoints (seguindo distribuição de frequência)
- Taxa de sucesso esperada: 95%

**Cenário 2: Carga de Pico**
- 500 usuários concorrentes
- 20 requisições por usuário por minuto
- Foco pesado em autenticação e consulta de usuários
- Taxa de sucesso esperada: 90%

**Cenário 3: Teste de Estresse**
- 1000+ usuários concorrentes
- 50 requisições por usuário por minuto
- Monitorar degradação graciosa do sistema
- Limites de performance degradada aceitáveis

### 8.4 Simulação de Cache

**Taxas de Acerto de Cache** (valores alvo):
- Consultas de usuário: 80%
- Dados de role/permissão: 95%
- Dados de perfil: 70%
- Dados de autenticação: 60%

**Gatilhos de Invalidação de Cache**:
- Atualizações de usuário: Limpar caches específicos do usuário
- Mudanças de role: Limpar caches relacionados a roles
- Mudanças de permissão: Limpar caches de permissões
- Limpeza geral: A cada 6 horas

### 8.5 Testes de Segurança

**Rate Limiting**:
- Tentativas de login: 5 por minuto por IP
- Mudanças de senha: 3 por hora por usuário
- Chamadas da API: 100 por minuto por usuário autenticado

**Teste de Autenticação**:
- Cenários de token válido: 85%
- Cenários de token expirado: 10%
- Cenários de token inválido: 5%

### 8.6 Teste de Consistência de Dados

**Operações Concorrentes**:
- Múltiplos usuários atualizando o mesmo recurso
- Atribuições de roles durante atualizações de usuário
- Consistência de cache durante atualizações de alta frequência
- Cenários de rollback de transação

Este mapeamento abrangente de endpoints da API fornece todas as informações necessárias para implementar um simulador de tráfego robusto que pode gerar padrões de carga realistas, testar vários cenários de erro e validar a performance do sistema sob diferentes condições.