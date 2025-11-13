```mermaid
graph TD
    subgraph 1. Inicio y Captura
        A1(Reportador envia 'Incidencia') --> B1(Bot: 'Tipo de incidencia?')
        B1 --> A2("Reportador elige: [Manto], [Calidad], etc.")
        A2 --> B2(Bot: 'Sub-tipo de incidencia?')
        B2 --> A3("Reportador elige: [Mecanico], [Electrico], [Otro]")
        A3 --> B3{Eleccion es 'Otro'?}
        B3 -- Si --> B4(Bot: 'Describe la falla:')
        B4 --> A4(Reportador escribe free text)
        B3 -- No --> B5(Bot crea Incidencia #123)
        A4 --> B5
    end

    B5 --> B6(Bot Inicia Timer Global: 1 Hora)
    B6 --> B7("Bot asigna a Mecanico 1 [Zona]")
    B7 --> M1(Bot contacta a Mecanico 1)

    subgraph 2. Asignacion Mecanico 1
        M1 --> M1_DEC{"Mecanico 1 [Timer 5 min]"}
        M1_DEC -- Acepta  --> PROC(En Proceso)
        M1_DEC -- Rechaza  --> M1_FAIL
        M1_DEC -- No contesta (Timeout 5 min) --> M1_FAIL(Falla Asignacion 1)
    end

    M1_FAIL --> B8(Bot notifica a Jefe: 'Mec 1 no acepto')
    B8 --> B9("Bot busca Mecanico 2 [Otra zona]")
    B9 --> M2(Bot contacta a Mecanico 2)

    subgraph 3. Asignacion Mecanico 2
        M2 --> M2_DEC{"Mecanico 2 [Timer 5 min]"}
        M2_DEC -- Acepta  --> PROC
        M2_DEC -- Rechaza  --> M2_FAIL
        M2_DEC -- No contesta (Timeout 5 min) --> M2_FAIL(Falla Asignacion 2)
    end

    subgraph 4. En Proceso
        PROC(Incidencia Aceptada) --> PROC_OPT("Bot: Opciones [Foto Antes], [Comentar Antes], [Ver Detalles], [Escalar Ahora], [Falla Arreglada]")
        PROC_OPT --> PROC_DEC{Mecanico gestiona}
        PROC_DEC -- Trabaja y Resuelve --> PROC_FIX("Mecanico presiona [Falla Arreglada]")
        PROC_DEC -- Necesita Ayuda --> PROC_ESC("Mecanico presiona [Escalar Ahora]")
    end

    B6 -. Expira 1 Hora .-> ESC(Escalacion Automatica por Tiempo)

    M2_FAIL --> ESC
    PROC_ESC --> ESC

    subgraph 5. Escalacion a Supervisor
        ESC(Incidencia escalada a Supervisor) --> SUP_OPT("Bot: Opciones Supervisor [Reasignar Manual], [Ver Detalles]")
        SUP_OPT --> SUP_FIN(Supervisor gestiona manualmente)
    end

    subgraph 6. Cierre y Confirmacion
        PROC_FIX --> C1("Bot pide al Mecanico: [Foto Despues], [Comentar Despues], [Omitir]")
        C1 --> C2(Mecanico envia detalles o omite)
        C2 --> C3(Bot contacta a Reportador Original)
        C3 --> C4(Bot: 'Incidencia #123 marcada como resuelta. ¿Confirmas?')
        C4 --> C5{Reportador confirma?}
        C5 --  Ya Quedo --> FIN(Incidencia Cerrada)
        C5 --  No Quedo --> C6(Bot: 'Desacuerdo reportado')
        C6 --> ESC
    end
