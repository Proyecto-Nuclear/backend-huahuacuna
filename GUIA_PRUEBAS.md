# 🧪 Guía de Pruebas del Sistema de Autenticación

## 📋 Credenciales de Prueba

### Super Admin (Ya creado)
```
Email: super@fundacion.org
Password: SuperAdmin123
```

---

## 🚀 Acceso a Swagger UI

La manera más fácil de probar todos los endpoints es usando **Swagger UI**:

**URL:** http://localhost:3000/api/docs

---

## 📝 Secuencia de Pruebas

### 1️⃣ Verificar Comunicación (Sin autenticación)

**Endpoint:** `GET /api/auth/test`

**Respuesta esperada:**
```json
{
  "message": "Auth service is running successfully",
  "timestamp": "2025-01-27T..."
}
```

---

### 2️⃣ Login como Super Admin

**Endpoint:** `POST /api/auth/login`

**Request Body:**
```json
{
  "email": "super@fundacion.org",
  "password": "SuperAdmin123"
}
```

**Respuesta esperada:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "email": "super@fundacion.org",
    "name": "Super Administrador",
    "role": "SUPER_ADMIN",
    "status": "ACTIVE",
    "emailVerified": true
  }
}
```

**⚠️ IMPORTANTE:** Guarda el `accessToken` - lo necesitarás para los siguientes endpoints protegidos.

---

### 3️⃣ Obtener Perfil Actual (Protegido)

**Endpoint:** `GET /api/auth/profile`

**Headers:**
```
Authorization: Bearer {tu_accessToken}
```

**Respuesta esperada:**
```json
{
  "id": 1,
  "email": "super@fundacion.org",
  "name": "Super Administrador",
  "role": "SUPER_ADMIN",
  "status": "ACTIVE",
  "phone": "+51999888777",
  "documentId": "12345678",
  "address": "Oficina Central de la Fundación",
  "emailVerified": true,
  "lastLoginAt": "2025-01-27T...",
  "createdAt": "2025-01-27T..."
}
```

---

### 4️⃣ Crear un Administrador (Solo SUPER_ADMIN)

**Endpoint:** `POST /api/auth/admins`

**Headers:**
```
Authorization: Bearer {tu_accessToken}
```

**Request Body:**
```json
{
  "name": "Admin de Prueba",
  "email": "admin@fundacion.org",
  "password": "AdminPass123",
  "role": "ADMIN"
}
```

**Respuesta esperada:**
```json
{
  "id": 2,
  "email": "admin@fundacion.org",
  "name": "Admin de Prueba",
  "role": "ADMIN",
  "status": "ACTIVE",
  "emailVerified": true,
  "createdAt": "2025-01-27T...",
  "createdById": 1
}
```

---

### 5️⃣ Listar Administradores (Solo SUPER_ADMIN)

**Endpoint:** `GET /api/auth/admins`

**Headers:**
```
Authorization: Bearer {tu_accessToken}
```

**Respuesta esperada:**
```json
[
  {
    "id": 1,
    "email": "super@fundacion.org",
    "name": "Super Administrador",
    "role": "SUPER_ADMIN",
    "status": "ACTIVE",
    "lastLoginAt": "2025-01-27T...",
    "createdAt": "2025-01-27T...",
    "createdBy": null
  },
  {
    "id": 2,
    "email": "admin@fundacion.org",
    "name": "Admin de Prueba",
    "role": "ADMIN",
    "status": "ACTIVE",
    "lastLoginAt": null,
    "createdAt": "2025-01-27T...",
    "createdBy": {
      "id": 1,
      "name": "Super Administrador",
      "email": "super@fundacion.org"
    }
  }
]
```

---

### 6️⃣ Actualizar Administrador (Solo SUPER_ADMIN)

**Endpoint:** `PATCH /api/auth/admins/2`

**Headers:**
```
Authorization: Bearer {tu_accessToken}
```

**Request Body:**
```json
{
  "name": "Admin Actualizado",
  "status": "INACTIVE"
}
```

**Respuesta esperada:**
```json
{
  "id": 2,
  "email": "admin@fundacion.org",
  "name": "Admin Actualizado",
  "role": "ADMIN",
  "status": "INACTIVE",
  "emailVerified": true,
  "updatedAt": "2025-01-27T..."
}
```

---

### 7️⃣ Registrar un Padrino (Sin autenticación)

**Endpoint:** `POST /api/auth/register`

**Request Body:**
```json
{
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "password": "Password123",
  "phone": "+51987654321",
  "documentId": "87654321",
  "address": "Av. Principal 123, Lima"
}
```

**Respuesta esperada:**
```json
{
  "message": "Usuario registrado exitosamente. Por favor verifica tu email.",
  "userId": 3
}
```

**Nota:** El usuario queda en estado `PENDING_ACTIVATION` hasta verificar el email.

---

### 8️⃣ Verificar Email de Padrino

**IMPORTANTE:** Como no tenemos servicio de email configurado, necesitamos obtener el token manualmente de la BD:

```sql
-- Ejecutar en la base de datos
SELECT "emailVerificationToken" FROM users WHERE email = 'juan@example.com';
```

**Endpoint:** `POST /api/auth/verify-email`

**Request Body:**
```json
{
  "token": "{token_de_la_bd}"
}
```

**Respuesta esperada:**
```json
{
  "message": "Email verificado exitosamente. Ya puedes iniciar sesión."
}
```

**O ALTERNATIVA (más fácil para testing):**
```sql
-- Activar manualmente el padrino
UPDATE users
SET "emailVerified" = true,
    status = 'ACTIVE',
    "emailVerificationToken" = null
