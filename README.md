# Bitácora de Gym

Tracker de rutina de 5 días (superior/inferior), racha, sueño, creatina y progreso de peso por ejercicio.

Los datos se sincronizan entre dispositivos con Supabase (login con correo y contraseña) — puedes usarlo desde el celular en el gym y ver lo mismo en la laptop.

## 1. Crear el proyecto en Supabase

1. Ve a [supabase.com](https://supabase.com) → crea un proyecto nuevo (gratis).
2. En **SQL Editor**, corre esto para crear la tabla donde vive tu bitácora:

```sql
create table gym_data (
  user_id uuid references auth.users not null primary key,
  state jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table gym_data enable row level security;

create policy "select_own" on gym_data
  for select using (auth.uid() = user_id);
create policy "insert_own" on gym_data
  for insert with check (auth.uid() = user_id);
create policy "update_own" on gym_data
  for update using (auth.uid() = user_id);
```

Esto asegura que solo tú (con tu sesión) puedas leer o escribir tus propios datos.

3. (Opcional pero recomendado para empezar rápido) En **Authentication → Providers → Email**, desactiva "Confirm email" para no tener que confirmar por correo cada cuenta de prueba. Puedes reactivarlo después.

4. En **Settings → API**, copia:
   - **Project URL**
   - **anon public key**

## 2. Conectar el código

Abre `index.html`, busca estas dos líneas cerca del inicio del `<script>` y reemplázalas con tus valores:

```js
const SUPABASE_URL = 'TU_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'TU_SUPABASE_ANON_KEY';
```

## 3. Deploy en GitHub Pages

```bash
cd bitacora-gym
git init
git add .
git commit -m "Primera versión con Supabase"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/bitacora-gym.git
git push -u origin main
```

Luego en GitHub: **Settings → Pages → Source** → rama `main`, carpeta `/root`. Te da una URL tipo `https://tu-usuario.github.io/bitacora-gym/`.

Desde ese momento, cada `git push` a `main` publica solo, sin pasos extra.

## 4. Primer uso

Abre la URL, dale a **Crear cuenta** con tu correo y una contraseña, inicia sesión, y listo — ya puedes entrar desde el celular y la laptop con la misma cuenta y vas a ver los mismos datos en ambos.

## Para futuros cambios de código

```bash
git add .
git commit -m "describe el cambio"
git push
```

El código se actualiza en GitHub Pages, pero tus datos siguen intactos en Supabase — viven completamente separados del código.

## Nota de seguridad

La `anon key` de Supabase queda visible en el código fuente del sitio (es pública por diseño). Lo que protege tus datos no es ocultarla, sino las políticas de **Row Level Security** del paso 1 — sin ellas, cualquiera con la key podría leer la tabla completa. No las quites.

