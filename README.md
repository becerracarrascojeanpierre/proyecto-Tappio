# Tappio — starter para Vercel

Base inicial de una aplicación privada para administrar documentos vehiculares mediante un NFC por vehículo.

## Stack
- Next.js App Router + TypeScript
- Supabase Auth (sesiones SSR)
- Supabase PostgreSQL con RLS
- Supabase Storage privado
- Vercel

## Qué incluye
- Estructura inicial de Next.js.
- Pantallas de inicio, login, panel y ruta NFC `/v/[slug]`.
- Clientes Supabase para navegador y servidor.
- SQL inicial con perfiles, vehículos, etiquetas NFC y documentos.
- Bucket privado y políticas RLS iniciales.
- Ejemplo de invitación administrativa mediante Supabase Auth Admin API.
- Variables de entorno y pasos de despliegue.

> Este es un starter técnico, no una aplicación ya desplegada ni auditada. Antes de producción se deben completar las pantallas CRUD, probar las políticas RLS y revisar los flujos de invitación y recuperación.

## Requisitos
- Node.js 20.9 o superior.
- Proyecto Supabase.
- Proyecto Vercel conectado a un repositorio Git.

## Instalación
```bash
npm install
cp .env.example .env.local
npm run dev
```

Completa `.env.local` con:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY` (solo servidor; nunca usar prefijo `NEXT_PUBLIC_`)
- `NEXT_PUBLIC_APP_URL`

## Preparar Supabase
1. Crea un proyecto Supabase.
2. Abre SQL Editor y ejecuta `supabase/schema.sql`.
3. En Authentication, desactiva el registro público (Allow new users to sign up).
4. Crea el bucket privado `vehicle-documents` si el SQL no lo creó automáticamente.
5. Configura URL Site URL y Redirect URLs para tu dominio local y Vercel.
6. Añade las variables de entorno a Vercel y vuelve a desplegar.

## Primer administrador
El primer usuario administrador debe crearse manualmente desde Supabase Auth (Dashboard) y luego recibir el rol `admin` en `public.profiles`. No se debe exponer una ruta pública para asignar el rol administrador.

Ejemplo SQL (reemplaza el UUID por el ID real del usuario):
```sql
update public.profiles
set role = 'admin'
where id = 'UUID-DEL-USUARIO';
```

## Alta de usuarios
El ejemplo `app/api/admin/invite/route.ts` usa `SUPABASE_SERVICE_ROLE_KEY` exclusivamente en el servidor. Protege la operación comprobando que quien llama sea administrador. El usuario recibe un correo de invitación y define su contraseña mediante el flujo de Supabase Auth.

Flujo recomendado para la etiqueta:
1. El administrador invita al usuario.
2. El administrador crea el vehículo y genera su etiqueta NFC desde el panel (pendiente de implementar en UI).
3. La etiqueta apunta a `https://tu-dominio/v/SLUG-ALEATORIO`.
4. El usuario inicia sesión. La página NFC solo muestra datos si el usuario tiene permiso sobre ese vehículo.
5. El slug NFC no es una contraseña ni concede permisos.

## NFC
Usa un slug aleatorio de alta entropía como localizador, no patente, correo, UUID secuencial ni datos personales. La página `/v/[slug]` debe requerir sesión y validar permisos en servidor. Para mayor protección, el SQL guarda un hash del slug; la implementación de generación/consulta del slug debe completarse antes de producción.

## Seguridad antes de producción
- No hacer público el bucket de documentos.
- No guardar documentos en la etiqueta NFC.
- No poner `SUPABASE_SERVICE_ROLE_KEY` en el navegador.
- Mantener RLS activa en todas las tablas expuestas.
- Probar acceso cruzado entre dos usuarios distintos.
- Limitar tamaño y tipos de archivo en Storage.
- Añadir rate limiting a login, invitaciones y generación de NFC.
- Añadir auditoría de cambios y política de retención.
- Revisar cumplimiento legal y privacidad aplicable en Chile.
- Usar HTTPS y un dominio propio.

## Despliegue en Vercel
1. Sube esta carpeta a un repositorio Git.
2. En Vercel selecciona **Add New → Project** e importa el repositorio.
3. Añade las variables de entorno en Project Settings → Environment Variables.
4. Despliega.
5. Configura el dominio y actualiza las URLs permitidas en Supabase.

Comandos:
```bash
npm run build
```

## Estructura
```text
app/
  api/admin/invite/route.ts
  login/page.tsx
  panel/page.tsx
  v/[slug]/page.tsx
  layout.tsx
  page.tsx
lib/supabase/
  client.ts
  server.ts
supabase/
  schema.sql
```


## Refuerzos de seguridad incluidos en esta revisión
- Middleware Supabase SSR para refrescar cookies de sesión.
- Cabeceras HTTP de seguridad y CSP inicial.
- Callback de autenticación con protección contra open redirect.
- Endpoint administrativo para emitir/rotar NFC aleatorio (256 bits), almacenar solo SHA-256 y desactivarlo.
- RLS más estricta y acceso Storage limitado por UUID del vehículo y bucket privado.
- Auditoría de cambios de vehículos, documentos y etiquetas (metadatos mínimos).
- Restricción de MIME y tamaño de archivo en bucket y tabla.
- Administrador no puede elevar roles desde el cliente; el primer admin se configura manualmente.

## Limitaciones y controles aún obligatorios
Este ZIP sigue siendo una base de desarrollo. La interfaz CRUD de documentos, carga validada, descarga con URL firmada de corta duración, interfaz de usuarios y panel de auditoría aún deben completarse. El endpoint NFC devuelve la URL una sola vez, pero falta la interfaz para mostrarla y programar la etiqueta.
La CSP es una política inicial y debe ajustarse/probarse con el despliegue real. La protección contra fuerza bruta, abuso de invitaciones y scraping requiere rate limiting distribuido (por ejemplo, Redis/Upstash o WAF), no un contador en memoria de funciones serverless.
Antes de producción se debe ejecutar pruebas de autorización con dos usuarios, pruebas de carga/descarga, escaneo de dependencias, revisión de secretos, revisión legal y una auditoría externa independiente.


## Versiones y cadena de suministro
La base fija Next.js 15.5.28 y React 19.1.7 como mínimo de esta línea, para incorporar correcciones de seguridad publicadas hasta octubre de 2026. Genera y versiona `package-lock.json` con `npm install` antes de desplegar; no publiques sin ejecutar `npm audit` y actualizar cualquier dependencia vulnerable. Next.js 15 está en mantenimiento de seguridad hasta el 31 de enero de 2027; planifica migrar a la línea Active LTS cuando corresponda.
