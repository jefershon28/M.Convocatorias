# Diagrama de Casos de Uso - Mermaid (Alternativa)

Puedes visualizar este diagrama en cualquier editor que soporte Mermaid (GitHub, GitLab, Notion, VS Code con extensión Mermaid, etc.)

## Opción 1: Diagrama compacto

```mermaid
flowchart TB
    subgraph Actores
        A1[Gerencia]
        A2[Recursos Humanos]
        A3[Jefe de Área]
        A4[Postulante]
    end

    subgraph Fase1["1. Requerimientos"]
        UC1[Crear requerimiento de personal]
        UC2[Recepcionar y evaluar]
        UC3[Archivar rechazados]
    end

    subgraph Fase2["2. Convocatoria"]
        UC4[Crear convocatoria]
        UC5[Publicar en foros de empleo]
    end

    subgraph Fase3["3. Postulación"]
        UC6[Enviar CV / Postularse]
        UC7[Recepcionar y evaluar CVs]
        UC8[Archivar CVs inaptos]
    end

    subgraph Fase4["4. Selección"]
        UC9[Entrevista telefónica]
        UC10[Entrevista jefe de área]
        UC11[Prueba aptitud académica]
        UC12[Entrevista gerencia]
    end

    subgraph Fase5["5. Contratación"]
        UC13[Seleccionar ganador]
        UC14[Registrar contratación]
        UC15[Archivar no seleccionados]
    end

    A1 --> UC1
    A1 --> UC12
    A1 --> UC13
    A2 --> UC1
    A2 --> UC2
    A2 --> UC3
    A2 --> UC4
    A2 --> UC5
    A2 --> UC7
    A2 --> UC8
    A2 --> UC9
    A2 --> UC11
    A2 --> UC14
    A2 --> UC15
    A3 --> UC1
    A3 --> UC10
    A4 --> UC6

    UC1 --> UC2
    UC2 --> UC3
    UC2 --> UC4
    UC4 --> UC5
    UC5 --> UC6
    UC6 --> UC7
    UC7 --> UC8
    UC7 --> UC9
    UC9 --> UC10
    UC10 --> UC11
    UC11 --> UC12
    UC12 --> UC13
    UC13 --> UC14
    UC13 --> UC15
```

## Opción 2: Flujo de proceso (secuencial)

```mermaid
flowchart TD
    Start([Inicio]) --> UC1[1. Crear requerimiento<br/>Gerencia/RRHH/Jefe Área]
    UC1 --> UC2[2. RRHH recepciona y evalúa]
    UC2 --> |Rechazado| Arch1[Archivar]
    UC2 --> |Aprobado| UC3[3. Crear convocatoria]
    UC3 --> UC4[4. Publicar en foros]
    UC4 --> UC5[5. Recepcionar CVs]
    UC5 --> UC6[6. Evaluar CVs]
    UC6 --> |Inaptos| Arch2[Archivar]
    UC6 --> |Aptos| UC7[7. Entrevista telefónica]
    UC7 --> |Inaptos| Arch3[Archivar]
    UC7 --> |Aptos| UC8[8. Entrevista Jefe Área]
    UC8 --> |Inaptos| Arch4[Archivar]
    UC8 --> |Aptos| UC9[9. Prueba aptitud académica]
    UC9 --> |Inaptos| Arch5[Archivar]
    UC9 --> |Aptos| UC10[10. Entrevista Gerencia]
    UC10 --> UC11[11. Gerencia selecciona ganador]
    UC11 --> UC12[12. Registrar contratación<br/>personal activo]
    UC11 --> UC13[Archivar no seleccionados]
    UC12 --> End([Fin])
    UC13 --> End
    Arch1 --> End
    Arch2 --> End
    Arch3 --> End
    Arch4 --> End
    Arch5 --> End
```

## Matriz de actores y casos de uso

| Caso de Uso | Gerencia | RRHH | Jefe Área | Postulante |
|-------------|:--------:|:----:|:---------:|:----------:|
| Crear requerimiento | ✓ | ✓ | ✓ | |
| Recepcionar y evaluar requerimiento | | ✓ | | |
| Archivar rechazados | | ✓ | | |
| Crear convocatoria | | ✓ | | |
| Publicar convocatoria | | ✓ | | |
| Enviar CV / Postularse | | | | ✓ |
| Recepcionar y evaluar CVs | | ✓ | | |
| Archivar CVs inaptos | | ✓ | | |
| Entrevista telefónica | | ✓ | | |
| Entrevista jefe de área | | | ✓ | |
| Prueba aptitud académica | | ✓ | | |
| Entrevista gerencia | ✓ | | | |
| Seleccionar candidato ganador | ✓ | | | |
| Registrar contratación | | ✓ | | |
| Archivar no seleccionados | | ✓ | | |