WHERE email = 'juan@example.com';
```

---

### 9️⃣ Login como Padrino

**Endpoint:** `POST /api/auth/login`

**Request Body:**
```json
{
  "email": "juan@example.com",
  "password": "Password123"
}
```

**Respuesta esperada:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 3,
    "email": "juan@example.com",
    "name": "Juan Pérez",
    "role": "PADRINO",
    "status": "ACTIVE",
    "emailVerified": true
  }
}
```

---

### 🔟 Actualizar Perfil de Padrino (Solo PADRINO)

**Endpoint:** `PATCH /api/auth/profile`

**Headers:**
```
Authorization: Bearer {accessToken_del_padrino}
```

**Request Body:**
```json
{
  "phone": "+51999111222",
  "address": "Nueva dirección 456, Cusco",
  "avatar": "https://example.com/avatar.jpg"
}
```

**Respuesta esperada:**
```json
{
  "id": 3,
  "email": "juan@example.com",
  "name": "Juan Pérez",
  "role": "PADRINO",
  "phone": "+51999111222",
  "address": "Nueva dirección 456, Cusco",
  "avatar": "https://example.com/avatar.jpg",
  "updatedAt": "2025-01-27T..."
}
```

---

### 1️⃣1️⃣ Refrescar Token

**Endpoint:** `POST /api/auth/refresh`

**Request Body:**
```json
{
  "refreshToken": "{tu_refreshToken}"
}
```

**Respuesta esperada:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 1️⃣2️⃣ Solicitar Recuperación de Contraseña

**Endpoint:** `POST /api/auth/password/request-reset`

**Request Body:**
```json
{
  "email": "juan@example.com"
}
```

**Respuesta esperada:**
```json
{
  "message": "Si el email existe, recibirás instrucciones para restablecer tu contraseña."
}
```

**Obtener token de reset de la BD:**
```sql
SELECT "resetPasswordToken" FROM users WHERE email = 'juan@example.com';
```

---

### 1️⃣3️⃣ Restablecer Contraseña

**Endpoint:** `POST /api/auth/password/reset`

**Request Body:**
```json
{
  "token": "{resetPasswordToken_de_la_bd}",
  "newPassword": "NewPassword123"
}
```

**Respuesta esperada:**
```json
{
  "message": "Contraseña restablecida exitosamente"
}
```

**Efectos:**
- Contraseña actualizada
- Todos los refresh tokens revocados
- Contador de intentos fallidos reseteado
- Cuenta desbloqueada si estaba bloqueada

---

### 1️⃣4️⃣ Cerrar Sesión

**Endpoint:** `POST /api/auth/logout`

