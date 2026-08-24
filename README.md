# 🐾 PawSocial

Red social para mascotas: feed de fotos, citas (veterinario, peluquería, paseador...) y una tienda administrada por ti, con pedidos gestionados manualmente.

Los usuarios externos crean su cuenta iniciando sesión con **Google** o **Facebook** (no hay contraseñas que gestionar). Tú controlas quién es administrador mediante la variable de entorno `ADMIN_EMAILS`.

## Funciones

- **Feed social**: publicaciones con foto, likes y comentarios.
- **Citas**: los usuarios agendan servicios (veterinario, peluquería, paseador, guardería, entrenamiento, spa) para sus mascotas.
- **Tienda administrada por ti**: solo el administrador publica productos; los clientes hacen pedidos y tú los gestionas manualmente (sin cobro en línea todavía).
- **Mensajería**: chat entre usuarios y un botón de "Contactar soporte" que abre conversación con el administrador.
- **Panel de administración** (`/admin`): productos, pedidos y usuarios (puedes ascender/descender administradores desde ahí).

## Stack técnico

- [Next.js 16](https://nextjs.org) (App Router, Server Actions)
- [Auth.js / NextAuth v5](https://authjs.dev) con proveedores Google y Facebook
- [Drizzle ORM](https://orm.drizzle.team) + PostgreSQL
- [Tailwind CSS v4](https://tailwindcss.com)
- [Vercel Blob](https://vercel.com/docs/storage/vercel-blob) para las fotos que suban los usuarios

---

## 1. Configuración local

### Requisitos

- Node.js 20.9 o superior
- Una base de datos Postgres (puede ser local o gratuita en la nube, ver paso 2)

### Instalación

```bash
npm install
cp .env.example .env
# edita .env con tus propios valores (ver los siguientes pasos)
```

### 2. Base de datos Postgres

La forma más rápida y gratuita es [Neon](https://neon.tech) (o puedes usar Vercel Postgres / Supabase):

1. Crea una cuenta gratis en neon.tech y un proyecto nuevo.
2. Copia la cadena de conexión ("Connection string") y pégala en `DATABASE_URL` dentro de `.env`.
3. Aplica el esquema de la base de datos:

   ```bash
   npm run db:push
   ```

4. Siembra los servicios por defecto de "Citas" (veterinario, peluquería, etc.):

   ```bash
   npm run db:seed
   ```

### 3. Iniciar sesión con Google

1. Ve a [Google Cloud Console → Credenciales](https://console.cloud.google.com/apis/credentials).
2. Crea un "OAuth client ID" de tipo **Web application**.
3. En "Authorized redirect URIs" agrega:
   - `http://localhost:3000/api/auth/callback/google` (para desarrollo)
   - `https://TU-DOMINIO/api/auth/callback/google` (cuando despliegues, ver paso 6)
4. Copia el Client ID y Client Secret en `AUTH_GOOGLE_ID` y `AUTH_GOOGLE_SECRET` en `.env`.

### 4. Iniciar sesión con Facebook

1. Ve a [Facebook for Developers](https://developers.facebook.com/apps) y crea una app de tipo "Consumer".
2. Agrega el producto "Facebook Login" y configura las "Valid OAuth Redirect URIs":
   - `http://localhost:3000/api/auth/callback/facebook`
   - `https://TU-DOMINIO/api/auth/callback/facebook`
3. Copia el App ID y App Secret en `AUTH_FACEBOOK_ID` y `AUTH_FACEBOOK_SECRET` en `.env`.
4. Mientras la app de Facebook esté en modo "Desarrollo", solo las personas que agregues como probadores/testers podrán iniciar sesión. Para abrirla a cualquier persona, debes enviarla a revisión de Meta ("App Review") solicitando el permiso `public_profile` (y `email` si lo necesitas) — esto puede tardar unos días.

### 5. Generar `AUTH_SECRET` y definir el administrador

```bash
npx auth secret
```

Copia el valor generado en `AUTH_SECRET` dentro de `.env`.

En `ADMIN_EMAILS`, escribe el correo (o correos, separados por coma) que deben convertirse automáticamente en **administrador** la primera vez que inicien sesión. Por ejemplo:

```
ADMIN_EMAILS="vato8263@gmail.com"
```

> Nota: el correo debe coincidir exactamente con el correo de la cuenta de Google/Facebook con la que inicies sesión. Una vez que ese usuario inicia sesión por primera vez, queda marcado como administrador para siempre (puedes gestionar más administradores luego desde `/admin/usuarios`).

### 6. Correr en local

```bash
npm run dev
```

Abre [http://localhost:3000](http://localhost:3000). Inicia sesión con la cuenta de Google/Facebook que pusiste en `ADMIN_EMAILS` para acceder al panel de administración en `/admin`.

---

## 2. Subida de fotos (opcional pero recomendado)

Las fotos de publicaciones, mascotas y productos se suben usando **Vercel Blob**. Sin configurarlo, todo funciona igual, solo que las publicaciones se guardan sin foto.

1. En tu proyecto de [Vercel](https://vercel.com), ve a **Storage → Create Database → Blob**.
2. Copia el token y pégalo en `BLOB_READ_WRITE_TOKEN` en tus variables de entorno.

---

## 3. Desplegar en producción (recomendado: Vercel)

1. Sube este proyecto a un repositorio de GitHub.
2. Entra a [vercel.com](https://vercel.com), "Add New Project" e importa el repositorio.
3. En "Environment Variables" agrega todas las variables de tu `.env` (`DATABASE_URL`, `AUTH_SECRET`, `AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`, `AUTH_FACEBOOK_ID`, `AUTH_FACEBOOK_SECRET`, `ADMIN_EMAILS`, `BLOB_READ_WRITE_TOKEN`).
4. Despliega. Vercel te dará un dominio como `https://tu-proyecto.vercel.app` (luego puedes conectar tu propio dominio en "Settings → Domains").
5. **Importante**: vuelve a Google Cloud Console y Facebook for Developers y agrega las redirect URIs de producción usando ese dominio final (ver pasos 3 y 4 arriba).
6. Aplica el esquema a tu base de datos de producción (puedes correr esto desde tu máquina apuntando a la misma `DATABASE_URL` que configuraste en Vercel):

   ```bash
   npm run db:push
   npm run db:seed
   ```

7. Entra al sitio ya desplegado e inicia sesión con el correo que pusiste en `ADMIN_EMAILS` — quedarás como administrador automáticamente.

---

## 4. Gestionar administradores

- El primer administrador se asigna automáticamente vía `ADMIN_EMAILS` la primera vez que esa persona inicia sesión.
- Para dar o quitar el rol de administrador a otras personas después de que ya se hayan registrado, entra a `/admin/usuarios` (con una cuenta admin) y usa el botón "Hacer admin" / "Quitar admin".

## 5. Pagos

Por ahora los pedidos de la tienda se gestionan manualmente: el cliente hace el pedido, tú lo ves en `/admin/pedidos` con sus datos de contacto, y coordinas el cobro (transferencia, efectivo, etc.) y la entrega por fuera del sitio, actualizando el estado del pedido a medida que avanza. Si en el futuro quieres cobro con tarjeta integrado (Stripe), es un siguiente paso natural — avísame cuando quieras agregarlo.

## Comandos útiles

| Comando | Descripción |
| --- | --- |
| `npm run dev` | Corre el sitio en local |
| `npm run build` | Compila para producción |
| `npm run db:push` | Aplica el esquema (`src/db/schema.ts`) a la base de datos |
| `npm run db:seed` | Crea los servicios por defecto de "Citas" |
| `npm run db:studio` | Abre una interfaz visual para explorar/editar la base de datos |
