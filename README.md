# drone-docs

Documentación y requisitos del dron de reparto urbano de última milla (ROS 2 Jazzy + PX4 v1.17).

> Proyecto de aprendizaje personal. Sigue un proceso inspirado en ARP4754A/ARP4761A, **sin certificación**.

- **Nivel de control:** CC1 (control completo), según el plan de configuración (DOC-09 §5).
- **Estado:** esqueleto. La documentación viva (DOC-xx) está en Claude Docs, no en ficheros de este repo.

## Contenido

| Ruta | Qué es |
| --- | --- |
| `requirements/` | Requisitos en texto, con trazabilidad. Se usará [Doorstop](https://doorstop.readthedocs.io/); aún vacío |

## Cómo se relaciona con los otros repos

- [`drone-ros`](https://github.com/XabMS/drone-ros): software (CC1).
- [`drone-px4`](https://github.com/XabMS/drone-px4): fork de PX4 con `drop_guard` (CC1).
- [`drone-sim`](https://github.com/XabMS/drone-sim): entorno de simulación y `drone.repos` (CC2).

`drone-model` (SysML) y `drone-hw` (BOM, esquemas) se crearán cuando haya contenido.

## Baselines

Los tags de baseline siguen el formato `vFASE.BASELINE.N` (por ejemplo `v1.PDR.0`) y corresponden a las revisiones
SRR, PDR, CDR y TRR/FRR. Todavía no hay ninguno.

## Licencia

Apache-2.0 (ver [`LICENSE`](LICENSE)).
