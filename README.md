# AULA DIGITAL RG — versión inicial

Plataforma educativa web con panel docente, panel estudiante y clases sincronizadas en tiempo real.

## Antes de empezar
- Cuenta gratuita en Supabase.
- Cuenta gratuita en GitHub.
- Cuenta gratuita en Cloudflare.
- Node.js y Supabase CLI para desplegar Edge Functions (opcional en la primera prueba).

## 1. Crear el proyecto Supabase
1. Crea un proyecto en https://supabase.com.
2. En SQL Editor, ejecuta `supabase/schema.sql`.
3. En Project Settings > API, copia Project URL y anon/publishable key.
4. Pega ambos valores en `assets/js/config.js`.
5. En Database > Publications, habilita `live_sessions` para Realtime (si no aparece habilitada).

## 2. Crear la primera cuenta docente
1. En Supabase > Authentication > Users, crea un usuario con correo y contraseña.
2. En SQL Editor, ejecuta (reemplaza el correo):
   ```sql
   update public.profiles p
   set role='teacher', full_name='Docente'
   from auth.users u
   where p.id=u.id and u.email='tu-correo@ejemplo.com';
   ```
3. No habilites registro público de docentes. Solo el administrador debe autorizar cuentas docentes.

## 3. Desplegar la función para códigos de estudiantes
Instala Supabase CLI e inicia sesión:
```bash
supabase login
supabase link --project-ref TU_PROJECT_REF
supabase functions deploy redeem-code
```
La función usa `SUPABASE_SERVICE_ROLE_KEY` como secreto interno del entorno Supabase. No lo copies a ningún archivo del frontend.
La función permite canjear un código una sola vez y crear la cuenta del estudiante.

## 4. Probar localmente
Por ser módulos JavaScript, no abras los HTML con `file://`. Ejecuta un servidor local:
```bash
python -m http.server 8000
```
Abre `http://localhost:8000`.

## 5. Flujo de prueba
1. Ingresa como docente.
2. Crea un aula.
3. En SQL Editor o desde una futura pantalla de gestión, genera un código:
   ```sql
   select public.create_invite_code('UUID_DEL_AULA');
   ```
   El UUID se consulta en `public.classrooms`.
4. El estudiante canjea el código en `estudiante.html` y establece su contraseña.
5. El docente selecciona el aula e inicia la clase.
6. El estudiante entra a la clase; al avanzar el docente, el contenido cambia en tiempo real.

## 6. Publicar gratis
Sube la carpeta a un repositorio GitHub y conéctalo desde Cloudflare Pages.
- Framework preset: None
- Build command: vacío
- Build output directory: `/` (raíz)
Obtendrás un subdominio gratuito `*.pages.dev`.

## Alcance de esta versión
Incluye autenticación docente, creación de aulas, canje de códigos de estudiante mediante Edge Function, panel estudiante, sesión en vivo, sincronización de diapositivas, actividad de cuaderno y una pregunta de opción múltiple con registro de respuestas.

## Pendiente para la siguiente iteración
- Interfaz docente para generar y exportar lotes de códigos.
- Panel de seguimiento de respuestas y asistencia.
- Biblioteca de materiales y tareas con subida de archivos y límites.
- Editor de presentaciones y constructor de juegos.
- Reglas de almacenamiento y validación de archivos.
- Moderación, auditoría, protección antiabuso y pruebas de carga.

## Seguridad y privacidad
- La clave `anon/publishable` puede estar en el frontend; la `service_role` nunca.
- RLS es obligatorio; no desactives políticas para "hacer que funcione".
- La cuenta docente se promueve manualmente en esta primera versión.
- La Edge Function de canje requiere endurecimiento adicional antes de uso institucional: límites de intentos, validación de origen, captcha/antibots, control de concurrencia del canje y registro de auditoría.
- Evita cargar datos personales innecesarios de menores. Informa a la institución y familias sobre el uso de la plataforma y aplica las reglas de protección de datos correspondientes.
