# Proyecto Sentria: Bot de Incidencias de Fábrica

Este repositorio contiene la documentación y la lógica de flujo para el bot de IA (`Sentria`) encargado de gestionar las incidencias y fallas reportadas en la planta.

## Objetivo del Proyecto

El objetivo principal es **automatizar y agilizar** el ciclo completo de reporte, asignación y resolución de fallas, utilizando **WhatsApp** como la interfaz principal para todo el personal (reportadores, mecánicos y supervisores).

## Funcionalidad General

El bot está diseñado para manejar el siguiente flujo:

* **Reporte Guiado:** El personal de planta reporta una incidencia (Mantenimiento, Calidad, etc.) usando botones de WhatsApp para minimizar el texto libre.
* **Asignación Inteligente:** El bot asigna automáticamente la incidencia al Mecánico 1 (Mecánico de Zona).
* **Gestión de Tiempos (Timeouts):**
    * Si el Mecánico 1 no acepta la incidencia en **5 minutos**, el bot notifica al jefe de área.
    * Se re-asigna a un Mecánico 2 (de otra zona, si está disponible).
* **Escalación Automática:**
    * Si el Mecánico 2 no acepta, se escala a un **Supervisor** para asignación manual.
    * Si la incidencia completa no se cierra en **1 hora**, se escala automáticamente al Supervisor.
* **Cierre y Validación:** El mecánico marca la falla como resuelta (adjuntando fotos/comentarios). El **reportador original** debe confirmar por WhatsApp que el problema se solucionó para poder cerrar la incidencia.

---

## Diagrama de Flujo Completo

Toda la lógica de decisión, los timers, los mensajes y las rutas de escalación están documentados visualmente en el siguiente archivo.

Este diagrama es la "fuente de verdad" para el desarrollo del bot.

> **[Haz clic aquí para ver el Diagrama de Flujo (FLUJO_BOT.md)](FLUJO_BOT.md)**
