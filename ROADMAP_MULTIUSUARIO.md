# Roadmap futuro: Plataforma entrenador-alumno

## 1. Vision

La app actual es una ficha local-first para una sola persona, con carga mensual desde Excel y datos guardados en `localStorage`.

La evolucion futura seria convertirla en una plataforma donde:

- El entrenador crea, edita y asigna rutinas desde un panel web.
- El alumno entra con su usuario y ve solo sus rutinas asignadas.
- El alumno registra cargas, repeticiones, RIR, RPE y notas desde el celular.
- El entrenador puede revisar progreso, adherencia y comentarios.
- Ya no es necesario importar Excel para cargar entrenamientos.

El objetivo no es perder la simpleza de la ficha actual, sino llevarla a nube y multiusuario de forma ordenada.

## 2. Roles

### Alumno

Acciones principales:

- Iniciar sesion.
- Ver rutina activa.
- Seleccionar Dia 1, Dia 2 o Dia 3.
- Registrar datos por ejercicio y semana.
- Ver estadisticas personales.
- Exportar datos si se mantiene una opcion de backup.
- Consultar explicaciones tecnicas de ejercicios e indicaciones.

Restricciones:

- No puede editar la rutina base asignada.
- No puede ver datos de otros alumnos.

### Entrenador

Acciones principales:

- Iniciar sesion.
- Crear y administrar alumnos.
- Crear rutinas desde un editor propio.
- Asignar rutinas por mes o bloque.
- Ver registros de entrenamiento de cada alumno.
- Ver estadisticas y tendencias.
- Ajustar rutinas del mes siguiente sin depender de Excel.
- Crear o editar biblioteca de ejercicios.
- Crear o editar metodos de entrenamiento.

Restricciones:

- Solo ve alumnos vinculados a su cuenta.
- Los cambios de rutina deben versionarse para no borrar historial.

### Admin futuro opcional

Si el sistema escala a varios entrenadores:

- Gestionar entrenadores.
- Gestionar planes/pagos.
- Ver metricas generales.
- Resolver soporte.

No es necesario para la primera version multiusuario.

## 3. Stack recomendado

### Frontend

Opcion inicial recomendada:

- React + Vite o Next.js.
- Tailwind CSS.
- Deploy en Vercel.
- PWA opcional para experiencia instalable en celular.

Motivo:

- La app actual ya es SPA.
- Vercel funciona bien para frontends.
- React facilita paneles, formularios y estados complejos.

### Backend y base de datos

Opcion recomendada:

- Supabase.

Servicios necesarios:

- Supabase Auth para usuarios.
- Postgres para datos relacionales.
- Row Level Security para aislar datos por usuario/entrenador.
- Storage opcional para archivos, imagenes o backups.

Motivo:

- Evita construir backend propio al inicio.
- Tiene auth, DB y permisos en una misma plataforma.
- El modelo relacional encaja bien con alumnos, rutinas, ejercicios y registros.

Alternativas:

- Firebase: buena opcion, especialmente para tiempo real, pero menos natural para relaciones complejas.
- Backend propio Node/Laravel: mas control, mas mantenimiento.

## 4. Modelo de datos propuesto

### users / profiles

Supabase Auth maneja el usuario base. Se agrega una tabla `profiles`.

Campos:

- `id`: UUID, igual al auth user id.
- `email`.
- `full_name`.
- `role`: `trainer`, `student`, `admin`.
- `created_at`.

### trainer_students

Relacion entre entrenador y alumno.

Campos:

- `id`.
- `trainer_id`.
- `student_id`.
- `status`: `active`, `paused`, `archived`.
- `created_at`.

Permite que un entrenador tenga varios alumnos.

### training_blocks

Representa un bloque, mes o plan asignado.

Campos:

- `id`.
- `trainer_id`.
- `student_id`.
- `name`: ejemplo `Mayo 2026`.
- `start_date`.
- `end_date`.
- `status`: `draft`, `active`, `archived`.
- `created_at`.
- `updated_at`.

Regla:

- Nunca borrar bloques con historial.
- Si se modifica una rutina ya usada, crear version o guardar cambios con auditoria.

