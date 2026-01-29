```mermaid
graph TD
    subgraph 1. Reporte inicial
        %% FLUJO PRINCIPAL
        A1("Usuario: Hola o Incidencia") --> P1["Plantilla 1: Bienvenida y Tipo - Menú Lista"]
        P1 --> DEC_TIPO{"Usuario elige opción"}
        
        DEC_TIPO -- "Manto, calidad, etc." --> SET_TYPE("Guardar tipo seleccionado")
        SET_TYPE --> P3["Plantilla 3: Ubicación - Menú Lista"]
        P3 --> A3("Usuario elige zona")
        
        %% AQUI ESTA EL PASO QUE FALTABA
        A3 --> P4["Plantilla 4: Solicitud Evidencia Inicial - Texto"]
        P4 --> A4_PHOTO("Usuario envía foto o texto")
        A4_PHOTO --> B_INIT("Bot inicia variables: Nivel_escalación 1, Intento_monitoreo 0")

        %% RAMA "OTRO"
        DEC_TIPO -- "Otro" --> P2["Plantilla 2: Descripción otro - Texto"]
        P2 --> A2_OTHER("Usuario escribe descripción")
        A2_OTHER --> SET_TYPE

        %% TIMEOUT WIZARD (DERECHA)
        P1 -.-> TIMEOUT_WIZARD{"Timeout creación 5min"}
        TIMEOUT_WIZARD -.->|"Usuario abandonó"| CANCEL_SESSION("Borrar sesión temporal")

        %% AJUSTE VISUAL
        DEC_TIPO ~~~ TIMEOUT_WIZARD
    end

    B_INIT --> CHECK_LEVEL

    subgraph 2. Lógica de asignación y escalamiento
        CHECK_LEVEL{{"Es Nivel 1?"}}
        
        %% PLANTILLAS RENUMERADAS (P4->P5, P5->P6)
        CHECK_LEVEL -- "Sí - Primer intento" --> P5["Plantilla 5: Notificación Asignación - CTA Link"]
        CHECK_LEVEL -- "No - Nivel superior" --> P6["Plantilla 6: Alerta Escalamiento - CTA Link"]
        
        P5 --> WAIT_RESP{"Esperar tiempo configurable"}
        P6 --> WAIT_RESP
        
        WAIT_RESP -- "Click en link" --> M_DM("Bot DM a responsable")
        %% PLANTILLA RENUMERADA (P6->P7)
        M_DM --> P7{"Plantilla 7: Aceptación Técnico - Quick Reply"}
        
        P7 -- "Acepta" --> START_WORK("Iniciar timer de solución configurable")

        %% RETORNO AL BUCLE
        WAIT_RESP -- "Tiempo agotado o rechaza" --> INC_LEVEL("Nivel = Nivel + 1")
        P7 -- "Rechaza" --> INC_LEVEL
        INC_LEVEL --> CHECK_LEVEL

        %% AJUSTE VISUAL
        M_DM ~~~ INC_LEVEL
    end

    subgraph 3. Ejecución
        START_WORK --> EXECUTION("Fase de ejecución activa")
        %% PLANTILLA RENUMERADA (P7->P8)
        EXECUTION --> P8["Plantilla 8: Menú Acciones - Menú Lista"]
        P8 --> DEC_TEC{"Decisión técnico"}
        
        DEC_TEC -- "Subir evidencia" --> P8
        %% PLANTILLA RENUMERADA (P8->P9)
        DEC_TEC -- "Falla solucionada" --> P9["Plantilla 9: Solicitud Foto Solución - Texto"]

        %% RAMAS LATERALES
        DEC_TEC -- "Pedir ayuda" --> INC_LEVEL
        START_WORK -.-> CHECK_WORK_TIMER{{"Timer solución agotado?"}}
        CHECK_WORK_TIMER -.->|"Sí - Se acabó el tiempo"| INC_LEVEL

        %% AJUSTE VISUAL
        DEC_TEC ~~~ CHECK_WORK_TIMER
    end

    subgraph 4. Validación reportador
        %% PLANTILLA RENUMERADA (P9->P10)
        P9 --> P10["Plantilla 10: Confirmación Usuario - Quick Reply"]
        P10 --> U_VAL{"Usuario confirma?"}
        
        U_VAL -- "Sí - Ya quedó" --> PRE_MON("Incidencia resuelta tentativa")
        U_VAL -- "No - Sigue fallando" --> REOPEN_LOGIC("Falla detectada por usuario")
        
        %% TIMEOUT LATERAL
        P10 -.-> TIMEOUT_VAL{"Timeout 2 horas sin respuesta"}
        TIMEOUT_VAL -.->|"AUTO-ACEPTAR: Asumir resuelto"| PRE_MON

        %% AJUSTE VISUAL
        U_VAL ~~~ TIMEOUT_VAL
    end

    subgraph 5. Monitoreo y segunda instancia
        PRE_MON --> CHECK_LOOP{{"Contador menor a N veces?"}}
        
        CHECK_LOOP -- "Sí - Sigue monitoreando" --> WAIT_MON("Esperar X horas configurable")
        %% PLANTILLA RENUMERADA (P10->P11)
        WAIT_MON --> P11["Plantilla 11: Monitoreo Calidad - Quick Reply"]
        
        P11 --> RESP_MON{"Respuesta usuario"}
        RESP_MON -- "Sí, todo bien" --> INC_LOOP("Contador = Contador + 1")
        INC_LOOP --> CHECK_LOOP
        
        %% TIMEOUT LATERAL
        P11 -.-> TIMEOUT_MON{"Timeout 1 hora sin respuesta"}
        TIMEOUT_MON -.->|"AUTO-CONTINUAR: Asumir OK"| INC_LOOP

        %% LOGICA REAPERTURA (REUSA PLANTILLA 6 AHORA)
        RESP_MON -- "No, falló de nuevo" --> REOPEN_LOGIC
        
        REOPEN_LOGIC --> CALC_BOSS("Calcular jefe inmediato Nivel + 1")
        CALC_BOSS --> SEND_WARN["Reusar Plantilla 6: Alerta Escalamiento (Motivo: Reincidencia)"]
        
        SEND_WARN --> RESET_L1("Resetear Nivel a 1 mecánico original")
        RESET_L1 --> SET_FLAG("Marcar flag: Segunda instancia")
        SET_FLAG --> CHECK_LEVEL
        
        CHECK_LOOP -- "No - Se cumplieron N veces" --> CLOSE("CIERRE DEFINITIVO")

        %% AJUSTE VISUAL
        RESP_MON ~~~ TIMEOUT_MON
    end
