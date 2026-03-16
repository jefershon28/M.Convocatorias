### Modelo entidad-relación para microservicio de condiciones y subsidios

Este documento describe el modelo entidad–relación propuesto a partir del Excel compartido, con algunas mejoras pensadas desde la perspectiva de un backend senior y de un entorno de microservicios.

---

### 1. Entidad `TipoCondicion`

Catálogo maestro de tipos de condición (alimentación, movilidad, etc.).

- **Campos**
  - `id_tipo_condicion` (PK)
  - `nombre`
  - `categoria` (ej.: `ALIMENTACION`, `MOVILIDAD`, `OTROS`)
  - `tipo_calculo` (ej.: `PORCENTAJE`, `MONTO_FIJO`, `MIXTO`)
  - `monto_base` (nullable si no aplica)
  - `activo` (boolean)
  - `fecha_creacion`
  - `fecha_actualizacion`

- **Relaciones**
  - 1–N con `CondicionTrabajador`
  - 1–N con `SegmentoComercial`
  - 1–N con `Subsidio`

---

### 2. Entidad `CondicionTrabajador`

Representa la asignación de un tipo de condición a un trabajador específico.

- **Campos**
  - `id_condicion_trabajador` (PK)
  - `id_trabajador` (FK lógica a MS de personas/recursos humanos) 🟨
  - `id_tipo_condicion` (FK a `TipoCondicion`)
  - `estado` (`PENDIENTE`, `ACTIVO`, `SUSPENDIDO`, `ANULADO`, etc.)
  - `fecha_inicio_vigencia`
  - `fecha_fin_vigencia` (nullable)
  - `origen` (ej.: `MANUAL`, `SISTEMA`, `CAMPAÑA`)
  - `fecha_creacion`
  - `fecha_actualizacion`

- **Relaciones**
  - N–1 con `TipoCondicion`
  - N–1 con la entidad externa `Trabajador`

- **Mejoras respecto al Excel**
  - Separación clara entre fechas de vigencia (`fecha_inicio_vigencia`, `fecha_fin_vigencia`) y campos de auditoría.
  - `estado` tipado con un conjunto acotado de valores.

---

### 3. Entidad `SegmentoComercial`

Segmentos o reglas comerciales que agrupan condiciones bajo ciertos criterios.

- **Campos**
  - `id_segmento_comercial` (PK)
  - `codigo_segmento` (código de negocio, ej.: `SEG001`)
  - `nombre`
  - `descripcion`
  - `id_tipo_condicion` (FK a `TipoCondicion`)
  - `monto_minimo` (nullable)
  - `monto_maximo` (nullable)
  - `porcentaje` (nullable, según tipo de cálculo)
  - `fecha_inicio_vigencia`
  - `fecha_fin_vigencia`
  - `estado` (`BORRADOR`, `PENDIENTE_APROBACION`, `APROBADO`, `RECHAZADO`, `CADUCADO`)
  - `fecha_creacion`
  - `fecha_actualizacion`

- **Relaciones**
  - N–1 con `TipoCondicion`
  - 1–N con `AprobacionSegmento`

- **Mejoras respecto al Excel**
  - El segmento no depende directamente de un solo trabajador; si se requiere, se propone una tabla de asignación adicional.

---

### 4. Entidad `AprobacionSegmento`

Histórico de aprobaciones/rechazos de un segmento comercial.

- **Campos**
  - `id_aprobacion_segmento` (PK)
  - `id_segmento_comercial` (FK a `SegmentoComercial`)
  - `id_aprobador` (FK lógica a MS de usuarios/seguridad) 🟨
  - `resultado` (`APROBADO`, `RECHAZADO`)
  - `comentario` (nullable)
  - `fecha_solicitud`
  - `fecha_resolucion`

- **Relaciones**
  - N–1 con `SegmentoComercial`
  - N–1 con la entidad externa `Usuario`/`Aprobador`

- **Mejoras respecto al Excel**
  - Se separa claramente el workflow de aprobación del estado del segmento.
  - El estado actual del segmento se deriva del último registro de aprobación.

---

### 5. Entidad `Subsidio`

Corresponde a la información de `MSV_SUBSIDIOS` en el Excel, modelada como entidad propia del microservicio.

- **Campos**
  - `id_subsidio` (PK)
  - `id_servicio` (FK lógica a MS de servicios) 🟨
  - `id_cliente` (FK lógica a MS de clientes) 🟨
  - `id_persona` (FK lógica a MS de personas) 🟨
  - `id_moneda` (FK lógica a MS de monedas) 🟨
  - `nombre`
  - `descripcion`
  - `monto_subsidio`
  - `porcentaje_subsidio` (nullable, si aplica)
  - `fecha_inicio_vigencia`
  - `fecha_fin_vigencia`
  - `estado` (`PENDIENTE`, `ACTIVO`, `SUSPENDIDO`, `ANULADO`)
  - `fecha_creacion`
  - `fecha_actualizacion`

- **Relaciones**
  - Opcionalmente N–1 con `TipoCondicion` si un subsidio está asociado a un tipo de condición específico (`id_tipo_condicion` nullable).
  - N–1 con entidades externas `Servicio`, `Cliente`, `Persona`, `Moneda`.

- **Mejoras respecto al Excel**
  - Estados bien definidos y campos de vigencia.
  - Explicita los IDs externos como dependencias de otros microservicios.

---

### 6. Posibles entidades adicionales de asignación

Dependiendo de la necesidad del negocio, se pueden agregar tablas de asignación:

- **`AsignacionSegmentoTrabajador`**
  - `id_asignacion_segmento_trabajador` (PK)
  - `id_segmento_comercial` (FK)
  - `id_trabajador` (FK lógica, 🟨)
  - `fecha_inicio_vigencia`
  - `fecha_fin_vigencia`
  - `estado`

Estas entidades permiten reutilizar un mismo segmento con distintos trabajadores sin duplicar la definición del segmento.

---

### 7. Consideraciones de microservicios

- **Campos amarillos** se tratan siempre como **FK lógicas** a otros microservicios (solo IDs).
- Las descripciones o nombres legibles se deben:
  - Resolver en tiempo de consulta llamando a los otros MS, o
  - Cachear localmente en este microservicio con una estrategia de refresco, pero nunca como fuente de verdad.
- Definir índices en:
  - Todas las FKs (`id_trabajador`, `id_tipo_condicion`, `id_segmento_comercial`, `id_servicio`, `id_cliente`, `id_persona`, `id_moneda`).
  - Campos de búsqueda frecuentes como `estado`, `fecha_inicio_vigencia`, `fecha_fin_vigencia`.

Este modelo sirve como base para implementar las entidades del dominio en tu ORM/base de datos y para diseñar los endpoints del microservicio.

