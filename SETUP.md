# Libertad Financiera — Guía de instalación

Tu dashboard como **app real**: entras con correo y contraseña, y tus datos se
sincronizan solos en **todos tus dispositivos** (compu, celular, tablet).

Son **3 partes**, todas con plan **gratis**. Tiempo estimado: ~20 minutos.

- **Supabase** → guarda tus datos y maneja el login (la "nube").
- **Netlify** (o Vercel) → publica la página en internet (el "hosting").
- **index.html** → el archivo de la app (ya lo tienes en esta carpeta).

---

## PARTE 1 — Crear la base de datos en Supabase

### 1. Crea tu cuenta y proyecto
1. Entra a **https://supabase.com** y haz clic en **Start your project** (regístrate con Google o correo).
2. Clic en **New project**.
3. Ponle nombre (ej. `libertad-financiera`), crea una **Database Password** (guárdala) y elige la región más cercana (ej. *East US*).
4. Clic en **Create new project** y espera ~2 minutos a que termine de crearse.

### 2. Crea la tabla de datos
1. En el menú izquierdo, entra a **SQL Editor** → **New query**.
2. Pega **exactamente** esto y haz clic en **Run**:

```sql
create table if not exists public.user_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.user_data enable row level security;

create policy "Cada quien ve solo lo suyo (leer)"
  on public.user_data for select using (auth.uid() = user_id);
create policy "Cada quien crea solo lo suyo"
  on public.user_data for insert with check (auth.uid() = user_id);
create policy "Cada quien edita solo lo suyo"
  on public.user_data for update using (auth.uid() = user_id);
```

Debe decir **Success. No rows returned**. Eso está perfecto.

> Esto crea una tabla donde cada usuario tiene UNA fila con todos sus datos, y
> activa la seguridad (RLS) para que **nadie pueda ver los datos de otro**.

### 3. Copia tus 2 llaves
1. Menú izquierdo → **Project Settings** (el engranaje) → **API**.
2. Copia estos dos valores:
   - **Project URL** → algo como `https://abcdefgh.supabase.co`
   - **Project API keys → anon / public** → un texto largo que empieza con `eyJ...`

> La llave `anon` es **pública y segura** de poner en la página. Tus datos siguen
> protegidos porque solo se accede con login + las reglas RLS del paso 2.
> **Nunca** uses la llave `service_role` aquí.

### 4. (Recomendado) Login instantáneo sin confirmar correo
Para que al registrarte entres de una vez, sin esperar un email de confirmación:
1. Menú izquierdo → **Authentication** → **Sign In / Providers** (o **Providers → Email**).
2. **Desactiva** la opción **Confirm email** y guarda.

> Si la dejas activada, al crear cuenta tendrás que abrir el correo que te llega y
> hacer clic en el enlace antes de poder entrar. Cualquiera de las dos formas sirve.

---

## PARTE 2 — Pegar tus llaves en la app

1. Abre **index.html** (en esta carpeta) con el Bloc de notas, VS Code, o cualquier editor.
2. Hasta arriba verás este bloque:

```js
window.SUPABASE_CONFIG = {
  url: "PEGA_AQUI_TU_PROJECT_URL",
  anonKey: "PEGA_AQUI_TU_ANON_KEY"
};
```

3. Reemplaza los textos por los que copiaste en el paso 3. Debe quedar algo así:

```js
window.SUPABASE_CONFIG = {
  url: "https://abcdefgh.supabase.co",
  anonKey: "eyJhbGciOiJI...tu-llave-larga...aQ"
};
```

4. **Guarda** el archivo. (Mantén las comillas `"` y la coma).

---

## PARTE 3 — Publicar la app en internet (Netlify)

La forma más fácil, sin instalar nada:

1. Entra a **https://app.netlify.com/drop**
2. **Arrastra la carpeta `finanzas-app` completa** a esa página (o solo el archivo `index.html`).
3. En segundos te da una dirección tipo `https://algo-al-azar.netlify.app`.
4. (Opcional) Crea una cuenta gratis en Netlify para que el sitio quede permanente
   y puedas cambiarle el nombre en **Site settings → Change site name**
   (ej. `mutant-finanzas.netlify.app`).

> **Alternativa Vercel:** entra a https://vercel.com, crea cuenta, **Add New → Project**,
> y sube la carpeta. También gratis.

¡Listo! Abre esa dirección en tu celular y tu compu:
- **Crea tu cuenta** (una sola vez) con tu correo y una contraseña.
- Agrega tus movimientos en un dispositivo → aparecen en el otro al abrir. ✅

Guarda la dirección en favoritos. En el celular puedes usar **"Agregar a pantalla
de inicio"** para que se vea como una app.

---

## Preguntas frecuentes

**¿Cuánto cuesta?**
$0 en los planes gratis de Supabase y Netlify. Para uso personal te sobra
muchísimo (Supabase gratis incluye 500 MB de base de datos; tus finanzas pesan
kilobytes).

**¿Es seguro?**
Sí. Cada cuenta solo ve sus propios datos (garantizado por las reglas RLS del
paso 2). La conexión es cifrada (HTTPS).

**¿Y si pierdo internet?**
La app guarda una copia local en el navegador, así que sigue funcionando; cuando
vuelve la conexión, sincroniza. El indicador arriba a la derecha muestra el estado
(*Sincronizado / Guardando… / Sin conexión*).

**¿Cómo actualizo la app si Claude me da una versión nueva?**
Reemplaza el `index.html` (volviendo a pegar tus 2 llaves) y vuelve a arrastrarlo
a Netlify. Tus datos NO se tocan: viven en Supabase, no en el archivo.

**Olvidé mi contraseña.**
Por ahora se restablece desde Supabase → **Authentication → Users** (puedes
borrar/recrear el usuario). Si quieres, pídele a Claude que agregue el botón de
"¿Olvidaste tu contraseña?" y te lo integra.
