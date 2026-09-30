# Dron de reparto urbano (ROS 2 + PX4): documentación

Sep 25, 2026 · @Xabier

## Propósito y alcance

Este paquete documental define, justifica y verifica un dron multirrotor semiautónomo para reparto de última milla en entorno urbano, con PX4 como autopiloto y ROS 2 en el ordenador de a bordo (companion computer).

El enfoque sigue la lógica de ingeniería de sistemas aeronáutica (ARP4754A a escala reducida): concepto de operación → requisitos → arquitectura → seguridad → diseño → verificación. La profundidad de cada documento se ajustará al objetivo real del proyecto (prototipo en simulación, demostrador físico o producto regulado).

## Lista de documentación

Son 14 documentos en 4 fases; los 5 primeros son los que hay que cerrar antes de escribir código serio.

| ID | Documento | Qué contiene | Referencia | Fase |
| --- | --- | --- | --- | --- |
| [DOC-01](DOC-01-conops.md) | ConOps (Concepto de operación) y análisis funcional F1–F11 | Misión, actores, escenario urbano, modos de operación, nivel de autonomía, flujo de una entrega | IEEE 1362 / ISO 29148 | 1. Definición |
| DOC-02 | Análisis regulatorio y SORA | Categoría EASA (Specific), SORA 2.5: GRC/ARC, SAIL, OSOs, U-space, requisitos nacionales (DGAC / AESA) | Reg. UE 2019/947, 2019/945, 2021/664 | 1. Definición |
| [DOC-03](DOC-03-requisitos.md) | Especificación de requisitos del sistema (SRS) | Requisitos funcionales, prestaciones, seguridad, entorno, interfaces; con ID y método de verificación | ISO 29148 / ARP4754A | 1. Definición |
| [DOC-04](DOC-04-fha.md) | Evaluación de seguridad (FHA + PSSA ligera); hoy solo la FHA v0.2 | Condiciones de fallo, severidad, contingencias (pérdida de enlace, GNSS, batería, motor), geofencing, paracaídas/FTS | ARP4761A / SORA | 1. Definición |
| DOC-05 | Arquitectura del sistema (SAD) | Descomposición HW/SW, reparto PX4 ↔ ROS 2, estación de tierra, nube/flota, diagramas de bloques y de despliegue | ARP4754A / SysML | 2. Diseño |
| DOC-06 | Documento de control de interfaces (ICD) | uXRCE-DDS (tópicos uORB↔ROS 2), MAVLink, API de pedidos, enlace C2, mecanismo de entrega | — | 2. Diseño |
| DOC-07 | Diseño de software ROS 2 | Nodos, tópicos, servicios, acciones, máquina de estados de misión, gestión de lifecycle, QoS | ROS 2 / REP | 2. Diseño |
| DOC-08 | Especificación hardware y presupuestos | Airframe, motores, batería, sensores, companion computer; presupuestos de masa, potencia, autonomía y enlace | — | 2. Diseño |
| DOC-09 | Plan de gestión de configuración y desarrollo | Repos, ramas, versiones de PX4/ROS 2, CI, parámetros PX4 bajo control, estándar de código | Inspirado en DO-178C (SCMP/SDP) | 3. Ejecución |
| DOC-10 | Plan de simulación (SITL/HITL) | Gazebo, mundos urbanos, escenarios nominales y de fallo, criterios de paso | — | 3. Ejecución |
| DOC-11 | Plan y procedimientos de verificación | Matriz de trazabilidad requisito → prueba, campañas SIL/HIL/vuelo | ARP4754A | 3. Ejecución |
| DOC-12 | Manual de operaciones (OM) y procedimientos de emergencia | Checklists, roles del piloto remoto, límites operativos, mantenimiento | Requisito SORA (OSO) | 4. Operación |
| DOC-13 | Plan de proyecto y registro de riesgos | Hitos, presupuesto, riesgos técnicos/regulatorios/negocio | — | Transversal |
| DOC-14 | Registro de decisiones (ADR) | Decisiones de arquitectura con alternativas y motivo | — | Transversal |

Como el proyecto sigue ARP4754A, DOC-09 se amplía al juego de planes que pide la norma (§5.0 y apéndices): Plan de desarrollo, Plan del programa de seguridad, Plan de validación de requisitos, Plan de verificación, Plan de gestión de configuración y Plan de aseguramiento de proceso. Están en la pestaña DOC-09 Planes.

## Supuestos y preguntas abiertas

El riesgo técnico principal es la suelta con paracaídas: con 8 m/s de viento, un paquete de 1 kg soltado a 20 m puede derivar decenas de metros, lo que obliga a zonas de entrega amplias o a compensar la deriva.

**Supuestos de trabajo (v0.1)**

- Proyecto de aprendizaje: DOC-02 (regulación) y DOC-12 (operaciones) se hacen como análisis "como si", no como solicitud real.
- En la UE soltar objetos no está permitido en categoría Abierta; una operación BVLOS urbana con suelta sería categoría Específica con SORA.
- Stack: PX4 v1.17 , ROS 2 Jazzy, Gazebo Harmonic, uXRCE-DDS, QGroundControl.
- Una sola aeronave (1:1); la gestión de flota queda fuera de v1.

**Decisiones tomadas (25/09/2026)**

| # | Tema | Decisión |
| --- | --- | --- |
| D1 | Plataforma | Objetivo: llegar a hardware real. Componentes por decidir (estudio de alternativas en DOC-08 / ADR) |
| D2 | Escenario | Toulouse como referencia; sistema parametrizable para otras ciudades |
| D3 | Punto de entrega | Coordenada GNSS, sin marcador visual |
| D4 | Percepción | Sin cámara en v1 |
| D5 | Alcance por fases | Fase 1 dron + tierra, fase 2 multi-dron, fase 3 pedidos |
| D6 | Rigor | Proceso completo según ARP4754A (y ARP4761A para seguridad), sin certificación real |
| D7 | Terminación de vuelo | Solo FTS (corte de motores), sin paracaídas de aeronave |
| D8 | Escala de severidad | MOC Light-UAS.2510 de EASA |
| D9 | Herramientas | Híbrido: requisitos y trazabilidad en Git con Doorstop + arquitectura en Eclipse Papyrus (SysML 1.6) |
| D10 | Presupuesto HW | Menos de 1.000 € para el prototipo |

**Pendiente para más adelante (backlog)**

- [ ] Cámara para verificar zona libre antes de la suelta (y aterrizaje de precisión)
- [ ] Gestión multi-dron y U-space (fase 2)
- [ ] Sistema de pedidos (fase 3)
- [ ] Selección de componentes hardware

