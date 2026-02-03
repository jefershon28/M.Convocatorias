# Diagrama de Secuencias - Sistema de Reclutamiento de Personal

Basado en el flujo del proceso, la arquitectura del proyecto (**Personal-frontend** + **Personal**) y la comunicación vía **GraphQL**.

---

## Arquitectura del sistema

| Capa | Proyecto | Tecnología |
|------|----------|------------|
| **Frontend** | Personal-frontend | React, Redux, Apollo Client |
| **Backend API** | Personal | GraphQL (resolvers) |
| **Lógica de negocio** | Personal | Servicios de aplicación |
| **Persistencia** | Personal | MongoDB (repositorios) |

---

## 1. Crear y evaluar requerimiento

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Frontend
    participant API as GraphQL API
    participant SVC as RequerimientoPersonalService
    participant DB as MongoDB

    U->>FE: Crear requerimiento (formulario)
    FE->>FE: Validar datos
    FE->>API: mutation addRequerimientoPersonal(input)
    API->>SVC: crearRequerimientoPersonal()
    SVC->>SVC: RequerimientoPersonal.crear()
    SVC->>DB: save RequerimientoPersonal
    DB-->>SVC: OK
    SVC-->>API: RequerimientoPersonal
    API-->>FE: RequerimientoPersonal (estado: En espera)
    FE-->>U: Confirmación

    Note over U,DB: Evaluación por RR.HH.

    U->>FE: Evaluar requerimiento (aprobar/desaprobar)
    FE->>API: mutation updateRequerimientoPersonal(id, estado)
    API->>SVC: actualizarRequerimientoPersonal()
    SVC->>DB: update estado
    alt Aprobado
        DB-->>SVC: OK
        SVC-->>API: RequerimientoPersonal (Aprobado)
        API-->>FE: RequerimientoPersonal
        FE-->>U: Requerimiento aprobado
    else Desaprobado / Archivado
        SVC-->>API: RequerimientoPersonal
        API-->>FE: RequerimientoPersonal
        FE-->>U: Requerimiento archivado
    end
```

---

## 2. Publicar convocatoria y postulación

```mermaid
sequenceDiagram
    participant RRHH as RR.HH.
    participant P as Postulante
    participant FE as Frontend
    participant API as GraphQL API
    participant SVC as Backend Services
    participant DB as MongoDB
    participant F as Foros de Empleo

    RRHH->>FE: Cambiar estado a Convocatoria
    FE->>API: mutation updateRequerimientoPersonal(estado: "Convocatoria")
    API->>SVC: actualizarRequerimientoPersonal()
    SVC->>DB: update estado
    DB-->>SVC: OK
    SVC-->>API: RequerimientoPersonal
    API-->>FE: OK
    FE-->>RRHH: Convocatoria publicada

    opt Publicación externa
        RRHH->>F: Publicar en foros (manual o integración)
    end

    P->>FE: Ver convocatorias / Postularse
    FE->>API: query listRequerimientosPersonales(estado: "Convocatoria")
    API->>SVC: listarRequerimientosPersonales()
    SVC->>DB: find
    DB-->>SVC: requerimientos
    SVC-->>API: RequerimientoPersonal[]
    API-->>FE: lista

    P->>FE: Enviar CV (formulario postulación)
    FE->>API: mutation addPostulacion(input) / addPostulante(input)
    Note over API: Postulante y Postulacion (cuando existan)
    API->>SVC: crearPostulacion()
    SVC->>DB: save
    DB-->>SVC: OK
    SVC-->>API: Postulacion
    API-->>FE: OK
    FE-->>P: Confirmación
```

---

## 3. Evaluar CVs y programar entrevista telefónica

```mermaid
sequenceDiagram
    participant RRHH as RR.HH.
    participant FE as Frontend
    participant API as GraphQL API
    participant SVC as Backend Services
    participant DB as MongoDB
    participant P as Postulante

    RRHH->>FE: Consultar postulaciones
    FE->>API: query listPostulaciones(requerimiento_id, estado)
    API->>SVC: listarPostulaciones()
    SVC->>DB: find Postulacion + Postulante
    DB-->>SVC: datos
    SVC-->>API: Postulacion[]
    API-->>FE: Lista de CVs
    FE-->>RRHH: Mostrar CVs

    loop Por cada postulante
        RRHH->>FE: Evaluar CV (Apto/Inapto)
        FE->>API: mutation updatePostulacion(id, estado)
        API->>SVC: actualizarPostulacion()
        SVC->>DB: update estado
        DB-->>SVC: OK
        SVC-->>API: Postulacion
    end
    API-->>FE: OK
    FE-->>RRHH: Evaluación guardada

    RRHH->>FE: Programar entrevista telefónica
    FE->>API: mutation addEntrevista(postulacion_id, tipo: "Telefonica")
    API->>SVC: crearEntrevista()
    SVC->>DB: save EntrevistaTelefonica
    DB-->>SVC: OK
    SVC-->>API: Entrevista
    API-->>FE: OK
    FE->>P: Notificar (email/sistema)
    FE-->>RRHH: Entrevista programada
