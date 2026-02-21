# Diagramas de Secuencia - MS. Condiciones, Anticipos y Descuentos

Documento con los flujos principales del sistema en formato Mermaid (compatible con GitHub).

---

## 1. Asignar condición a trabajador

RRHH asigna una condición (alimentación, movilidad, vivienda) a un trabajador. La condición queda pendiente de aprobación hasta que RRHH la revise en el Kanban.

```mermaid
sequenceDiagram
    participant RRHH
    participant SistemaCAD
    participant API_RRHH
    participant CondicionesStore
    participant AprobacionesStore

    RRHH->>SistemaCAD: Ingresa a Registro de Condiciones
    SistemaCAD->>API_RRHH: obtenerTodosTrabajadores()
    API_RRHH-->>SistemaCAD: Lista de trabajadores activos
    SistemaCAD-->>RRHH: Muestra listado de trabajadores

    RRHH->>SistemaCAD: Selecciona trabajador y configuración de condición
    SistemaCAD->>CondicionesStore: obtenerCondicionesPorTrabajador(trabajadorId)
    CondicionesStore-->>SistemaCAD: Condiciones actuales (valida duplicados)
    
    RRHH->>SistemaCAD: Ingresa monto, fundamento, fecha inicio/fin
    SistemaCAD->>CondicionesStore: asignarCondicion(datos)
    CondicionesStore->>CondicionesStore: Crea CondicionTrabajador (estado: pendiente_aprobacion)
    CondicionesStore-->>SistemaCAD: condicionId

    SistemaCAD->>API_RRHH: obtenerTrabajador(trabajadorId)
    API_RRHH-->>SistemaCAD: Datos del trabajador (nombre, DNI)

    SistemaCAD->>AprobacionesStore: registrarSolicitud(condicion, trabajador, fundamento)
    AprobacionesStore->>AprobacionesStore: Crea Solicitud (estado: registrada)
    AprobacionesStore-->>SistemaCAD: solicitudId

    SistemaCAD-->>RRHH: Confirmación: "Condición asignada - Pendiente de aprobación"
```

---

## 2. Registrar préstamo, adelanto o descuento

RRHH registra una operación (préstamo, adelanto o descuento) para un trabajador. La operación inicia en estado pendiente_aprobacion.

```mermaid
sequenceDiagram
    participant RRHH
    participant SistemaCAD
    participant API_RRHH
    participant OperacionesStore

    RRHH->>SistemaCAD: Ingresa a Registro de Operaciones
    SistemaCAD->>API_RRHH: obtenerTodosTrabajadores()
    API_RRHH-->>SistemaCAD: Lista de trabajadores activos
    SistemaCAD-->>RRHH: Muestra trabajadores

    RRHH->>SistemaCAD: Selecciona tipo (préstamo/adelanto/descuento) y trabajador
    SistemaCAD->>OperacionesStore: obtenerOperacionesPorTrabajador(trabajadorId)
    OperacionesStore-->>SistemaCAD: Operaciones activas (validar límites)

    alt Préstamo
        RRHH->>SistemaCAD: Ingresa monto total, cuotas, intereses, concepto
        SistemaCAD->>OperacionesStore: simularOperacion(tipo, monto, cuotas, intereses)
        OperacionesStore-->>SistemaCAD: Cronograma de pagos
        SistemaCAD-->>RRHH: Muestra simulación
        RRHH->>SistemaCAD: Confirma registro
        SistemaCAD->>OperacionesStore: crearPrestamo(datos)
    else Adelanto
        RRHH->>SistemaCAD: Ingresa monto, cuotas, concepto
        SistemaCAD->>OperacionesStore: simularOperacion(tipo, monto, cuotas)
        OperacionesStore-->>SistemaCAD: Cronograma
        RRHH->>SistemaCAD: Confirma registro
        SistemaCAD->>OperacionesStore: crearAdelanto(datos)
    else Descuento
        RRHH->>SistemaCAD: Ingresa monto, tipo (fijo/%), permanente, fechas
        SistemaCAD->>OperacionesStore: crearDescuento(datos)
    end

    OperacionesStore->>OperacionesStore: Crea operación (estado: pendiente_aprobacion)
    OperacionesStore-->>SistemaCAD: operacionId
    SistemaCAD-->>RRHH: "Operación registrada - Pendiente de aprobación en Kanban"
```

---

## 3. Calcular subsidios mensuales

RRHH genera el cálculo mensual de subsidios (alimentación, pasajes, arriendo) usando asistencias del mes desde API RRHH.

