```mermaid
graph TD
    subgraph Uno ["1. Reporte express"]
        START((Inicio)) --> A1("Reportador elige: Tipo [Calidad, Manto, Seguridad]")
        A1 --> A2("Reportador elige: Linea [L1, L2, L3]")
        A2 --> A3(Bot pide foto evidencia)
        A3 --> A4(Reportador envia foto)
        A4 --> B1(Bot crea ticket #123, estatus: Abierto)
        B1 --> TIMER_START((Inicia timer global 1 hr))
    end

    subgraph Dos ["2. Notificacion y escalacion"]
        B1 --> G1("Bot envia a grupo: Atencion @Responsable + foto")
        
        G1 --> G2{Alguien presiona gestionar?}
        G2 -- Usuario incorrecto --> G3(Bot: Tu no eres el responsable)
        G3 --> G2
        G2 -- Usuario correcto --> G4("Bot actualiza mensaje grupo: Atendiendo")
        G4 --> DM(Bot abre chat privado)

        G1 -.-> TIMER_5{Pasaron 5 min sin respuesta?}
        TIMER_5 -- Si --> ESC1(Bot busca jefe inmediato en BD)
        ESC1 --> ESC2("Bot envia a grupo: Escalado a @Jefe + foto")
        ESC2 --> TIMER_5
        ESC2 --> G2
    end

    subgraph Tres ["3. Ejecucion y clasificacion"]
        DM --> W1(Mecanico trabaja en falla)
        
        TIMER_START -.-> TIMER_60{Paso 1 hora sin cierre?}
        TIMER_60 -- Si --> ALARM("Bot envia a grupo: Alerta critica - Ticket vencido")
        ALARM --> W1

        W1 --> W2{Mecanico termina?}
        W2 -- Si --> W3("Bot pide: Foto de solucion")
        W3 --> W4("Bot pide: Clasificacion final - Menu top 5 + Otro")
        W4 --> W5{Eleccion mecanico}
        
        W5 -- Menu top 5 --> W6(Guardar categoria en BD)
        W5 -- Opcion otro --> W7("Mecanico escribe texto libre")
        W7 --> W8(API IA procesa texto y guarda categoria)
    end

    subgraph Cuatro ["4. Validacion reportador"]
        W6 --> V1("Bot pregunta a reportador: Ya quedo?")
        W8 --> V1
        
        V1 --> V2{Reportador confirma?}
        
        V2 -- No --> REJECT(Bot marca estatus: Rechazado)
        REJECT --> ESC1
        
        V2 -- Si --> V3(Bot marca estatus: Resuelto - Inicia niñera)
    end

    subgraph Cinco ["5. Loop de monitoreo"]
        V3 --> MON1(Bot espera tiempo X)
        MON1 --> MON2("Bot pregunta a mecanico: Sigue funcionando?")
        MON2 --> MON3{Respuesta mecanico}
        
        MON3 -- Fallo --> REOPEN_ACT("BD: Estatus reabierto / Atenciones +1")
        REOPEN_ACT --> ESC1
        
        MON3 -- Todo bien --> MON4{Contador menor a N veces?}
        MON4 -- Si --> MON1
        MON4 -- No --> FIN((Estatus: Cerrado / Archivado))
    end
