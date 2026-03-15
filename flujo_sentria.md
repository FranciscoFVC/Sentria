```mermaid
graph TD
    subgraph 1. Reporte inicial
        %% FLUJO PRINCIPAL
        A1("Usuario: Hola o Incidencia") --> P1["P1: Bienvenida y Tipo - Menú Lista"]
        P1 --> DEC_TIPO{"Usuario elige opción"}
        
        DEC_TIPO -- "Manto, Calidad, Seg..." --> SET_TYPE("Guardar tipo")
        DEC_TIPO -- "Otro" --> SET_TYPE_OTRO("Guardar tipo 'Otro'")
        
        SET_TYPE --> P2["P2: Ubicación - Menú Lista"]
        SET_TYPE_OTRO --> P2
        
        P2 --> A2("Usuario elige zona")
        
        %% --- SUB-FLUJO EFICIENTE PARA "OTRO" ---
        A2 --> CHECK_TIPO{{"¿Tipo es 'Otro'?"}}
        
        CHECK_TIPO -- "No" --> P3["P3: Solicitud Evidencia Inicial - Texto"]
        CHECK_TIPO -- "Sí" --> P3_COMBO["P3_COMBO: Solicitud Descripción + Foto"]
        
        P3 --> A3_PHOTO("Usuario envía foto o texto")
        P3_COMBO --> A3_PHOTO_DESC("Usuario envía foto con descripción")
        
        A3_PHOTO --> B_INIT("Bot inicia variables")
        A3_PHOTO_DESC --> B_INIT

        %% TIMEOUT
        P1 -.-> TIMEOUT_WIZARD{"Timeout creación 5min"}
        TIMEOUT_WIZARD -.->|"Usuario abandonó"| CANCEL_SESSION("Borrar sesión")

        DEC_TIPO ~~~ TIMEOUT_WIZARD
    end

    B_INIT --> CHECK_LEVEL

    subgraph 2. Lógica de asignación
        CHECK_LEVEL{{"Es Nivel 1?"}}
        
        CHECK_LEVEL -- "Sí" --> P4["P4: Notificación Asignación - CTA Link"]
        CHECK_LEVEL -- "No" --> P5["P5: Alerta Escalamiento - CTA Link"]
        
        P4 --> WAIT_RESP{"Esperar tiempo configurable"}
        P5 --> WAIT_RESP
        
        WAIT_RESP -- "Click en link" --> M_DM("Bot DM a responsable")
        M_DM --> P6{"P6: Aceptación Técnico - Quick Reply"}
        
        P6 -- "Acepta" --> START_WORK("Iniciar timer")

        %% RETORNO
        WAIT_RESP -- "Tiempo agotado" --> INC_LEVEL("Nivel = Nivel + 1")
        P6 -- "Rechaza" --> INC_LEVEL
        INC_LEVEL --> CHECK_LEVEL

        M_DM ~~~ INC_LEVEL
    end

    subgraph 3. Ejecución y Clasificación IA
        START_WORK --> EXECUTION("Fase de ejecución")
        EXECUTION --> P7["P7: Menú Acciones - Menú Lista"]
        P7 --> DEC_TEC{"Decisión técnico"}
        
        DEC_TEC -- "Subir evidencia" --> P7
        DEC_TEC -- "Pedir ayuda" --> INC_LEVEL
        
        %% --- SUB-FLUJO "COMBO" TÉCNICO ---
        DEC_TEC -- "Falla solucionada" --> P8_COMBO["P8_COMBO: Evidencia Final (Foto + Audio/Texto)"]
        P8_COMBO --> A_COMBO("Técnico envía Foto con nota de voz/texto")
        
        A_COMBO --> AI_PROCESS("Backend: IA Clasifica")
        AI_PROCESS --> P9["P9: Confirmación IA - Quick Reply"]
        
        P9 -- "Sí, correcto" --> P11
        
        %% FALLBACK MANUAL
        P9 -- "No, cambiar" --> P10["P10: Selección Manual Familia - Menú Lista"]
        P10 --> MANUAL_SEL("Guarda Familia Seleccionada")
        MANUAL_SEL --> P11

        %% TIMER
        START_WORK -.-> CHECK_WORK_TIMER{{"Timer agotado?"}}
        CHECK_WORK_TIMER -.->|"Sí"| INC_LEVEL

        P7 ~~~ CHECK_WORK_TIMER
    end

    subgraph 4. Validación reportador
        P11["P11: Confirmación Usuario - Quick Reply"]
        P11 --> U_VAL{"Usuario confirma?"}
        
        U_VAL -- "Sí - Ya quedó" --> PRE_MON("Incidencia resuelta")
        U_VAL -- "No - Sigue fallando" --> REOPEN_LOGIC("Falla detectada")
        
        P11 -.-> TIMEOUT_VAL{"Timeout 2h"}
        TIMEOUT_VAL -.->|"AUTO-ACEPTAR"| PRE_MON

        U_VAL ~~~ TIMEOUT_VAL
    end

    subgraph 5. Monitoreo Calidad
        PRE_MON --> CHECK_LOOP{{"Contador < N?"}}
        
        CHECK_LOOP -- "Sí" --> WAIT_MON("Esperar X horas")
        WAIT_MON --> P12["P12: Monitoreo Calidad - Quick Reply"]
        
        P12 --> RESP_MON{"Respuesta usuario"}
        RESP_MON -- "Todo bien" --> INC_LOOP("Contador + 1")
        INC_LOOP --> CHECK_LOOP
        
        P12 -.-> TIMEOUT_MON{"Timeout 1h"}
        TIMEOUT_MON -.->|"AUTO-CONTINUAR"| INC_LOOP

        RESP_MON -- "Falló de nuevo" --> REOPEN_LOGIC
        
        REOPEN_LOGIC --> CALC_BOSS("Jefe Inmediato")
        CALC_BOSS --> SEND_WARN["Reusar P5: Alerta Escalamiento"]
        SEND_WARN --> RESET_L1("Reset Nivel 1")
        RESET_L1 --> CHECK_LEVEL
        
        CHECK_LOOP -- "No" --> CLOSE("CIERRE DEFINITIVO")

        RESP_MON ~~~ TIMEOUT_MON
    end