```mermaid
sequenceDiagram
    participant RRHH
    participant SistemaCAD
    participant API_RRHH
    participant CondicionesStore

    RRHH->>SistemaCAD: Ingresa a Subsidios Mensuales
    RRHH->>SistemaCAD: Selecciona mes y año

    SistemaCAD->>CondicionesStore: condicionesAsignadas (activas)
    CondicionesStore-->>SistemaCAD: Lista de trabajadores con condiciones activas

    loop Por cada trabajador con condiciones activas
        SistemaCAD->>API_RRHH: obtenerAsistencia(trabajadorId, mes, anio)
        API_RRHH-->>SistemaCAD: RegistroAsistencia (diasAsistidos, diasHabiles, diasFaltados)

        SistemaCAD->>CondicionesStore: calcularCondicionesMensual(trabajadorId, mes, anio)
        CondicionesStore->>CondicionesStore: Obtiene condiciones activas del trabajador
        CondicionesStore->>CondicionesStore: Aplica regla: monto * (diasAsistidos / diasHabiles) si calculoProporcionado
        CondicionesStore->>CondicionesStore: Genera CalculoCondicionMensual por cada condición
        CondicionesStore->>CondicionesStore: Genera ResumenCondicionesMensual
        CondicionesStore-->>SistemaCAD: Resumen (alimentación, pasajes, arriendo, total)
    end

    SistemaCAD-->>RRHH: Muestra resumen consolidado de subsidios del mes

    RRHH->>SistemaCAD: Revisa y confirma
    RRHH->>SistemaCAD: Registra pago de subsidios (por trabajador o masivo)

    SistemaCAD->>CondicionesStore: registrarPagoSubsidios(trabajadorId, mes, anio)
    CondicionesStore->>CondicionesStore: Crea HistorialRegistroSubsidio
    CondicionesStore-->>SistemaCAD: OK
    SistemaCAD-->>RRHH: "Pago de subsidios registrado"
```

---

## 4. Calcular y aplicar descuentos mensuales

RRHH genera el cálculo mensual de descuentos (préstamos, adelantos, descuentos) para aplicar a planillas.

```mermaid
sequenceDiagram
    participant RRHH
    participant SistemaCAD
    participant OperacionesStore

    RRHH->>SistemaCAD: Ingresa a Descuentos Mensuales
    RRHH->>SistemaCAD: Selecciona mes y año

    SistemaCAD->>OperacionesStore: prestamos, adelantos, descuentos (estado: activo)
    OperacionesStore-->>SistemaCAD: Operaciones activas por trabajador

    SistemaCAD->>OperacionesStore: generarCalculoMasivoMensual(mes, anio)
    
    loop Por cada trabajador con operaciones activas
        OperacionesStore->>OperacionesStore: calcularOperacionesMensual(trabajadorId, mes, anio)
        OperacionesStore->>OperacionesStore: Por préstamo/adelanto: montoDescuento = cuotaMensual
        OperacionesStore->>OperacionesStore: Por descuento: montoDescuento = monto fijo o %
        OperacionesStore->>OperacionesStore: Genera CalculoOperacionMensual
        OperacionesStore->>OperacionesStore: Genera ResumenOperacionesMensual
    end

    OperacionesStore-->>SistemaCAD: Resúmenes por trabajador
    SistemaCAD-->>RRHH: Muestra resumen consolidado de descuentos del mes

    RRHH->>SistemaCAD: Revisa y confirma
    RRHH->>SistemaCAD: Aplicar descuentos mensuales

    SistemaCAD->>OperacionesStore: aplicarDescuentosMensuales(trabajadorId, mes, anio)
    OperacionesStore->>OperacionesStore: Crea HistorialRegistroDescuento
    OperacionesStore->>OperacionesStore: Actualiza cuotasPagadas en préstamos/adelantos
    OperacionesStore-->>SistemaCAD: OK
    SistemaCAD-->>RRHH: "Descuentos aplicados y registrados"
```

---

## 5. Revisar y aprobar solicitudes en Kanban

RRHH revisa solicitudes pendientes (condiciones, préstamos, adelantos, descuentos) y las aprueba o rechaza.

