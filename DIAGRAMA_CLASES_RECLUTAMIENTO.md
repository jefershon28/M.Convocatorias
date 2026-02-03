# Diagrama de Clases - Sistema de Reclutamiento de Personal

Basado en la arquitectura del proyecto: **Personal** (backend) + **Personal-frontend** (frontend), con GraphQL como API.

---

## Arquitectura del proyecto

| Capa | Proyecto | Ubicación | Contenido |
|------|----------|-----------|-----------|
| **Dominio** | Personal | `src/dominio/entidades/` | RequerimientoPersonal, Empleabilidad, EmpleadoCH, Usuario, etc. |
| **Aplicación** | Personal | `src/aplicacion/servicios/` | RequerimientoPersonalService, EmpleabilidadService, EmpleadoCHService, etc. |
| **Infraestructura** | Personal | `src/infraestructura/` | GraphQL resolvers, repositorios MongoDB |
| **UI + Estado** | Personal-frontend | `src/slices/`, `src/pages/` | requerimientoPersonalSlice, empleabilidadSlice, formularios, Kanban |
| **API Client** | Personal-frontend | `src/services/` | requerimientoPersonalService (GraphQL), apolloClient |

---

## Decisiones de diseño

| Tema | Decisión |
|------|----------|
| **Convocatoria** | Solo un **estado** del RequerimientoPersonal (backend: `estado = "Convocatoria"`), no es clase separada |
| **Entrevistas** | **Subclases**: EntrevistaTelefonica, EntrevistaJefeArea, PruebaAptitudAcademica, EntrevistaGerencia |
| **Evaluación entrevistas** | Guardar **ambos**: puntaje numérico y resultado Apto/Inapto |
| **CV** | Solo URL (archivo), sin datos estructurados |
| **Estados RequerimientoPersonal** | Aprobado, Desaprobado, En espera, Archivado |
| **Empleabilidad** | Se crea **manualmente** cuando RR.HH. registra la contratación (no automático) |
| **Postulante** | **Entidad nueva** (personas nuevas); no se reutiliza EmpleadoCH |
| **Estados Postulacion** | Pendiente, Apto, Inapto, En entrevista, Archivado, Seleccionado, Contratado |
| **Área** | **Propuesta**: Usuario pertenece a Área; Jefe de Área = Usuario responsable del Área |

---

## Diagrama 1: Entidades del dominio (Backend + Propuestas)

```mermaid
classDiagram
    direction TB

    %% ============ BACKEND - EXISTENTES (Personal) ============
    class RequerimientoPersonal {
        <<Backend - Personal>>
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
        <<Backend - Personal>>
        +String id
        +String nombres
        +String apellidos
        +String rol_id
        +Object cargo_id
        +String dni
        +String usuario
        +String email
        +String telefono
    }

    class CargoCategoriaEspecialidad {
        <<Backend - Personal>>
        +String id
        +String cargo_id
        +String categoria_id
        +String especialidad_id
        +Boolean activo
    }

    class Obra {
        <<Backend - Personal>>
        +String id
        +String nombre
        +String estado
    }

    class Empresa {
        <<Backend - Personal>>
        +String id
        +String razon_social
        +String estado
    }

    class EmpleadoCH {
        <<Backend - Personal>>
        +String id
        +String dni
        +String nombres
        +String ap_paterno
        +String ap_materno
        +String celular
        +String correo_personal
        +Boolean estado
    }

    class Empleabilidad {
        <<Backend - Personal>>
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

    %% ============ PROPUESTAS (a implementar) ============
    class Area {
        <<Propuesta>>
        +String id
        +String nombre
        +String codigo
        +String responsable_id
    }

    class Postulante {
        <<Propuesta>>
        +String id
        +String nombres
        +String apellidos
        +String documento
        +String email
        +String telefono
        +Date fecha_registro
    }

    class Postulacion {
        <<Propuesta>>
        +String id
        +String requerimiento_personal_id
        +String postulante_id
        +String cv_url
        +String estado
        +Date fecha_postulacion
        +String evaluacion_cv
    }

    class Entrevista {
        <<abstract>>
        <<Propuesta>>
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
        <<Propuesta>>
    }

    class EntrevistaJefeArea {
        <<Propuesta>>
    }

    class PruebaAptitudAcademica {
        <<Propuesta>>
    }

    class EntrevistaGerencia {
        <<Propuesta>>
    }

    %% Herencia
    Entrevista <|-- EntrevistaTelefonica
    Entrevista <|-- EntrevistaJefeArea
    Entrevista <|-- PruebaAptitudAcademica
    Entrevista <|-- EntrevistaGerencia

    %% Relaciones existentes
    Usuario "1" --> "*" RequerimientoPersonal : crea
    RequerimientoPersonal "1" --> "1" CargoCategoriaEspecialidad : puesto
    RequerimientoPersonal "1" --> "1" Obra : obra
    RequerimientoPersonal "0..1" --> "1" Empresa : empresa
    Empleabilidad "1" --> "1" EmpleadoCH : empleado
    Empleabilidad "0..1" --> "1" RequerimientoPersonal : origen

    %% Relaciones propuestas
    Usuario "*" --> "0..1" Area : pertenece a
    Area "1" --> "0..1" Usuario : jefe
    RequerimientoPersonal "1" --> "0..1" Area : área
    RequerimientoPersonal "1" --> "*" Postulacion : recibe
    Postulante "1" --> "*" Postulacion : realiza
    Postulacion "1" --> "*" Entrevista : tiene
    Usuario "1" --> "*" Entrevista : evalúa
    Postulacion "0..1" --> "0..1" Empleabilidad : contratado
```

