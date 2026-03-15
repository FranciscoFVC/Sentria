```mermaid
graph TD
    subgraph 1. Reporte inicial
        %% FLUJO PRINCIPAL
        A1("Usuario: Hola o Incidencia") --> P1["P1: Bienvenida y Tipo - Menú Lista"]
        P1 --> DEC_TIPO{"Usuario elige opción"}
        
        DEC_TIPO -- "Manto, Calidad, Seg..." --> SET_TYPE("Guardar tipo")
        DEC_TIPO -- "Otro" --> SET_TYPE_OTRO("Guardar tipo 'Otro'")
        
        SET_TYPE --> P3["P3: Ubicación - Menú Lista"]
        SET_TYPE_OTRO --> P3
        
        P3 --> A3("Usuario elige zona")
        
        %% --- NUEVO SUB-FLUJO EFICIENTE PARA "OTRO" ---
        A3 --> CHECK_TIPO{{"¿Tipo es 'Otro'?"}}
        
        CHECK_TIPO -- "No" --> P4["P4: Solicitud Evidencia Inicial - Texto"]
        CHECK_TIPO -- "Sí" --> P4_COMBO["P2/P4_COMBO: Solicitud Descripción + Foto"]
        
        P4 --> A4_PHOTO("Usuario envía foto o texto")
        P4_COMBO --> A4_PHOTO_DESC("Usuario envía foto con descripción")
        
        A4_PHOTO --> B_INIT("Bot inicia variables")
        A4_PHOTO_DESC --> B_INIT

        %% TIMEOUT
        P1 -.-> TIMEOUT_WIZARD{"Timeout creación 5min"}
        TIMEOUT_WIZARD -.->|"Usuario abandonó"| CANCEL_SESSION("Borrar sesión")

        DEC_TIPO ~~~ TIMEOUT_WIZARD
    end

    B_INIT --> CHECK_LEVEL

    subgraph 2. Lógica de asignación
        CHECK_LEVEL{{"Es Nivel 1?"}}
        
        CHECK_LEVEL -- "Sí" --> P5["P5: Notificación Asignación - CTA Link"]
        CHECK_LEVEL -- "No" --> P6["P6: Alerta Escalamiento - CTA Link"]
        
        P5 --> WAIT_RESP{"Esperar tiempo configurable"}
        P6 --> WAIT_RESP
        
        WAIT_RESP -- "Click en link" --> M_DM("Bot DM a responsable")
        M_DM --> P7{"P7: Aceptación Técnico - Quick Reply"}
        
        P7 -- "Acepta" --> START_WORK("Iniciar timer")

        %% RETORNO
        WAIT_RESP -- "Tiempo agotado" --> INC_LEVEL("Nivel = Nivel + 1")
        P7 -- "Rechaza" --> INC_LEVEL
        INC_LEVEL --> CHECK_LEVEL

        M_DM ~~~ INC_LEVEL
    end

    subgraph 3. Ejecución y Clasificación IA
        START_WORK --> EXECUTION("Fase de ejecución")
        EXECUTION --> P8["P8: Menú Acciones - Menú Lista"]
        P8 --> DEC_TEC{"Decisión técnico"}
        
        DEC_TEC -- "Subir evidencia" --> P8
        DEC_TEC -- "Pedir ayuda" --> INC_LEVEL
        
        %% --- SUB-FLUJO "COMBO" TÉCNICO ---
        DEC_TEC -- "Falla solucionada" --> P9_COMBO["P9_COMBO: Evidencia Final (Foto + Audio/Texto)"]
        P9_COMBO --> A_COMBO("Técnico envía Foto con nota de voz/texto")
        
        A_COMBO --> AI_PROCESS("Backend: IA Clasifica")
        AI_PROCESS --> P10["P10: Confirmación IA - Quick Reply"]
        
        P10 -- "Sí, correcto" --> P13
        
        %% FALLBACK MANUAL
        P10 -- "No, cambiar" --> P11["P11: Selección Manual Familia - Menú Lista"]
        P11 --> MANUAL_SEL("Guarda Familia Seleccionada")
        MANUAL_SEL --> P13

        %% TIMER
        START_WORK -.-> CHECK_WORK_TIMER{{"Timer agotado?"}}
        CHECK_WORK_TIMER -.->|"Sí"| INC_LEVEL

        P8 ~~~ CHECK_WORK_TIMER
    end

    subgraph 4. Validación reportador
        P13["P13: Confirmación Usuario - Quick Reply"]
        P13 --> U_VAL{"Usuario confirma?"}
        
        U_VAL -- "Sí - Ya quedó" --> PRE_MON("Incidencia resuelta")
        U_VAL -- "No - Sigue fallando" --> REOPEN_LOGIC("Falla detectada")
        
        P13 -.-> TIMEOUT_VAL{"Timeout 2h"}
        TIMEOUT_VAL -.->|"AUTO-ACEPTAR"| PRE_MON

        U_VAL ~~~ TIMEOUT_VAL
    end

    subgraph 5. Monitoreo Calidad
        PRE_MON --> CHECK_LOOP{{"Contador < N?"}}
        
        CHECK_LOOP -- "Sí" --> WAIT_MON("Esperar X horas")
        WAIT_MON --> P14["P14: Monitoreo Calidad - Quick Reply"]
        
        P14 --> RESP_MON{"Respuesta usuario"}
        RESP_MON -- "Todo bien" --> INC_LOOP("Contador + 1")
        INC_LOOP --> CHECK_LOOP
        
        P14 -.-> TIMEOUT_MON{"Timeout 1h"}
        TIMEOUT_MON -.->|"AUTO-CONTINUAR"| INC_LOOP

        RESP_MON -- "Falló de nuevo" --> REOPEN_LOGIC
        
        REOPEN_LOGIC --> CALC_BOSS("Jefe Inmediato")
        CALC_BOSS --> SEND_WARN["Reusar P6: Alerta Escalamiento"]
        SEND_WARN --> RESET_L1("Reset Nivel 1")
        RESET_L1 --> CHECK_LEVEL
        
        CHECK_LOOP -- "No" --> CLOSE("CIERRE DEFINITIVO")

        RESP_MON ~~~ TIMEOUT_MON
    end
