```mermaid
graph TD
    subgraph Uno ["1. Reporte Express"]
        START((Inicio)) --> A1("Reportador elige: TIPO [Calidad, Manto, Seguridad]")
        A1 --> A2("Reportador elige: LINEA [L1, L2, L3]")
        A2 --> A3(Bot pide FOTO Evidencia)
        A3 --> A4(Reportador envia Foto)
        A4 --> B1(Bot busca Responsable en Turno)
    end

    subgraph Dos ["2. Notificacion en Grupo"]
        B1 --> G1("Bot envia a GRUPO: Atencion @JuanPerez en L1 + FOTO")
        
        G1 --> G2{Alguien presiona Gestionar?}
        G2 -- Usuario Incorrecto --> G3(Bot: Tu no eres el responsable)
        G3 --> G2
        G2 -- Usuario Correcto --> G4("Bot actualiza grupo: ✅ Atendiendo")
        G4 --> DM(Bot abre Chat Privado)

        G1 -.-> TIMER{Pasaron 5 min sin respuesta?}
        TIMER -- Si --> ESC1(Bot busca Jefe Inmediato)
        ESC1 --> ESC2("Bot envia a GRUPO: 🚨 ESCALADO a @Jefe + FOTO")
        ESC2 --> TIMER
        ESC2 --> G2
    end

    subgraph Tres ["3. Ejecucion y Clasificacion"]
        DM --> W1(Mecanico trabaja en falla)
        W1 --> W2{Termino?}
        W2 -- Si --> W3("Bot pide: FOTO de Solucion")
        W3 --> W4("Bot pide: CLASIFICACION FINAL (Menu Top 5)")
        W4 --> W5(Mecanico clasifica incidencia)
    end

    subgraph Cuatro ["4. Confirmacion Reportador"]
        W5 --> V1("Bot pregunta a Reportador: ¿Ya quedo?")
        V1 --> V2{Reportador Confirma?}
        
        V2 --  No quedo --> REJECT(Bot marca: Rechazado por Usuario)
        REJECT --> ESC1
        
        V2 --  Si quedo --> V3(Incidencia Resuelta - Inicia Monitoreo)
    end

    subgraph Cinco ["5. Loop de Monitoreo"]
        V3 --> MON1(Bot espera Tiempo X)
        MON1 --> MON2("Bot pregunta a Mecanico: Sigue funcionando?")
        MON2 --> MON3{Respuesta Mecanico}
        
        MON3 --  Fallo de nuevo --> REOPEN(Reabrir: Nueva Atencion)
        REOPEN --> ESC1
        
        MON3 --  Todo bien --> MON4{Contador N veces?}
        MON4 -- Faltan revisiones --> MON1
        MON4 -- Revisiones Completas --> FIN((Cierre Final y Archivo))
    end