---

## Diagrama 2: Capas Backend (Personal)

```mermaid
classDiagram
    direction TB

    class RequerimientoPersonalResolver {
        <<GraphQL Resolver>>
        +listRequerimientosPersonales()
        +getRequerimientoPersonal()
        +addRequerimientoPersonal()
        +updateRequerimientoPersonal()
        +deleteRequerimientoPersonal()
    }

    class RequerimientoPersonalService {
        <<Application Service>>
        +crearRequerimientoPersonal()
        +obtenerRequerimientoPersonal()
        +actualizarRequerimientoPersonal()
        +listarRequerimientosPersonales()
        +listarRequerimientosPersonalesPaginado()
    }

    class IRequerimientoPersonalRepository {
        <<Interface>>
        +crear()
        +actualizar()
        +buscarPorId()
        +listar()
    }

    class MongoRequerimientoPersonalRepository {
        <<MongoDB>>
        +crear()
        +actualizar()
        +buscarPorId()
        +listar()
    }

    class RequerimientoPersonal {
        <<Entidad>>
    }

    RequerimientoPersonalResolver --> RequerimientoPersonalService : usa
    RequerimientoPersonalService --> IRequerimientoPersonalRepository : usa
    MongoRequerimientoPersonalRepository ..|> IRequerimientoPersonalRepository : implementa
    RequerimientoPersonalService --> RequerimientoPersonal : trabaja con
```

---

## Diagrama 3: Capas Frontend (Personal-frontend)

```mermaid
classDiagram
    direction TB

    class FormularioRequerimientoPersonal {
        <<React Component>>
        +onSubmit()
        +validar()
    }

    class VistaRequerimientoPersonal {
        <<React Component>>
        +handleCambiarEstado()
    }

    class KanbanBoardPersonal {
        <<React Component>>
        +fetchRequerimientos()
    }

    class requerimientoPersonalSlice {
        <<Redux Slice>>
        +createRequerimientoPersonal
        +updateRequerimientoPersonal
        +listRequerimientosPersonalesPaginado
    }

    class requerimientoPersonalService {
        <<Service - Apollo>>
        +createRequerimientoPersonalService()
        +updateRequerimientoPersonalService()
        +listRequerimientosPersonalesPaginadoService()
    }

    class RequerimientoPersonal {
        <<Interface / Tipo>>
        +id
        +estado
        +vacantes
        ...
    }

    FormularioRequerimientoPersonal --> requerimientoPersonalSlice : dispatch
    VistaRequerimientoPersonal --> requerimientoPersonalSlice : dispatch
    KanbanBoardPersonal --> requerimientoPersonalSlice : dispatch
    requerimientoPersonalSlice --> requerimientoPersonalService : llama
    requerimientoPersonalService --> RequerimientoPersonal : trabaja con
```

---

## Diagrama 4: Modelo simplificado (relaciones principales)

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

    Usuario "1" --> "*" RequerimientoPersonal : crea
    RequerimientoPersonal --> CargoCategoriaEspecialidad : puesto
    RequerimientoPersonal --> Obra : obra
    Empleabilidad --> EmpleadoCH : empleado
    Empleabilidad --> RequerimientoPersonal : origen
```

---

## Estados de RequerimientoPersonal

| Estado | Descripción |
|--------|-------------|
| **En espera** | Requerimiento creado; pendiente de evaluación por RR.HH. |
| **Aprobado** | RR.HH. aprobó; puede publicarse y recibir postulaciones |
| **Desaprobado** | Rechazado por RR.HH. durante la evaluación |
| **Archivado** | Archivado (ej. proceso finalizado o cerrado) |

*Nota: En el backend actual también se usan "Solicitud personal", "Aprobacion de Gerencia", "Convocatoria".*

---

## Estados de Postulacion (propuesta)

| Estado | Descripción |
|--------|-------------|
| **Pendiente** | CV recibido, pendiente de evaluación |
| **Apto** | Aprobado en evaluación de CV |
| **Inapto** | No cumple requisitos en CV |
| **En entrevista** | En proceso de entrevistas |
| **Archivado** | Rechazado en alguna etapa; archivado |
| **Seleccionado** | Ganador elegido por Gerencia |
| **Contratado** | Empleabilidad creada; ya es personal activo |

---

## Resumen: Backend (Personal) vs Propuestas

| Clase | Ubicación | Estado |
|-------|-----------|--------|
| **RequerimientoPersonal** | Personal/dominio/entidades | Existe |
| **Usuario** | Personal/dominio/entidades | Existe |
| **CargoCategoriaEspecialidad** | Personal/dominio/entidades | Existe |
| **Obra** | Personal/dominio/entidades | Existe |
| **Empresa** | Personal/dominio/entidades | Existe |
| **EmpleadoCH** | Personal/dominio/entidades | Existe |
| **Empleabilidad** | Personal/dominio/entidades | Existe |
| **RequerimientoPersonalService** | Personal/aplicacion/servicios | Existe |
| **RequerimientoPersonalResolver** | Personal/infraestructura/graphql | Existe |
| **requerimientoPersonalSlice** | Personal-frontend/slices | Existe |
| **requerimientoPersonalService** | Personal-frontend/services | Existe |
| **Area** | — | Propuesta |
| **Postulante** | — | Propuesta |
| **Postulacion** | — | Propuesta |
| **Entrevista** (y subclases) | — | Propuesta |
