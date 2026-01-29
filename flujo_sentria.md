```mermaid
graph TD
    subgraph 1. Reporte inicial
        %% FLUJO PRINCIPAL
        A1("Usuario: Hola o Incidencia") --> T1["Plantilla 1: Tipo - Quick Reply"]
        T1 --> DEC_TIPO{"Usuario elige opción"}
        
        DEC_TIPO -- "Manto, calidad, etc." --> SET_TYPE("Guardar tipo seleccionado")
        SET_TYPE --> T3["Plantilla 3: Ubicación - Quick Reply"]
        T3 --> A3("Usuario elige zona")
        A3 --> B_INIT("Bot inicia variables: Nivel_escalación 1, Intento_monitoreo 0")

        %% RAMA "OTRO"
        DEC_TIPO -- "Otro" --> T2["Plantilla 2: Describe el tipo - Texto"]
        T2 --> A2_OTHER("Usuario escribe descripción")
        A2_OTHER --> SET_TYPE

        %% TIMEOUT WIZARD (DERECHA)
        T1 -.-> TIMEOUT_WIZARD{"Timeout creación 5min"}
        TIMEOUT_WIZARD -.->|"Usuario abandonó"| CANCEL_SESSION("Borrar sesión temporal")

        %% AJUSTE VISUAL
        DEC_TIPO ~~~ TIMEOUT_WIZARD
    end

    B_INIT --> CHECK_LEVEL

    subgraph 2. Lógica de asignación y escalamiento
        CHECK_LEVEL{{"Es Nivel 1?"}}
        
        CHECK_LEVEL -- "Sí - Primer intento" --> SEND_P5["Plantilla 5: Asignación inicial - Mecánico"]
        CHECK_LEVEL -- "No - Nivel superior" --> SEND_P7["Plantilla 7: Aviso de escalamiento - Jefe"]
        
        SEND_P5 --> WAIT_RESP{"Esperar tiempo configurable"}
        SEND_P7 --> WAIT_RESP
        
        WAIT_RESP -- "Click en link" --> M_DM("Bot DM a responsable")
        M_DM --> T_ACCEPT{"Plantilla NUEVA: Aceptar o rechazar"}
        
        T_ACCEPT -- "Acepta" --> START_WORK("Iniciar timer de solución configurable")

        %% RETORNO AL BUCLE
        WAIT_RESP -- "Tiempo agotado o rechaza" --> INC_LEVEL("Nivel = Nivel + 1")
        T_ACCEPT -- "Rechaza" --> INC_LEVEL
        INC_LEVEL --> CHECK_LEVEL

        %% AJUSTE VISUAL
        M_DM ~~~ INC_LEVEL
    end

    subgraph 3. Ejecución
        START_WORK --> EXECUTION("Fase de ejecución activa")
        EXECUTION --> T8["Plantilla 8: Menú técnico - Lista"]
        T8 --> DEC_TEC{"Decisión técnico"}
        
        DEC_TEC -- "Subir evidencia" --> T8
        DEC_TEC -- "Falla solucionada" --> T9["Plantilla 9: Foto solución"]

        %% RAMAS LATERALES
        DEC_TEC -- "Pedir ayuda" --> INC_LEVEL
        
        %% AQUI ESTA EL CAMBIO VISUAL DEL TIMER
        START_WORK -.-> CHECK_WORK_TIMER{{"Timer solución agotado?"}}
        CHECK_WORK_TIMER -.->|"Sí - Se acabó el tiempo"| INC_LEVEL

        %% AJUSTE VISUAL: Alinear los dos diamantes (Decisión y Timer)
        DEC_TEC ~~~ CHECK_WORK_TIMER
    end

    subgraph 4. Validación reportador
        T9 --> T12["Plantilla 12: Confirmación usuario"]
        T12 --> U_VAL{"Usuario confirma?"}
        
        U_VAL -- "Sí - Ya quedó" --> PRE_MON("Incidencia resuelta tentativa")
        U_VAL -- "No - Sigue fallando" --> REOPEN_LOGIC("Falla detectada por usuario")
        
        %% TIMEOUT LATERAL
        T12 -.-> TIMEOUT_VAL{"Timeout 2 horas sin respuesta"}
        TIMEOUT_VAL -.->|"AUTO-ACEPTAR: Asumir resuelto"| PRE_MON

        %% AJUSTE VISUAL
        U_VAL ~~~ TIMEOUT_VAL
    end

    subgraph 5. Monitoreo y segunda instancia
        PRE_MON --> CHECK_LOOP{{"Contador menor a N veces?"}}
        
        CHECK_LOOP -- "Sí - Sigue monitoreando" --> WAIT_MON("Esperar X horas configurable")
        WAIT_MON --> T13["Plantilla 13: Sigue funcionando?"]
        
        T13 --> RESP_MON{"Respuesta usuario"}
        RESP_MON -- "Sí, todo bien" --> INC_LOOP("Contador = Contador + 1")
        INC_LOOP --> CHECK_LOOP
        
        %% TIMEOUT LATERAL
        T13 -.-> TIMEOUT_MON{"Timeout 1 hora sin respuesta"}
        TIMEOUT_MON -.->|"AUTO-CONTINUAR: Asumir OK"| INC_LOOP

        %% LOGICA REAPERTURA
        RESP_MON -- "No, falló de nuevo" --> REOPEN_LOGIC
        
        REOPEN_LOGIC --> CALC_BOSS("Calcular jefe inmediato Nivel + 1")
        CALC_BOSS --> SEND_WARN["Enviar Plantilla 7 a jefe inmediato - Motivo: Reincidencia"]
        
        SEND_WARN --> RESET_L1("Resetear Nivel a 1 mecánico original")
        RESET_L1 --> SET_FLAG("Marcar flag: Segunda instancia")
        SET_FLAG --> CHECK_LEVEL
        
        CHECK_LOOP -- "No - Se cumplieron N veces" --> CLOSE("CIERRE DEFINITIVO")

        %% AJUSTE VISUAL
        RESP_MON ~~~ TIMEOUT_MON
    end
