# Documentación del dron de reparto urbano (ROS 2 + PX4)

Exportación de la documentación existente a fecha 29/09/2026 (versión v0.1, borradores). Los originales se redactaron como documento vivo en Claude Docs; a partir de aquí, la fuente de verdad es este repositorio.

| Fichero | Contenido |
| --- | --- |
| [00-indice-y-alcance.md](00-indice-y-alcance.md) | Propósito, lista de los 15 documentos, supuestos de trabajo y decisiones D1–D10 |
| [DOC-01-conops.md](DOC-01-conops.md) | ConOps (borrador v0.1) y análisis funcional F1–F11 (extraído del índice el 30/09/2026) |
| [DOC-03-requisitos.md](DOC-03-requisitos.md) | Requisitos de aeronave (AR), hipótesis (AS), valores TBD y requisitos de sistema (SR) |
| [DOC-04-fha.md](DOC-04-fha.md) | FHA de aeronave (borrador v0.2): condiciones de fallo FC-01…FC-20, justificación, mitigaciones y puntos abiertos |
| [DOC-05-arquitectura.md](DOC-05-arquitectura.md) | Arquitectura del sistema (SAD), ADR-001 a ADR-009 |
| [DOC-06-icd.md](DOC-06-icd.md) | Control de interfaces: uXRCE-DDS, MAVLink, RC, FTS, interfaces ROS 2 |
| [DOC-08-hardware.md](DOC-08-hardware.md) | Estudio de hardware, presupuesto, masa y energía |
| [DOC-09-planes.md](DOC-09-planes.md) | Planes ARP4754A: desarrollo, seguridad, validación, verificación, configuración, proceso, cumplimiento |
| [DOC-10-simulacion.md](DOC-10-simulacion.md) | Plan de simulación (SITL), escenarios SIM-01…SIM-20, hitos S1–S7 |
| [DOC-15-seguridad-informacion.md](DOC-15-seguridad-informacion.md) | Modelo de amenazas (STRIDE), controles, requisitos SR-SEC derivados y casos de prueba de seguridad de la información |
| [design/drop_guard-diseno.md](design/drop_guard-diseno.md) | Diseño del módulo PX4 `drop_guard` (ADR-007) |
| [project/estado-proyecto.md](project/estado-proyecto.md) | Estado del proyecto, decisiones y hechos comprobados |
| [project/HANDOFF_git_github.md](project/HANDOFF_git_github.md) | Handoff de organización Git/GitHub (ya ejecutado; histórico) |

Documentos aún sin escribir: DOC-02, DOC-07, DOC-11, DOC-12, DOC-13 y DOC-14 (el registro de ADR vive hoy en DOC-05 §8). De DOC-04 solo existe la FHA; faltan la PASA/PSSA y el análisis de causa común.

Notas de la exportación: los «chips» del editor (fecha, mención, referencia) quedan como texto plano; los diagramas Mermaid se conservan en bloques ```mermaid y las fórmulas de drop_guard se pasaron a bloques ```math, que GitHub renderiza; los caracteres `\_` y `\~` son escapes de la exportación y se pueden limpiar más adelante.
