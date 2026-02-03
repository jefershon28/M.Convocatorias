# Diagrama de Clases - Sistema de Reclutamiento de Personal

Basado en el diagrama de casos de uso, las entidades del backend y las decisiones de diseño acordadas.

---

## Decisiones de diseño

| Tema | Decisión |
|------|----------|
| **Convocatoria** | Solo un **estado** del RequerimientoPersonal (backend: `estado = "Convocatoria"`), no es clase separada |
| **Entrevistas** | **Subclases**: EntrevistaTelefonica, EntrevistaJefeArea, PruebaAptitudAcademica, EntrevistaGerencia |
| **Evaluación entrevistas** | Guardar **ambos**: puntaje numérico y resultado Apto/Inapto |
| **CV** | Solo URL (archivo), sin datos estructurados |
| **Estados RequerimientoPersonal** | Solicitud personal → Aprobación de Gerencia → Convocatoria (y Rechazado, Cancelado, Suspendido) |
| **Empleabilidad** | Se crea **manualmente** cuando RR.HH. registra la contratación (no automático) |

---

## Diagrama de clases completo (Mermaid)

```mermaid
classDiagram
    direction TB

    %% ============ ENTIDADES EXISTENTES ============
    class RequerimientoPersonal {
        +String id
        +String cargo_categoria_especialidad_id
        +String usuario_id
        +String obra_id
        +String empresa_id
        +Object requisitos
        +Number vacantes
        +Number vacantes_disponibles
        +String prioridad
        +String descripcion
        +String estado
        +Date fecha_creacion
        +Date fecha_actualizacion
        +Date fecha_inicio
        +Date fecha_fin
    }

    class Usuario {
        +String id
        +String nombres
        +String apellidos
        +String rol_id
        +String dni
        +String usuario
        +String email
        +String telefono
    }

    class CargoCategoriaEspecialidad {
        +String id
        +String cargo_id
        +String categoria_id
        +String especialidad_id
        +Boolean activo
    }

    class Obra {
        +String id
        +String nombre
        +String estado
    }

    class Empresa {
        +String id
        +String razon_social
        +String estado
    }

    class EmpleadoCH {
        +String id
        +String dni
        +String nombres
        +String ap_paterno
        +String ap_materno
        +String celular
        +String correo_personal
        +Date fecha_nacimiento
        +Boolean estado
    }

    class Empleabilidad {
        +String id
        +String empleadoch_id
        +String requerimiento_personal_id
        +String cargo_categoria_especialidad_id
        +String empresa_id
        +String obra_id
        +String tipo_personal
        +Date fecha_ingreso
        +Date fecha_inicio
        +String estado
        +Boolean activo
    }

    %% ============ ENTIDADES PROPUESTAS ============
    class Postulante {
        +String id
        +String nombres
        +String apellidos
        +String documento
        +String email
        +String telefono
        +Date fecha_registro
    }

    class Postulacion {
        +String id
        +String requerimiento_personal_id
        +String postulante_id
        +String cv_url
        +String estado
        +Date fecha_postulacion
        +String evaluacion_cv
    }

    %% Entrevista base + subclases
    class Entrevista {
        <<abstract>>
        +String id
        +String postulacion_id
        +String evaluador_id
        +Date fecha_programada
        +Date fecha_realizada
        +String resultado
        +Number puntaje
        +String observaciones
    }

    class EntrevistaTelefonica {
    }

    class EntrevistaJefeArea {
    }

    class PruebaAptitudAcademica {
    }

    class EntrevistaGerencia {
    }

    %% ============ HERENCIA ENTREVISTAS ============
    Entrevista <|-- EntrevistaTelefonica
    Entrevista <|-- EntrevistaJefeArea
    Entrevista <|-- PruebaAptitudAcademica
    Entrevista <|-- EntrevistaGerencia

    %% ============ RELACIONES ============
    Usuario "1" --> "*" RequerimientoPersonal : crea
    RequerimientoPersonal "1" --> "1" CargoCategoriaEspecialidad : puesto
    RequerimientoPersonal "1" --> "1" Obra : obra
    RequerimientoPersonal "0..1" --> "1" Empresa : empresa
    Empleabilidad "1" --> "1" EmpleadoCH : empleado
    Empleabilidad "0..1" --> "1" RequerimientoPersonal : origen

    RequerimientoPersonal "1" --> "*" Postulacion : recibe
    Postulante "1" --> "*" Postulacion : realiza
    Postulacion "1" --> "*" Entrevista : tiene
    Usuario "1" --> "*" Entrevista : evalúa
    Postulacion "0..1" --> "0..1" Empleabilidad : contratado
```

---

## Diagrama simplificado

```mermaid
classDiagram
    RequerimientoPersonal "1" --> "*" Postulacion : recibe
    Postulante "1" --> "*" Postulacion : envía
    Postulacion "1" --> "*" Entrevista : tiene
    Usuario "1" --> "*" Entrevista : evalúa
    Postulacion "0..1" --> "0..1" Empleabilidad : contratado

    Entrevista <|-- EntrevistaTelefonica
    Entrevista <|-- EntrevistaJefeArea
    Entrevista <|-- PruebaAptitudAcademica
    Entrevista <|-- EntrevistaGerencia

    RequerimientoPersonal : +id
    RequerimientoPersonal : +estado
    RequerimientoPersonal : +vacantes

    Postulacion : +id
    Postulacion : +estado
    Postulacion : +cv_url

    Entrevista : +resultado
    Entrevista : +puntaje
```

---

## Estados de RequerimientoPersonal

| Estado | Descripción |
|--------|-------------|
| **Solicitud personal** | Requerimiento creado, pendiente de evaluación |
| **Aprobacion de Gerencia** | En revisión por gerencia |
| **Convocatoria** | Aprobado y publicado; recibe postulaciones |
| **Solicitud personal Rechazado** | Rechazado, archivado |
| **Cancelado** | Cancelado |
| **Suspendido** | Suspendido temporalmente |

---

## Clases del diagrama

| Clase | Estado | Descripción |
|-------|--------|-------------|
| **RequerimientoPersonal** | Existe en backend | Solicitud de personal. Estados: Solicitud personal → Aprobación → Convocatoria |
| **Usuario** | Existe en backend | Actores: Gerencia, RR.HH., Jefe de Área |
| **CargoCategoriaEspecialidad** | Existe en backend | Puesto/cargo solicitado |
| **Obra** | Existe en backend | Proyecto u obra asociada |
| **EmpleadoCH** | Existe en backend | Persona en base de datos |
| **Empleabilidad** | Existe en backend | Alta como personal activo (contratación manual) |
| **Postulante** | Propuesta | Persona que envía CV |
| **Postulacion** | Propuesta | Postulación a un requerimiento en estado Convocatoria. CV = `cv_url` |
| **Entrevista** | Propuesta (abstracta) | Base para las 4 etapas de evaluación |
| **EntrevistaTelefonica** | Propuesta | Evaluación telefónica (RR.HH.) |
| **EntrevistaJefeArea** | Propuesta | Entrevista con jefe de área |
| **PruebaAptitudAcademica** | Propuesta | Prueba de aptitud (Jefe de Área) |
| **EntrevistaGerencia** | Propuesta | Entrevista con gerencia (decisión final) |

---

## Preguntas pendientes

- **Postulante vs EmpleadoCH**: ¿El Postulante es entidad nueva o se reutiliza EmpleadoCH?
- **Estados de Postulacion**: ¿Pendiente, Apto, Inapto, Archivado, Seleccionado?
- **Área / Jefe de Área**: ¿Existe entidad Área o solo Usuario?