**Headers:**
```
Authorization: Bearer {tu_accessToken}
```

**Request Body:**
```json
{
  "refreshToken": "{tu_refreshToken}"
}
```

**Respuesta esperada:**
```json
{
  "message": "Sesión cerrada exitosamente"
}
```

---

## 🧪 Pruebas de Seguridad

### Prueba 1: Intento de Login Fallido (Bloqueo de Cuenta)

Intenta hacer login con contraseña incorrecta **6 veces consecutivas**:

```json
{
  "email": "juan@example.com",
  "password": "ContraseñaIncorrecta"
}
```

**Después del 5to intento fallido:**
```json
{
  "error": true,
  "message": "Cuenta bloqueada temporalmente. Intenta nuevamente en 15 minuto(s).",
  "statusCode": 403
}
```

---

### Prueba 2: Intentar Crear Admin sin ser Super Admin

1. Login como Admin (no super-admin)
2. Intentar crear otro admin:

**Endpoint:** `POST /api/auth/admins`

**Respuesta esperada:**
```json
{
  "statusCode": 403,
  "message": "Se requiere uno de los siguientes roles: SUPER_ADMIN"
}
```

---

### Prueba 3: Padrino Intenta Actualizar Email (Campo Protegido)

**Endpoint:** `PATCH /api/auth/profile`

**Request Body:**
```json
{
  "email": "nuevo@email.com"
}
```

**Comportamiento:** El campo `email` será ignorado, solo se actualizan campos permitidos.

---

## 📊 Verificar Auditoría

Para ver los logs de auditoría en la base de datos:

```sql
SELECT
  al.id,
  al.action,
  al."createdAt",
  al."ipAddress",
  u1.email as affected_user,
  u2.email as performed_by
FROM audit_logs al
LEFT JOIN users u1 ON al."userId" = u1.id
LEFT JOIN users u2 ON al."performedBy" = u2.id
ORDER BY al."createdAt" DESC
LIMIT 20;
```

---

## 🎯 Checklist de Pruebas

- [ ] Test endpoint funciona
- [ ] Login con super admin exitoso
- [ ] Obtener perfil actual
- [ ] Crear administrador
- [ ] Listar administradores
- [ ] Actualizar administrador
- [ ] Registrar padrino
- [ ] Verificar email de padrino
- [ ] Login como padrino
- [ ] Actualizar perfil de padrino
- [ ] Refrescar token
- [ ] Solicitar reset de contraseña
- [ ] Restablecer contraseña
- [ ] Cerrar sesión
- [ ] Bloqueo por intentos fallidos (5 intentos)
- [ ] Protección de roles (padrino no puede crear admin)
- [ ] Campos protegidos (email, documentId)
- [ ] Auditoría registra todas las acciones

---

## 🛠️ Herramientas Recomendadas

1. **Swagger UI** (Recomendado)
   - URL: http://localhost:3000/api/docs
   - Interfaz visual
   - Documentación en vivo
   - Fácil de usar

2. **Postman**
   - Importar colección desde archivo JSON
   - Guardar variables de entorno (tokens)

3. **Thunder Client** (VS Code Extension)
   - Integrado en VS Code
   - Ligero y rápido

4. **Prisma Studio** (Para ver la BD)
   ```bash
   cd auth-service
   npx prisma studio
   ```
   URL: http://localhost:5555

---

## ⚡ Script de Prueba Rápida (opcional)

Voy a crear un script para probar todos los endpoints automáticamente...

---

## 🐛 Solución de Problemas

### Error: "Environment variable configuration error: Missing JWT_SECRET"
**Solución:** Agrega `JWT_SECRET` en `.env` del auth-service

### Error: "Cannot connect to Kafka"
**Solución:**
```bash
docker compose ps  # Verificar que Kafka esté corriendo
docker compose logs huahuacuna_kafka  # Ver logs
```

### Error: "P2002: Unique constraint failed"
**Solución:** El email o documento ya existe, usa uno diferente

### Error: "Email no verificado"
**Solución:** Ejecuta el SQL de activación manual o usa el endpoint de verify-email

---

¡Listo para empezar las pruebas! 🚀
