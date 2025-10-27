# Sistema de Autenticación Avanzado con Roles

## Descripción General

Sistema de autenticación completo para la plataforma web de la fundación, implementando roles jerárquicos, gestión de usuarios, auditoría y recuperación de contraseñas.

## Roles del Sistema

### 1. SUPER_ADMIN
- **Permisos completos** en el sistema
- Puede crear y gestionar administradores
- Acceso a auditoría completa del sistema
- No puede ser modificado por otros usuarios

### 2. ADMIN
- Personal de la fundación
- Gestión de niños
- Comunicación por chat entre padrino y administrador
- Gestión de donaciones
- **No puede** crear otros administradores

### 3. PADRINO
- Usuario regular
- Ver catálogo de niños
- Apadrinar un niño
- Chatear con administrador
- Ver bitácora del niño apadrinado
- Descargar PDF de seguimiento
- Realizar donaciones

## Requisitos Funcionales Implementados

### RF-001: Registro de Padrinos
**Endpoint:** `POST /api/auth/register`

Permite el registro de nuevos usuarios con rol de padrino.

**Request Body:**
```json
{
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "password": "Password123",
  "phone": "+51987654321",
  "documentId": "12345678",
  "address": "Av. Principal 123, Lima"
}
```

**Validaciones:**
- Email único en el sistema
- Contraseña mínimo 8 caracteres con mayúsculas, minúsculas y números
- Documento de identidad único
- Email de confirmación enviado (pendiente configurar servicio de email)
- Usuario creado en estado `PENDING_ACTIVATION`

**Response:**
```json
{
  "message": "Usuario registrado exitosamente. Por favor verifica tu email.",
  "userId": 1
}
```

---

### RF-002: Autenticación de Usuarios
**Endpoint:** `POST /api/auth/login`

Login con credenciales (email/contraseña) para todos los roles.

**Request Body:**
```json
{
  "email": "juan@example.com",
  "password": "Password123"
}
```

**Criterios de Seguridad:**
- Máximo 5 intentos fallidos bloquean cuenta temporalmente (15 minutos)
- Token JWT generado con expiración de 24 horas
- Refresh token con expiración de 7 días
- Registro de IP y User-Agent en auditoría
- Validación de email verificado antes de permitir login