### training_days

Dias dentro del bloque.

Campos:

- `id`.
- `block_id`.
- `name`: `Dia 1`, `Dia 2`, `Dia 3`.
- `focus`: ejemplo `Cuadriceps - Espalda - Biceps`.
- `order_index`.

### exercises_library

Biblioteca general de ejercicios.

Campos:

- `id`.
- `trainer_id` nullable si es global.
- `name`.
- `category`: pecho, espalda, pierna, hombro, brazo, core, etc.
- `equipment`: barra, mancuerna, polea, maquina, peso corporal, etc.
- `setup_notes`.
- `execution_notes`.
- `common_mistakes`.
- `video_url` opcional.
- `created_at`.

La ayuda tecnica actual del boton `?` deberia migrar a esta biblioteca.

### routine_items

Ejercicios asignados dentro de un dia.

Campos:

- `id`.
- `day_id`.
- `exercise_id` opcional.
- `exercise_name_snapshot`.
- `group_name`.
- `week_1_prescription`.
- `week_2_prescription`.
- `week_3_prescription`.
- `week_4_prescription`.
- `pause`.
- `method_notes`.
- `order_index`.

Se guarda `exercise_name_snapshot` para preservar historial aunque luego cambie el nombre en biblioteca.

### warmup_items

Entrada en calor por bloque o dia.

Campos:

- `id`.
- `block_id`.
- `day_id` nullable.
- `name`.
- `instructions`.
- `order_index`.

### methods

Metodos de entrenamiento.

Campos:

- `id`.
- `trainer_id` nullable.
- `title`.
- `description`.
- `created_at`.

### workout_logs

Registros cargados por el alumno.

Campos:

- `id`.
- `student_id`.
- `block_id`.
- `routine_item_id`.
- `week_number`: 1 a 4.
- `kg`.
- `reps`.
- `rir`.
- `rpe`.
- `notes`.
- `performed_at`.
- `created_at`.
- `updated_at`.

Regla:

- Un alumno puede tener como maximo un log por ejercicio/semana, salvo que luego se agregue soporte para multiples series reales.

### exercise_comments futuro opcional

Comentarios entre entrenador y alumno.

Campos:

- `id`.
- `routine_item_id`.
- `student_id`.
- `trainer_id`.
- `author_id`.
- `message`.
- `created_at`.

## 5. Flujo del entrenador

### Version 1 multiusuario

1. El entrenador inicia sesion.
2. Crea un alumno o invita por email.
3. Crea un bloque de entrenamiento, por ejemplo `Mayo 2026`.
4. Agrega Dia 1, Dia 2 y Dia 3.
5. Agrega ejercicios a cada dia.
6. Completa indicaciones por semana, pausa y notas.
7. Publica el bloque.
8. El alumno ve el bloque activo en su celular.

### Editor de rutina

Debe permitir:

- Agregar ejercicio.
- Elegir desde biblioteca o escribir nombre libre.
- Ordenar ejercicios.
- Agrupar por foco muscular.
- Cargar Semana 1 a Semana 4.
- Cargar pausa.
- Cargar metodo o nota.
- Duplicar dia.
- Duplicar bloque anterior para crear el mes siguiente.

La accion mas importante para ahorrar tiempo al entrenador:

- `Duplicar mes anterior` y ajustar solo lo necesario.

## 6. Flujo del alumno

1. El alumno inicia sesion.
2. Ve su bloque activo.
3. Selecciona Dia 1, Dia 2 o Dia 3.
4. Abre cada ejercicio colapsable.
5. Consulta el boton `?` si necesita tecnica o explicacion.
6. Carga kg, reps, RIR, RPE y nota.
7. Los datos se guardan en nube.
8. Las estadisticas se actualizan automaticamente.

La experiencia mobile-first actual debe conservarse.

## 7. Estadisticas para entrenador

Metricas utiles:

- Evolucion de kg por ejercicio.
- Evolucion de reps por ejercicio.
- Volumen por ejercicio: `kg * reps`.
- 1RM estimado.
- RPE/RIR promedio.
- Ejercicios con retroceso.
- Ejercicios sin datos cargados.
- Adherencia por semana.
- Comparacion entre bloques.
- Notas recientes del alumno.

