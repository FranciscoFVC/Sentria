```mermaid
flowchart LR
    %% Nodos globales de ruteo (Actúan como carril superior para no ensuciar las cajas)
    INC_LEVEL(("Nivel = Nivel + 1"))
    SEND_ALERT{"Enviar Alerta (SLA Activo)"}

    subgraph Fase1 [1. Reporte inicial]
        direction TB
        A1("Usuario: Hola o Incidencia") --> P1["P1: Menú Línea/Zona (Sin 'Otro')"]
        P1 --> DEC_LINEA("Usuario elige Línea")
        
        DEC_LINEA --> P2["P2: Menú Tipo de Incidencia (Con 'Otro')"]
        P2 --> DEC_TIPO("Usuario elige Tipo")
        
        DEC_TIPO --> P3["P3: Menú Máquina/Aparato (Con 'Otro')"]
        P3 --> DEC_MAQ("Usuario elige Máquina")
        
        DEC_MAQ --> CHECK_OTRO{"¿Tipo o Máquina es 'Otro'?"}
        
        CHECK_OTRO -- "Sí" --> P4["P4: Solicitud de Descripción - Texto"]
        P4 --> A_DESC("Usuario envía descripción")
        A_DESC --> P5["P5: Solicitud Foto Inicial - Obligatoria"]
        
        CHECK_OTRO -- "No" --> P5
        P5 --> A_FOTO("Usuario envía foto del problema")
        
        A_FOTO --> INIT_VAR("Backend: Nivel = 1")
    end

    INIT_VAR --> SEND_ALERT

    subgraph Fase2 [2. Lógica de Asignación en Bucle]
        direction TB
        SEND_ALERT -- "Si Nivel = 1" --> P6["P6: Asignación Inicial - CTA Link"]
        SEND_ALERT -- "Si Nivel > 1" --> P7["P7: Alerta de Escalamiento - CTA Link"]
        
        P6 --> WAIT_RESP{"Esperar SLA de Respuesta"}
        P7 --> WAIT_RESP
        
        WAIT_RESP -- "Click en link" --> M_DM("Bot DM al responsable")
        M_DM --> P8{"P8: Aceptación Técnico - Quick Reply"}
        
        P8 -- "Rechaza" --> REASSIGN("Reasignar a otro técnico (Mismo Nivel)")
        REASSIGN --> SEND_ALERT
        
        P8 -- "Acepta" --> START_WORK("Iniciar SLA de Solución")
    end

    %% Conexión del tiempo agotado hacia el hub externo
    WAIT_RESP -. "Tiempo agotado" .-> INC_LEVEL

    subgraph Fase3 [3. Ejecución Directa]
        direction TB
        START_WORK --> P9["P9: Menú Acciones Técnico - Lista"]
        P9 --> DEC_TEC{"Decisión técnico"}
        
        DEC_TEC -- "Agregar Comentario" --> A_TXT("Técnico envía nota") --> P9
        DEC_TEC -- "Agregar Foto Extra" --> A_FOTO_EXT("Técnico envía foto") --> P9
        
        DEC_TEC -- "Declarar Resuelto" --> P10["P10: Solicitud Foto Final"]
        P10 --> A_FOTO_FIN("Técnico envía foto de solución")
        
        START_WORK -.-> CHECK_SLA2{"SLA Solución Agotado?"}
    end

    %% Conexiones de escape hacia el hub externo
    DEC_TEC -. "Pedir Ayuda" .-> INC_LEVEL
    CHECK_SLA2 -. "Sí" .-> INC_LEVEL

    subgraph Fase4 [4. Validación Reportador]
        direction TB
        A_FOTO_FIN --> P11["P11: Confirmación Usuario - Quick Reply"]
        P11 --> U_VAL{"Usuario confirma?"}
        
        U_VAL -- "Sí - Ya quedó" --> PRE_MON("Pausa para monitoreo")
        
        P11 -.-> TIMEOUT_VAL{"Timeout 2h"}
        TIMEOUT_VAL -.->|"Auto-Aceptar"| PRE_MON
    end

    %% Conexión de escape
    U_VAL -. "No - Sigue fallando" .-> INC_LEVEL

    subgraph Fase5 [5. Monitoreo Calidad Simplificado]
        direction TB
        PRE_MON --> WAIT_2H("Esperar 2 horas exactas")
        WAIT_2H --> P12["P12: Monitoreo Calidad - Quick Reply"]
        
        P12 --> RESP_MON{"Respuesta usuario"}
        RESP_MON -- "Todo bien" --> CLOSE("Cierre Exitoso")
        
        P12 -.-> TIMEOUT_MON{"Timeout 1h"}
        TIMEOUT_MON -.->|"Auto-Cerrar"| CLOSE
    end

    %% Conexión de escape
    RESP_MON -. "Falló de nuevo" .-> INC_LEVEL

    %% Retorno del hub al flujo
    INC_LEVEL --> SEND_ALERT