**Response Exitoso:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "email": "juan@example.com",
    "name": "Juan Pérez",
    "role": "PADRINO",
    "status": "ACTIVE",
    "emailVerified": true
  }
}
```

**Errores Posibles:**
- `401`: Credenciales inválidas
- `403`: Cuenta bloqueada, inactiva o suspendida
- `403`: Email no verificado

---

### RF-003: Recuperación de Contraseña

#### Paso 1: Solicitar Reset
**Endpoint:** `POST /api/auth/password/request-reset`

**Request Body:**
```json
{
  "email": "juan@example.com"
}
```

**Response:**
```json
{
  "message": "Si el email existe, recibirás instrucciones para restablecer tu contraseña."
}
```

**Notas de Seguridad:**
- Siempre devuelve el mismo mensaje (no revela si el email existe)
- Token de reset válido por 1 hora
- Email enviado con link de recuperación (pendiente configurar servicio)

#### Paso 2: Confirmar Reset
**Endpoint:** `POST /api/auth/password/reset`

**Request Body:**
```json
{
  "token": "abc123def456...",
  "newPassword": "NewPassword123"
}
```

**Efectos:**
- Contraseña anterior invalidada
- Todos los refresh tokens revocados
- Contador de intentos fallidos reseteado
- Cuenta desbloqueada automáticamente

---

### RF-004: Gestión de Perfil de Padrino

#### Obtener Perfil Actual
**Endpoint:** `GET /api/auth/profile`
**Requiere:** JWT Token (cualquier rol)

**Response:**
```json
{
  "id": 1,
  "email": "juan@example.com",
  "name": "Juan Pérez",
  "role": "PADRINO",
  "status": "ACTIVE",
  "phone": "+51987654321",
  "address": "Av. Principal 123, Lima",
  "avatar": "https://example.com/avatar.jpg",
  "lastLoginAt": "2025-01-27T10:30:00Z",
  "createdAt": "2025-01-20T10:00:00Z"
}
```

#### Actualizar Perfil
**Endpoint:** `PATCH /api/auth/profile`
**Requiere:** JWT Token (rol PADRINO)

**Campos Permitidos para Actualizar:**
- `phone`: Teléfono
- `address`: Dirección
- `avatar`: URL de foto de perfil

**Campos NO Modificables:**
- `email`
- `documentId`
- `role`
- `status`

**Request Body:**
```json
{
  "phone": "+51999888777",
  "address": "Nueva dirección 456",
  "avatar": "https://example.com/new-avatar.jpg"
}
```

---

### RF-005: Gestión de Administradores

#### Crear Administrador
**Endpoint:** `POST /api/auth/admins`
**Requiere:** JWT Token (rol SUPER_ADMIN)

**Request Body:**
```json
{
  "name": "Admin User",
  "email": "admin@fundacion.org",
  "password": "AdminPass123",
  "role": "ADMIN"
}
```

**Características:**
- Solo SUPER_ADMIN puede crear administradores
- Administradores creados en estado `ACTIVE`
- Email automáticamente verificado
- Registro de quién creó al administrador (`createdBy`)
- Auditoría completa de la acción

#### Actualizar Administrador
**Endpoint:** `PATCH /api/auth/admins/:adminId`
**Requiere:** JWT Token (rol SUPER_ADMIN)

**Request Body:**
```json
{
  "name": "Admin Updated",
  "status": "ACTIVE"
}
```

**Restricciones:**
- No se puede actualizar a sí mismo
- Solo se pueden actualizar usuarios con rol ADMIN o SUPER_ADMIN
- Cambios registrados en auditoría

#### Listar Administradores
**Endpoint:** `GET /api/auth/admins`
**Requiere:** JWT Token (rol SUPER_ADMIN)

**Response:**
```json
[
  {
    "id": 2,
    "email": "admin@fundacion.org",
    "name": "Admin User",
    "role": "ADMIN",
    "status": "ACTIVE",
    "lastLoginAt": "2025-01-27T09:00:00Z",
    "createdAt": "2025-01-25T08:00:00Z",
    "createdBy": {
      "id": 1,
      "name": "Super Admin",
      "email": "super@fundacion.org"
    }
  }
]
```

---

## Endpoints Adicionales

### Verificar Email
**Endpoint:** `POST /api/auth/verify-email`

```json
{
  "token": "email-verification-token"
}
```

Activa la cuenta del usuario y permite el login.

### Refrescar Token
**Endpoint:** `POST /api/auth/refresh`

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:**
```json
{
  "accessToken": "nuevo-access-token"
}
```

### Cerrar Sesión
**Endpoint:** `POST /api/auth/logout`
**Requiere:** JWT Token

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

Revoca el refresh token especificado.

---

## Sistema de Auditoría

Todas las acciones importantes son registradas en la tabla `audit_logs`:

**Acciones Auditadas:**
- `USER_CREATED`: Creación de usuario
- `USER_UPDATED`: Actualización de usuario
- `USER_ACTIVATED`: Activación de cuenta
- `USER_DEACTIVATED`: Desactivación de cuenta
- `USER_SUSPENDED`: Suspensión de cuenta
- `PASSWORD_RESET`: Reset de contraseña
- `EMAIL_VERIFIED`: Verificación de email
- `LOGIN_SUCCESS`: Login exitoso
- `LOGIN_FAILED`: Intento de login fallido
- `LOGOUT`: Cierre de sesión
- `ADMIN_CREATED`: Creación de administrador

**Información Registrada:**
- Acción realizada
- Usuario afectado
- Usuario que realizó la acción
- IP Address
- User Agent
- Metadata adicional (cambios realizados, razón de fallo, etc.)
- Timestamp

---

## Base de Datos

### Modelo User
```prisma
model User {
  id                   Int              @id @default(autoincrement())
  email                String           @unique
  name                 String
  password             String
  role                 Role             @default(PADRINO)
  status               UserStatus       @default(PENDING_ACTIVATION)
  phone                String?
  address              String?
  documentId           String?          @unique
  avatar               String?
  lastLoginAt          DateTime?
  failedLoginCount     Int              @default(0)
  lockedUntil          DateTime?
  resetPasswordToken   String?          @unique
  resetPasswordExpires DateTime?
  emailVerified        Boolean          @default(false)
  emailVerificationToken String?        @unique
  createdById          Int?
  createdAt            DateTime         @default(now())
  updatedAt            DateTime         @updatedAt
}
```

### Estados de Usuario (UserStatus)
- `ACTIVE`: Usuario activo, puede usar el sistema
- `INACTIVE`: Usuario inactivo, no puede hacer login
- `PENDING_ACTIVATION`: Usuario registrado pendiente de verificar email
- `SUSPENDED`: Usuario suspendido por administrador

---

## Seguridad

### Hashing de Contraseñas
- Algoritmo: bcrypt
- Salt rounds: 10

### Tokens JWT
- **Access Token:**
  - Expiración: 24 horas
  - Contiene: userId, email, name, role
  - Algoritmo: HS256

- **Refresh Token:**
  - Expiración: 7 días
  - Almacenado en base de datos
  - Puede ser revocado

### Bloqueo de Cuenta
- Máximo 5 intentos fallidos
- Bloqueo temporal de 15 minutos
- Desbloqueo automático después del tiempo
- Contador reseteado en login exitoso

### Tokens de Reset/Verificación
- Generados con `crypto.randomBytes(32)`
- 64 caracteres hexadecimales
- Token de reset: válido 1 hora
- Token de verificación: sin expiración (hasta que se use)

---

## Configuración

### Variables de Entorno - Auth Service

```env
# Server
PORT=3002

