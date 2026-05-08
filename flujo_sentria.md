```mermaid
flowchart TD
    %% 1. Declaración de las fases y su contenido interno
    
    subgraph Fase1 [1. Reporte inicial]
        A1("Usuario: hola o incidencia") --> P1["P1: Servicios o mantenimiento"]
        P1 --> DEC_CAT("Elige categoría")
        DEC_CAT --> P2["P2: Menú línea/zona"]
        P2 --> DEC_LINEA("Elige línea")
        DEC_LINEA --> P3["P3: Tipo de incidencia (con 'otro')"]
        P3 --> DEC_TIPO("Elige tipo")
        DEC_TIPO --> CHECK_CAT{"¿Es mantenimiento?"}
        CHECK_CAT -- "Sí" --> P4["P4: Menú máquina/aparato (con 'otro')"]
        P4 --> DEC_MAQ("Elige máquina")
        CHECK_CAT -- "No (servicios)" --> CHECK_OTRO
        DEC_MAQ --> CHECK_OTRO{"¿Tipo o máquina es 'otro'?"}
        CHECK_OTRO -- "Sí" --> P5["P5: Solicitud descripción - texto"]
        P5 --> A_DESC("Envía descripción")
        A_DESC --> P6["P6: Solicitud foto inicial - obligatoria"]
        CHECK_OTRO -- "No" --> P6
        P6 --> A_FOTO("Envía foto del problema")
        A_FOTO --> INIT_VAR("Backend: inicia búsqueda")
    end

    subgraph Fase2 [2. Lógica de asignación continua]
        SEARCH_TECH{"Buscar técnico"}
        CHECK_AVAIL{"¿Quedan disponibles?"}
        ALERT_EMPTY(("Alerta a supervisor: sin técnicos"))
        P7["P7: Asignación a técnico - cta link"]
        WAIT_RESP{"Esperar respuesta técnico"}
        
        SEARCH_TECH --> CHECK_AVAIL
        CHECK_AVAIL -- "No" --> ALERT_EMPTY
        CHECK_AVAIL -- "Sí" --> P7
        P7 --> WAIT_RESP
        WAIT_RESP -- "Rechaza / no contesta" --> SEARCH_TECH
        ALERT_EMPTY -. "Reinicia lista o espera" .-> SEARCH_TECH
    end

    subgraph Fase3 [3. Ejecución directa]
        START_WORK("Iniciar sla de solución")
        P8["P8: Menú acciones técnico - lista"]
        DEC_TEC{"Decisión técnico"}
        A_TXT("Envía nota")
        A_FOTO_EXT("Envía foto")
        ASK_HELP(("Aviso a supervisor por ayuda"))
        P9["P9: Solicitud foto final"]
        A_FOTO_FIN("Envía foto de solución")
        
        START_WORK --> P8
        P8 --> DEC_TEC
        DEC_TEC -- "Agregar comentario" --> A_TXT --> P8
        DEC_TEC -- "Agregar foto extra" --> A_FOTO_EXT --> P8
        DEC_TEC -- "Pedir ayuda" --> ASK_HELP
        ASK_HELP --> P8
        DEC_TEC -- "Declarar resuelto" --> P9
        P9 --> A_FOTO_FIN
    end

    subgraph Fase4 [4. Validación reportador]
        P10["P10: Confirmación usuario - quick reply"]
        U_VAL{"Usuario confirma?"}
        TIMEOUT_VAL{"Timeout 15 min"}
        
        P10 --> U_VAL
        P10 -.-> TIMEOUT_VAL
    end

    subgraph Fase5 [5. Monitoreo calidad simplificado]
        PRE_MON("Pausa para monitoreo")
        WAIT_2H("Esperar 2 horas exactas")
        P11["P11: Monitoreo calidad - quick reply"]
        RESP_MON{"Respuesta usuario"}
        TIMEOUT_MON{"Timeout 1h"}
        CLOSE("Cierre exitoso")
        
        PRE_MON --> WAIT_2H
        WAIT_2H --> P11
        P11 --> RESP_MON
        RESP_MON -- "Todo bien" --> CLOSE
        P11 -.-> TIMEOUT_MON
        TIMEOUT_MON -.->|"Auto-cerrar"| CLOSE
    end

    subgraph Fase6 [6. Reapertura de incidencia]
        REWORK_SEARCH("Retrabajo: buscar tech original y aviso a supervisor")
    end

    subgraph Fase7 [7. Alertas paralelas a supervisores]
        WAIT_1H{"Pasa 1 hora activa"}
        NOTIFY_BOSS("Aviso a supervisor solo notificar")
        
        WAIT_1H --> NOTIFY_BOSS
        NOTIFY_BOSS -- "Escalamiento +1" --> WAIT_1H
    end

    %% 2. Forzar el orden visual (Enlaces invisibles para apilar las cajas)
    Fase1 ~~~ Fase2
    Fase2 ~~~ Fase3
    Fase3 ~~~ Fase4
    Fase4 ~~~ Fase5
    Fase5 ~~~ Fase6
    Fase6 ~~~ Fase7

    %% 3. Conexiones reales entre las fases
    INIT_VAR --> SEARCH_TECH
    WAIT_RESP -- "Click y acepta" --> START_WORK
    A_FOTO_FIN --> P10
    
    U_VAL -- "Sí - ya quedó" --> PRE_MON
    TIMEOUT_VAL -.->|"Auto-aceptar"| PRE_MON
    
    U_VAL -. "No - sigue fallando" .-> REWORK_SEARCH
    RESP_MON -. "Falló de nuevo" .-> REWORK_SEARCH
    
    REWORK_SEARCH --> SEARCH_TECH
    
    SEARCH_TECH -. "Inicia timer alertas" .-> WAIT_1H
    START_WORK -. "Continúa timer alertas" .-> WAIT_1H
