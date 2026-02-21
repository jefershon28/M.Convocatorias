# Diagrama de Estados - MS. Condiciones, Anticipos y Descuentos

Ciclo de vida de las entidades principales del sistema en formato Mermaid (compatible con GitHub).

---

## 1. Ciclo de vida de la Solicitud

Las solicitudes (condiciones, préstamos, adelantos, descuentos) atraviesan un flujo de aprobación gestionado en el Kanban.

```mermaid
stateDiagram-v2
    [*] --> Registrada: RRHH registra solicitud

    Registrada --> En_revision: RRHH mueve a revisión
    En_revision --> Registrada: RRHH devuelve a registrada
    En_revision --> Aprobada: RRHH aprueba
    En_revision --> Rechazada: RRHH rechaza (ingresa motivo)

    Aprobada --> [*]
    Rechazada --> [*]
```

| Estado       | Descripción                                                      |
|-------------|------------------------------------------------------------------|
| **Registrada**   | Solicitud creada, esperando ser tomada en revisión                |
| **En_revision**  | RRHH está evaluando la solicitud en el Kanban                     |
| **Aprobada**     | Solicitud aprobada; la condición/operación se activa             |
| **Rechazada**    | Solicitud rechazada; se registra motivo, no procede              |

---

## 2. Ciclo de vida de Condición / Operación

Condiciones asignadas a trabajadores y operaciones (préstamos, adelantos, descuentos) comparten este ciclo de vida.

```mermaid
stateDiagram-v2
    [*] --> Pendiente_aprobacion: Se crea condición u operación

    Pendiente_aprobacion --> Activo: RRHH aprueba solicitud
    Pendiente_aprobacion --> Rechazado: RRHH rechaza solicitud

    Activo --> Finalizado: Cuotas pagadas / Fecha fin vigencia
    Activo --> Inactivo: RRHH desactiva manualmente

    Rechazado --> [*]
    Finalizado --> [*]
    Inactivo --> [*]
```

| Estado                   | Descripción                                                      |
|--------------------------|------------------------------------------------------------------|
| **Pendiente_aprobacion** | Creada pero requiere aprobación en Kanban                        |
| **Activo**               | Aprobada y vigente; genera subsidios o descuentos                |
| **Rechazado**            | No aprobada; no tiene efecto en cálculos                         |
| **Finalizado**           | Concluida por fin de cuotas o vigencia                           |
| **Inactivo**             | Desactivada manualmente antes del fin natural                    |

---

## 3. Vista consolidada (Solicitud y Condición/Operación)

Relación entre el estado de la solicitud y el estado de la condición u operación asociada.

```mermaid
stateDiagram-v2
    state "Solicitud" as Sol {
        [*] --> registrada
        registrada --> en_revision
        en_revision --> aprobada
        en_revision --> rechazada
    }

    state "Condición / Operación" as CO {
        [*] --> pendiente_aprobacion
        pendiente_aprobacion --> activo: si aprobada
        pendiente_aprobacion --> rechazado: si rechazada
        activo --> finalizado: cuotas pagadas / fin vigencia
    }

    aprobada --> activo: vincula
    rechazada --> rechazado: vincula
```