# Kafka
KAFKA_BROKER=localhost:9092

# Database
DATABASE_URL=postgresql://user:password@localhost:54321/auth_service?schema=public

# JWT
JWT_SECRET=tu-secreto-jwt-super-seguro-cambiar-en-produccion
```

### Variables de Entorno - Client Gateway

```env
# Server
PORT=3000

# Kafka
KAFKA_BROKER=localhost:9092

# JWT (mismo secret que auth-service)
JWT_SECRET=tu-secreto-jwt-super-seguro-cambiar-en-produccion

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
```

---

## Flujo de Arquitectura

```
┌─────────────┐
│   Cliente   │
└──────┬──────┘
       │ HTTP Request (JWT en header)
       ▼
┌─────────────────────────────────┐
│      Client Gateway             │
│                                 │
│  1. JwtAuthGuard verifica token │
│  2. RolesGuard verifica roles   │
│  3. Controller recibe request   │
│  4. Service envía a Kafka       │
└──────────┬──────────────────────┘
           │ Kafka Message
           ▼
┌─────────────────────────────────┐
│      Auth Service               │
│                                 │
│  1. Controller recibe mensaje   │
│  2. AuthService procesa lógica  │
│  3. HashService / TokenService  │
│  4. PrismaService (DB)          │
│  5. AuditService registra       │
│  6. Responde por Kafka          │
└──────────┬──────────────────────┘
           │
           ▼
        Database
     (PostgreSQL)
```

---

## Próximos Pasos / TODOs

### Funcionalidades Pendientes:

1. **Servicio de Email**
   - Configurar servicio (SendGrid, AWS SES, etc.)
   - Implementar templates de email
   - Envío de verificación de email
   - Envío de recuperación de contraseña

2. **Rate Limiting**
   - Limitar intentos de registro por IP
   - Limitar intentos de login por IP
   - Limitar solicitudes de reset de contraseña

3. **2FA (Autenticación de Dos Factores)**
   - Opcional para administradores
   - TOTP (Google Authenticator)

4. **Gestión de Sesiones**
   - Ver sesiones activas
   - Cerrar sesiones específicas
   - Cerrar todas las sesiones

5. **Mejoras de Auditoría**
   - Dashboard de auditoría para super-admin
   - Exportación de logs de auditoría
   - Alertas de actividad sospechosa

---

## Testing

### Crear un Super Admin Inicial

Para empezar a usar el sistema, necesitas crear manualmente el primer super admin en la base de datos:

```sql
-- Hashea la contraseña primero usando bcrypt con 10 rounds
-- Por ejemplo, "SuperAdmin123" hasheado = "$2b$10$..."

INSERT INTO users (
  email,
  name,
  password,
  role,
  status,
  "emailVerified",
  "createdAt",
  "updatedAt"
) VALUES (
  'super@fundacion.org',
  'Super Administrador',
  '$2b$10$TuPasswordHasheadaAqui',
  'SUPER_ADMIN',
  'ACTIVE',
  true,
  NOW(),
  NOW()
);
```

O usar un script de seeding en Prisma (recomendado).

### Probar Endpoints con Swagger

Accede a: `http://localhost:3000/api/docs`

Swagger UI proporciona documentación interactiva de todos los endpoints.

---

## Comandos Útiles

### Auth Service

```bash
# Generar cliente Prisma
cd auth-service
pnpm run prisma:generate

# Crear migración
npx prisma migrate dev --name descripcion_migracion

# Ver base de datos con Prisma Studio
npx prisma studio

# Iniciar en desarrollo
pnpm run start:dev
```

### Client Gateway

```bash
cd client-gateway

# Iniciar en desarrollo
pnpm run start:dev
```

### Infrastructure

```bash
# Iniciar servicios (PostgreSQL, Kafka, Redis)
docker compose up -d

# Ver logs
docker compose logs -f

# Detener servicios
docker compose down
```

---

## Troubleshooting

### Error: "Environment variable configuration error: Missing JWT_SECRET"

**Solución:** Asegúrate de tener `JWT_SECRET` en el archivo `.env` de auth-service.

### Error: "Cannot connect to Kafka"

**Solución:** Verifica que Kafka esté corriendo con `docker compose ps` y que `KAFKA_BROKER` esté configurado correctamente.

### Error: "P2002: Unique constraint failed"

**Solución:** El email o documento ya existe en la base de datos. Usa uno diferente.

### Login devuelve "Email no verificado"

**Solución:** Para testing, actualiza manualmente `emailVerified = true` en la base de datos, o implementa el servicio de email para recibir el link de verificación.

---

## Documentación de API

Documentación interactiva disponible en: `http://localhost:3000/api/docs`

Todos los endpoints están documentados con:
- Descripción de la funcionalidad
- Request body schema
- Response schema
- Códigos de estado HTTP
- Ejemplos

---

## Contacto

Para preguntas o reportar issues relacionados con el sistema de autenticación, contactar al equipo de desarrollo.