```mermaid
sequenceDiagram
    participant RRHH
    participant SistemaCAD
    participant AprobacionesStore
    participant CondicionesStore
    participant OperacionesStore

    RRHH->>SistemaCAD: Ingresa a Kanban de Aprobaciones
    SistemaCAD->>AprobacionesStore: obtenerSolicitudesPorEstado() por columna
    AprobacionesStore-->>SistemaCAD: Registradas | En revisión | Aprobadas | Rechazadas
    SistemaCAD-->>RRHH: Muestra Kanban con columnas

    RRHH->>SistemaCAD: Aplica filtros (tipo, trabajador, estado, fechas)
    SistemaCAD->>AprobacionesStore: obtenerSolicitudesFiltradas()
    AprobacionesStore-->>SistemaCAD: Solicitudes filtradas
    SistemaCAD-->>RRHH: Actualiza vista del Kanban

    RRHH->>SistemaCAD: Selecciona solicitud (clic)
    SistemaCAD->>AprobacionesStore: obtenerSolicitudPorId(id)
    AprobacionesStore-->>SistemaCAD: Detalle completo de la solicitud
    SistemaCAD-->>RRHH: Muestra detalle (trabajador, monto, concepto, fundamento)

    alt RRHH aprueba
        RRHH->>SistemaCAD: Clic en Aprobar
        SistemaCAD->>AprobacionesStore: aprobarSolicitud(solicitudId)
        AprobacionesStore->>AprobacionesStore: cambiarEstadoSolicitud(aprobada)
        
        alt Si es solicitud de condición
            AprobacionesStore->>CondicionesStore: actualizarCondicion(condicionId, activo: true, estadoAprobacion: activo)
        end
        
        AprobacionesStore-->>SistemaCAD: OK
        SistemaCAD-->>RRHH: "Solicitud aprobada"
    else RRHH rechaza
        RRHH->>SistemaCAD: Clic en Rechazar
        RRHH->>SistemaCAD: Ingresa motivo de rechazo
        SistemaCAD->>AprobacionesStore: rechazarSolicitud(solicitudId, motivo)
        AprobacionesStore->>AprobacionesStore: cambiarEstadoSolicitud(rechazada, motivoRechazo)
        
        alt Si es solicitud de condición
            AprobacionesStore->>CondicionesStore: actualizarCondicion(estadoAprobacion: rechazado, motivoRechazo)
        end
        
        AprobacionesStore-->>SistemaCAD: OK
        SistemaCAD-->>RRHH: "Solicitud rechazada"
    end

    RRHH->>SistemaCAD: Mueve solicitud entre columnas (opcional)
    SistemaCAD->>AprobacionesStore: moverSolicitud(solicitudId, nuevoEstado)
```

---

## 6. Simular operación antes de registrar

RRHH simula un préstamo o adelanto para ver el cronograma de cuotas antes de registrar.

```mermaid
sequenceDiagram
    participant RRHH
    participant SistemaCAD
    participant OperacionesStore
    participant API_RRHH

    RRHH->>SistemaCAD: Ingresa a Registro de Operaciones
    RRHH->>SistemaCAD: Selecciona tipo: Préstamo o Adelanto

    SistemaCAD->>API_RRHH: obtenerTodosTrabajadores()
    API_RRHH-->>SistemaCAD: Lista de trabajadores
    SistemaCAD-->>RRHH: Muestra trabajadores para seleccionar

    RRHH->>SistemaCAD: Selecciona trabajador

    alt Préstamo
        RRHH->>SistemaCAD: Ingresa monto total, número de cuotas, intereses, fecha inicio
        SistemaCAD->>OperacionesStore: simularOperacion(tipo: prestamo, montoTotal, numeroCuotas, intereses, fechaInicio)
    else Adelanto
        RRHH->>SistemaCAD: Ingresa monto total, número de cuotas, fecha inicio
        SistemaCAD->>OperacionesStore: simularOperacion(tipo: adelanto, montoTotal, numeroCuotas, fechaInicio)
    end

    OperacionesStore->>OperacionesStore: Calcula cuotaMensual = (montoTotal + intereses) / numeroCuotas
    OperacionesStore->>OperacionesStore: Genera cronograma: [{numeroCuota, mes, montoCuota, saldoRestante}, ...]
    OperacionesStore-->>SistemaCAD: SimulacionOperacion (cronograma completo)

    SistemaCAD-->>RRHH: Muestra cronograma de pagos (tabla con cuotas, fechas, saldos)

    RRHH->>RRHH: Revisa si las cuotas son viables
    alt RRHH confirma
        RRHH->>SistemaCAD: Clic en Registrar
        Note over SistemaCAD: Continúa con flujo de Registro de operación
    else RRHH modifica
        RRHH->>SistemaCAD: Ajusta monto o cuotas
        Note over SistemaCAD: Vuelve a simular
    end
```
