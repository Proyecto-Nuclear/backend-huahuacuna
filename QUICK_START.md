# ⚡ Quick Start - Pruebas del Sistema de Autenticación

## 🎯 Inicio Rápido (3 opciones)

### Opción 1: Swagger UI (Recomendado - Más Fácil)

1. **Abrir Swagger:**
   ```
   http://localhost:3000/api/docs
   ```

2. **Hacer Login (obtener token):**
   - Click en `POST /api/auth/login`
   - Click en "Try it out"
   - Usar:
     ```json
     {
       "email": "super@fundacion.org",
       "password": "SuperAdmin123"
     }
     ```
   - Click "Execute"
   - **Copiar el `accessToken` de la respuesta**

3. **Autorizar Swagger:**
   - Click en el botón "Authorize" (arriba a la derecha, ícono de candado)
   - Pegar: `Bearer {tu_accessToken}`
   - Click "Authorize"
   - Click "Close"

4. **¡Ya puedes probar todos los endpoints!**
   - Los endpoints con candado requieren autenticación
   - Los que no tienen candado son públicos

---

### Opción 2: Postman

1. **Importar colección:**
   - Abrir Postman
   - Click "Import"
   - Seleccionar archivo: `Huahuacuna_Auth_API.postman_collection.json`

2. **Ejecutar requests en orden:**
   - Los tokens se guardan automáticamente en variables de entorno
   - Ejecuta desde el #1 hasta el #14

---

### Opción 3: cURL (Terminal)

```bash
# 1. Login
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "super@fundacion.org",
    "password": "SuperAdmin123"
  }'

# 2. Guardar el token que te devuelve
export TOKEN="tu_accessToken_aqui"

# 3. Obtener perfil
curl http://localhost:3000/api/auth/profile \
  -H "Authorization: Bearer $TOKEN"
```

---

## 📋 Credenciales Iniciales

```
Email: super@fundacion.org
Password: SuperAdmin123
```

---

## 🧪 Secuencia de Pruebas Recomendada

### Fase 1: Autenticación Básica
1. ✅ **Test** - `GET /api/auth/test`
2. ✅ **Login Super Admin** - `POST /api/auth/login`
3. ✅ **Ver Perfil** - `GET /api/auth/profile`

### Fase 2: Gestión de Administradores (requiere token)
4. ✅ **Crear Admin** - `POST /api/auth/admins`
5. ✅ **Listar Admins** - `GET /api/auth/admins`
6. ✅ **Actualizar Admin** - `PATCH /api/auth/admins/2`

### Fase 3: Registro de Padrino
7. ✅ **Registrar Padrino** - `POST /api/auth/register`
8. ⚠️ **Activar manualmente** (ver abajo)
9. ✅ **Login Padrino** - `POST /api/auth/login`
10. ✅ **Actualizar Perfil** - `PATCH /api/auth/profile`

### Fase 4: Gestión de Tokens
11. ✅ **Refrescar Token** - `POST /api/auth/refresh`
12. ✅ **Cerrar Sesión** - `POST /api/auth/logout`

### Fase 5: Recuperación de Contraseña
13. ✅ **Solicitar Reset** - `POST /api/auth/password/request-reset`
14. ⚠️ **Obtener token de BD** (ver abajo)
15. ✅ **Restablecer** - `POST /api/auth/password/reset`

---

## ⚠️ Para Testing: Activar Padrino Manualmente

Como no tenemos servicio de email configurado, después de registrar un padrino:

### Opción A: Prisma Studio (Visual)
```bash
cd auth-service
npx prisma studio
```
- Abrir en: http://localhost:5555
- Click en tabla "User"
- Buscar el padrino
- Cambiar `emailVerified` a `true`
- Cambiar `status` a `ACTIVE`
- Cambiar `emailVerificationToken` a `null`
- Guardar

### Opción B: SQL Directo
```sql
UPDATE users
SET "emailVerified" = true,
    status = 'ACTIVE',
    "emailVerificationToken" = null
WHERE email = 'juan@example.com';
```

---

## 🎯 Ejemplo Completo (Swagger UI)

### 1. Test de Comunicación
- Endpoint: `GET /api/auth/test`
- Click "Try it out" → "Execute"
- Debe responder: `"message": "Auth service is running successfully"`

### 2. Login
- Endpoint: `POST /api/auth/login`
- Body:
  ```json
  {
    "email": "super@fundacion.org",
    "password": "SuperAdmin123"
  }
  ```
- Copiar `accessToken` de la respuesta

### 3. Autorizar
- Click botón "Authorize" (arriba)
- Escribir: `Bearer {tu_token}`
- Click "Authorize" → "Close"

### 4. Crear Admin
- Endpoint: `POST /api/auth/admins`
- Body:
  ```json
  {
    "name": "Admin de Prueba",
    "email": "admin@fundacion.org",
    "password": "AdminPass123",
    "role": "ADMIN"
  }
  ```
- Click "Execute"
- Debe crear el admin exitosamente

### 5. Ver Admins
- Endpoint: `GET /api/auth/admins`
- Click "Execute"
- Debe listar el super admin y el nuevo admin

### 6. Registrar Padrino
- Endpoint: `POST /api/auth/register`
- Body:
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

### 7. Activar Padrino (Prisma Studio o SQL)
```sql
UPDATE users
SET "emailVerified" = true, status = 'ACTIVE'
WHERE email = 'juan@example.com';
```

### 8. Login como Padrino
- Endpoint: `POST /api/auth/login`
- Body:
  ```json
  {
    "email": "juan@example.com",
    "password": "Password123"
  }
  ```
- Copiar nuevo `accessToken`
- Autorizar nuevamente con este token

### 9. Actualizar Perfil
- Endpoint: `PATCH /api/auth/profile`
- Body:
  ```json
  {
    "phone": "+51999111222",
    "address": "Nueva dirección"
  }
  ```

---

## 🐛 Problemas Comunes

### "Unauthorized" / 401
- ¿Copiaste el token completo?
- ¿Escribiste "Bearer " antes del token?
- ¿El token no expiró? (válido 24h)

### "Forbidden" / 403
- ¿Tienes el rol correcto?
  - Crear admins: requiere SUPER_ADMIN
  - Actualizar perfil: requiere PADRINO
- ¿La cuenta está activa?

### "Email no verificado"
- Ejecuta el SQL para activar manualmente
- O usa Prisma Studio

### "Cuenta bloqueada"
- Esperá 15 minutos
- O ejecuta:
  ```sql
  UPDATE users
  SET "failedLoginCount" = 0, "lockedUntil" = null
  WHERE email = 'tu_email';
  ```

---

## 📊 Ver Datos en la BD

### Opción 1: Prisma Studio (Visual)
```bash
cd auth-service
npx prisma studio
# Abre en http://localhost:5555
```

### Opción 2: SQL directo
Conectar a PostgreSQL:
```bash
psql -h localhost -p 54321 -U user -d auth_service
```

Ver usuarios:
```sql
SELECT id, email, name, role, status, "emailVerified" FROM users;
```

Ver auditoría:
```sql
SELECT * FROM audit_logs ORDER BY "createdAt" DESC LIMIT 10;
```

---

## ✅ Checklist Rápido

- [ ] Servicios corriendo (Docker, auth-service, client-gateway)
- [ ] Super admin creado
- [ ] Login exitoso
- [ ] Token guardado
- [ ] Swagger autorizado
- [ ] Admin creado
- [ ] Padrino registrado y activado
- [ ] Auditoría registrando acciones

---

**¡Listo para probar!** 🚀

Si tienes problemas, revisa `GUIA_PRUEBAS.md` para más detalles.