Vista del entrenador:

- Dashboard por alumno.
- Semaforo simple:
  - verde: progreso o cumplimiento correcto.
  - amarillo: estancamiento o datos incompletos.
  - rojo: retroceso marcado, dolor reportado o falta de carga.

## 8. Migracion desde la app actual

La app actual usa `localStorage`.

Para migrar:

1. Mantener exportacion JSON.
2. Crear importador de backup en la nueva app.
3. Convertir:
   - `months` a `training_blocks`.
   - `routine` a `training_days` + `routine_items`.
   - `warmup` a `warmup_items`.
   - `methods` a `methods`.
   - `logs` a `workout_logs`.
4. Asociar todo al usuario alumno autenticado.

Esto permite no perder el historial local cuando se pase a nube.

## 9. Seguridad y permisos

Reglas indispensables:

- Un alumno solo puede leer/escribir sus propios logs.
- Un alumno solo puede leer rutinas asignadas a el.
- Un entrenador solo puede ver alumnos vinculados.
- Un entrenador puede crear/editar rutinas de sus alumnos.
- Los logs historicos no deben borrarse al cambiar una rutina.
- Usar Row Level Security en Supabase.

## 10. Roadmap por etapas

### Etapa 0: App actual

Estado:

- SPA local-first.
- Importacion Excel.
- LocalStorage.
- Backup JSON.
- Deploy Vercel.

### Etapa 1: Preparar arquitectura cloud

Objetivo:

- Crear proyecto Supabase.
- Definir tablas.
- Configurar Auth.
- Configurar RLS.
- Crear variables de entorno en Vercel.

Entregable:

- App con login y perfil.
- Sin panel complejo todavia.

### Etapa 2: Alumno con datos en nube

Objetivo:

- Mover rutinas y logs a Supabase.
- Mantener UI mobile-first actual.
- Importar backup JSON o Excel para cargar datos iniciales.

Entregable:

- Alumno puede iniciar sesion y ver sus datos desde cualquier dispositivo.

### Etapa 3: Panel basico del entrenador

Objetivo:

- Crear alumnos.
- Ver lista de alumnos.
- Crear bloque de entrenamiento.
- Asignar rutina a alumno.
- Ver logs del alumno.

Entregable:

- El entrenador deja de depender del Excel para nuevas rutinas.

### Etapa 4: Editor avanzado de rutinas

Objetivo:

- Biblioteca de ejercicios.
- Duplicar mes anterior.
- Reordenar ejercicios.
- Plantillas.
- Metodos reutilizables.

Entregable:

- Carga de rutina mas rapida que Excel.

### Etapa 5: Analitica y comunicacion

Objetivo:

- Dashboard de progreso.
- Alertas de retroceso o dolor.
- Comentarios entrenador-alumno.
- Reportes mensuales.

Entregable:

- Herramienta de seguimiento profesional.

### Etapa 6: Escala comercial opcional

Objetivo:

- Multiples entrenadores.
- Pagos.
- Planes.
- Panel admin.
- Branding por entrenador.

No avanzar a esta etapa hasta validar uso real con al menos un entrenador y varios alumnos.

## 11. Decisiones pendientes

- Si el entrenador sera el unico entrenador del sistema o habra multiples entrenadores.
- Si los alumnos se crean manualmente o por invitacion email.
- Si se necesita chat interno o solo comentarios por ejercicio.
- Si se quieren videos propios por ejercicio.
- Si se mantiene importacion Excel como herramienta auxiliar.
- Si cada semana tendra un solo registro por ejercicio o multiples series reales.
- Si se desea soporte offline con sincronizacion posterior.

## 12. Recomendacion

La mejor ruta es:

1. Mantener esta app local-first como MVP personal.
2. Validar el uso real durante algunos entrenamientos.
3. Cuando el flujo este comodo, migrar a Supabase.
4. Construir primero el login y persistencia cloud.
5. Luego construir el panel del entrenador.

No conviene empezar por el panel complejo sin antes confirmar que la ficha mobile del alumno sea realmente comoda durante el entrenamiento.