```

---

## 4. Proceso de entrevistas

```mermaid
sequenceDiagram
    participant RRHH as RR.HH.
    participant JA as Jefe de Área
    participant G as Gerencia
    participant FE as Frontend
    participant API as GraphQL API
    participant SVC as Backend Services
    participant DB as MongoDB

    Note over RRHH,DB: Etapa 1: Entrevista telefónica
    RRHH->>FE: Registrar resultado (Apto/Inapto, puntaje)
    FE->>API: mutation updateEntrevista(id, resultado, puntaje)
    API->>SVC: actualizarEntrevista() + actualizarPostulacion()
    SVC->>DB: update
    DB-->>SVC: OK
    SVC-->>API: OK
    API-->>FE: OK

    Note over RRHH,DB: Etapa 2: Entrevista Jefe de Área
    JA->>FE: Programar / Registrar resultado
    FE->>API: mutation addEntrevista / updateEntrevista
    API->>SVC: crearEntrevista(tipo: JefeArea)
    SVC->>DB: save
    DB-->>SVC: OK
    API-->>FE: OK

    Note over RRHH,DB: Etapa 3: Prueba aptitud académica
    JA->>FE: Registrar resultado prueba
    FE->>API: mutation updateEntrevista(tipo: PruebaAptitudAcademica)
    API->>SVC: actualizarEntrevista()
    SVC->>DB: update
    DB-->>SVC: OK
    API-->>FE: OK

    Note over RRHH,DB: Etapa 4: Entrevista Gerencia
    G->>FE: Registrar resultado final
    FE->>API: mutation updateEntrevista(tipo: Gerencia)
    API->>SVC: actualizarEntrevista()
    SVC->>DB: update
    DB-->>SVC: OK
    API-->>FE: OK
```

---

## 5. Selección y contratación

```mermaid
sequenceDiagram
    participant G as Gerencia
    participant RRHH as RR.HH.
    participant FE as Frontend
    participant API as GraphQL API
    participant SVC as Backend Services
    participant DB as MongoDB

    G->>FE: Seleccionar candidato ganador
    FE->>API: mutation updatePostulacion(id, estado: "Seleccionado")
    API->>SVC: actualizarPostulacion()
    SVC->>DB: update Postulacion (Seleccionado)
    SVC->>DB: update otras Postulacion (Archivado)
    DB-->>SVC: OK
    SVC-->>API: OK
    API-->>FE: OK
    FE-->>G: Candidato seleccionado

    RRHH->>FE: Registrar contratación
    FE->>API: mutation addEmpleadoCH(input) + addEmpleabilidad(input)
    API->>SVC: crearEmpleadoCH(desde Postulante)
    SVC->>DB: save EmpleadoCH
    API->>SVC: crearEmpleabilidad()
    SVC->>DB: save Empleabilidad
    SVC->>DB: update Postulacion (Contratado)
    SVC->>DB: update vacantes_disponibles
    DB-->>SVC: OK
    SVC-->>API: Empleabilidad
    API-->>FE: OK
    FE-->>RRHH: Contratación registrada
```

---

## 6. Flujo completo (visión sistema)

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Frontend
    participant BE as Backend

    U->>FE: 1. Crear requerimiento
    FE->>BE: addRequerimientoPersonal
    BE-->>FE: RequerimientoPersonal

    U->>FE: 2. Evaluar (RRHH)
    FE->>BE: updateRequerimientoPersonal(estado)
    BE-->>FE: OK

    alt Aprobado
        U->>FE: 3. Publicar convocatoria
        FE->>BE: updateRequerimientoPersonal(Convocatoria)
        U->>FE: 4. Postulante envía CV
        FE->>BE: addPostulacion
        U->>FE: 5-8. Evaluar CVs + Entrevistas
        FE->>BE: updatePostulacion / addEntrevista
        U->>FE: 9. Gerencia selecciona
        FE->>BE: updatePostulacion(Seleccionado)
        U->>FE: 10. RRHH registra contratación
        FE->>BE: addEmpleadoCH + addEmpleabilidad
    else Rechazado
        FE->>BE: updateRequerimientoPersonal(Archivado)
    end
```

---

## Resumen de diagramas

| # | Diagrama | Participantes | Descripción |
|---|----------|---------------|-------------|
| 1 | Crear y evaluar requerimiento | Usuario, Frontend, GraphQL API, Service, MongoDB | Flujo completo con capas Frontend ↔ Backend ↔ DB |
| 2 | Publicar convocatoria y postulación | RRHH, Postulante, Frontend, API, Services, DB, Foros | Publicación y recepción de postulaciones |
| 3 | Evaluar CVs | RRHH, Frontend, API, Services, DB, Postulante | Evaluación de CVs y programación de entrevista telefónica |
| 4 | Proceso de entrevistas | RRHH, Jefe Área, Gerencia, Frontend, API, Services, DB | Las 4 etapas con interacción Frontend-Backend |
| 5 | Selección y contratación | Gerencia, RRHH, Frontend, API, Services, DB | Selección y registro de contratación |
| 6 | Flujo completo | Usuario, Frontend, Backend | Visión resumida del flujo |

---

## Notas técnicas

- **Frontend**: Redux (slices) despacha acciones que llaman a los servicios (Apollo Client).
- **GraphQL**: Queries para leer, Mutations para crear/actualizar/eliminar.
- **Backend**: Resolvers → Servicios de aplicación → Repositorios → MongoDB.
- **Entidades Postulante, Postulacion, Entrevista**: Diagramas asumen que existirán en el backend; actualmente el proyecto tiene RequerimientoPersonal, Empleabilidad, EmpleadoCH.
