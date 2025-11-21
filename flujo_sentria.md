```mermaid
graph TD
    %% FASE 1: REPORTE SIMPLIFICADO (EL ACUSON)
    subgraph 1. Reporte Express
        A1(Reportador inicia bot) --> A2("Bot: Selecciona TIPO [Calidad], [Manto], [Seguridad]")
        A2 --> A3("Reportador elige: [Linea 1], [Linea 2], etc.")
        A3 --> A4(Bot pide FOTO Evidencia)
        A4 --> A5(Reportador envia Foto)
        A5 --> B1(Bot crea Incidencia #123 y busca responsable en turno)
    end

    %% FASE 2: NOTIFICACION GRUPAL Y ASIGNACION
    subgraph 2. Notificacion en Grupo
        B1 --> G1("Bot envia a GRUPO DE WHATSAPP: 'Atencion @JuanPerez en L1' + FOTO")
        G1 --> G2{Alguien presiona 'Gestionar'?}
        G2 -- Usuario Incorrecto --> G3(Bot: 'Tu no eres el responsable')
        G3 --> G2
        G2 -- Usuario Correcto --> G4("Bot actualiza grupo: 'Atendiendo'")
        G4 --> DM(Bot abre Chat Privado con Mecanico)
    end

    %% FASE 3: MOTOR DE ESCALACION (HIERARCHY LOOP)
    G1 -.-> TIMER{Timer Vencido?}
    TIMER -- Si --> ESC1(Bot busca Siguiente Nivel Jerarquico)
    ESC1 --> ESC2("Bot envia a GRUPO: 'Atencion @JefeInmediato (Escalado)'")
    ESC2 --> TIMER
    ESC2 --> G2

    %% FASE 4: RESOLUCION Y CATEGORIZACION (EL EXPERTO)
    subgraph 3. Ejecucion y Clasificacion
        DM --> W1(En Proceso de Reparacion)
        W1 --> W2{Mecanico termina?}
        W2 -- Si --> W3("Bot pide: FOTO de Solucion")
        W3 --> W4("Bot pide: CLASIFICACION FINAL (Menu Top 5 + Otro)")
        W4 --> W5(Mecanico selecciona Categoria Tecnica)
    end

    %% FASE 5: EL BUCLE DE LA NIÑERA (MONITOREO)
    subgraph 4. Monitoreo y Cierre
        W5 --> MON1(Bot espera Tiempo X)
        MON1 --> MON2("Bot pregunta a Mecanico: '¿Sigue funcionando bien?'")
        MON2 --> MON3{Respuesta Mecanico}
        
        MON3 --  Fallo de nuevo --> REOPEN(Reabrir Incidencia: Nueva Atencion)
        REOPEN --> ESC1 %% Se regresa a escalacion/grupo
        
        MON3 --  Todo bien --> MON4{¿Contador < N veces?}
        MON4 -- Faltan revisiones --> MON1 %% Vuelve a esperar
        MON4 -- Revisiones Completas --> FIN(Cierre Definitivo y Archivado)
    end
