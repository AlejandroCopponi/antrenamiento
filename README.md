# Ficha de Entrenamiento

SPA local-first para registrar entrenamiento desde el celular.

## Archivos

- `index.html`: app completa con HTML, CSS y JavaScript integrados.
- `AGENT.md`: especificacion del proyecto.
- `ALEJANDRO COPPONI 1 (1).xlsx`: Excel de ejemplo del entrenador.

## Probar localmente

Abrir `index.html` en el navegador.

La app usa CDN para Tailwind, SheetJS y Chart.js, por lo que necesita internet para cargar esas librerias.

## Uso

1. Entrar a la pestana `Datos`.
2. Importar el Excel del entrenador.
3. Elegir la hoja de rutina detectada.
4. Escribir el mes, por ejemplo `Mayo 2026`.
5. Confirmar la importacion.
6. Registrar kilos, reps, RIR, RPE y notas desde la pestana `Rutina`.

Los datos quedan guardados en `localStorage` del navegador.

## Backup

Desde `Datos`:

- `Exportar`: descarga un JSON con toda la informacion local.
- `Importar`: reemplaza los datos locales con un backup JSON previo.

## Deploy en Vercel

La app es estatica y puede subirse gratis a Vercel.

Flujo recomendado:

1. Crear un repositorio GitHub con estos archivos.
2. Subir `index.html`, `AGENT.md`, `README.md` y el Excel si se quiere conservar como muestra.
3. En Vercel, elegir `Add New Project`.
4. Importar el repositorio.
5. Framework Preset: `Other`.
6. Build Command: dejar vacio.
7. Output Directory: dejar vacio o `.`.
8. Deploy.

Importante: aunque la app este online, los datos de entrenamiento quedan guardados localmente en el celular donde se use.
