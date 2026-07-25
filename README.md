# Seguimiento de Pedidos Abiertos

App de una sola página, conectada en vivo a Supabase. No requiere backend ni build:
es HTML + JS puro, listo para desplegar como sitio estático.

## Archivos

- `index.html` — la app completa (filtros, tabla, edición en vivo de Fecha Atención/Comentarios).
- `config.js` — credenciales de Supabase (Project URL + Publishable key). Ya vienen cargadas.
- `vercel.json` — configuración mínima para Vercel.
- `supabase_proveedores.sql` — SQL para crear la tabla `proveedores` (nombre corto por RUC),
  que faltaba en el esquema original. Córrelo una vez en el SQL Editor de Supabase si
  todavía no lo hiciste, y sube `proveedores.csv` a esa tabla.

## Subir a GitHub

```bash
git init
git add .
git commit -m "Reporte de pedidos en tránsito conectado a Supabase"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
git push -u origin main
```

(O sube los archivos directo desde la interfaz web de GitHub: "Add file" → "Upload files".)

## Desplegar en Vercel

1. Entra a vercel.com → **Add New… → Project**.
2. Importa el repositorio de GitHub que acabas de crear.
3. No necesitas cambiar ningún ajuste de build (es un sitio estático) → **Deploy**.
4. Te da una URL tipo `tu-proyecto.vercel.app` — esa es la que compartes con tu equipo
   y con tus proveedores.

## Si cambias de proyecto Supabase

Edita `config.js` con la nueva Project URL y Publishable key, y vuelve a hacer
`git push` — Vercel redespliega solo.

## Cargar pedidos nuevos (rutina diaria/semanal)

La tabla `pedidos` tiene una restricción única en (`documento_compras`, `material`),
así que el importador CSV de Supabase no puede "actualizar" filas existentes, solo
insertar nuevas. Para refrescar con un exportado de SAP más reciente:

1. En el SQL Editor de Supabase: `truncate pedidos, seguimiento;`
   (esto también borra el seguimiento cargado — solo hazlo si tu nuevo export ya
   trae todas las líneas vigentes y no te importa perder el historial de seguimiento).
2. Exporta tu SAP a CSV con las columnas: `fecha_documento, documento_compras, material,
   texto_breve, centro, proveedor_texto, cantidad_pedido, cantidad_pendiente`.
3. Table Editor → `pedidos` → Insert → Import data from CSV.

Si prefieres no perder el seguimiento cada vez, avísame y armamos una función que
haga upsert real en vez de truncar.

## Seguridad

- La `Publishable key` en `config.js` es segura de exponer públicamente: las tablas
  tienen Row Level Security activo, con lectura pública y escritura pública SOLO en
  la tabla `seguimiento`.
- Nunca pongas la `Secret key` de Supabase en ningún archivo de este repo.
