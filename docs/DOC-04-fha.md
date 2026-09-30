# DOC-04 FHA de aeronave (borrador v0.2)

Sep 30, 2026 · @Xabier

Evaluación de peligros funcional de la aeronave (ARP4761A, FHA de nivel aeronave). Entradas: funciones F1–F9 (DOC-01 §Análisis funcional) y ConOps. Salidas: condiciones de fallo (FC) con severidad, requisitos de aeronave (AR) que las cubren (DOC-03) y puntos abiertos para la PASA/PSSA (DOC-09 §2).

**Cambios respecto a la v0.1 (25/09/2026)**

- Extraída del índice a este fichero.
- Nuevas FC-13 a FC-18: motor, batería, companion (dos casos), colisión y motores en marcha en tierra. Las contingencias «fallo del companion» y «fallo de motor» del ConOps §5 no tenían FC propia.
- Corregida la nota sobre el companion: puede causar FC-08 (§4).
- FC-19 y FC-20 (origen malicioso), añadidas desde el análisis de seguridad de la información (DOC-15). Los ataques son causas de las FC anteriores salvo dos efectos nuevos: la orden de mando falsa y la configuración alterada.
- Cada FC lleva la justificación de su severidad y los AR que la cubren. FC-01 y FC-06 no tenían ninguno; se añaden AR-021 a AR-027 en DOC-03 (propuestos).
- Nuevo §3 con el objetivo de fallo simple y qué FC lo incumplen por diseño.

## 1. Escala de severidad (D8)

Escala del MOC Light-UAS.2510 de EASA, definida por el efecto en terceros en tierra y en aire.

| Severidad | Efecto principal |
| --- | --- |
| Catastrófico | Posible fatalidad de terceros (en tierra o en aire) |
| Peligroso | Posibles lesiones graves a terceros, o gran reducción de los márgenes de seguridad |
| Mayor | Reducción significativa de márgenes o aumento importante de la carga del piloto remoto; lesiones leves posibles |
| Menor | Ligera reducción de márgenes; sin lesiones |

**Pendiente de cotejo (PA-02):** el texto anterior es un resumen de trabajo. El 30/09/2026 solo se pudo consultar un resumen de búsqueda del MOC (versión final del 11/02/2025), no el documento; los objetivos cualitativos de §3 salen de ese resumen. El propio MOC remite a ED-280 para la FHA y su aplicabilidad por SAIL no está clara (el resumen dice SAIL IV, el nombre del fichero dice SAIL V y VI). Hasta cotejarlo con el texto oficial, ningún nivel de este documento pasa a baseline.

**Orden de magnitud del peligro:** a 4 kg y 12 m/s la energía cinética ronda los 290 J; en caída libre desde 100 m, con velocidad terminal estimada de 20–25 m/s, pasa de 800 J. Ambos valores superan de sobra el umbral de 80 J que usa la normativa europea como referencia de riesgo para personas. Para el paquete de 1 kg, 80 J corresponden a unos 12,6 m/s de impacto (½·m·v²); es la referencia para TBD-6.

## 2. Condiciones de fallo

De 20 condiciones, 8 salen catastróficas (FC-01, 08, 11, 12, 13, 14, 17 y 19). Sin paracaídas de aeronave (D7), la protección tiene que venir de la contención, del FTS y de planificar rutas que no sobrevuelen multitudes.

| ID | Función | Condición de fallo | Fase de vuelo | Efecto | Severidad propuesta |
| --- | --- | --- | --- | --- | --- |
| FC-01 | F1 | Pérdida de control en vuelo | Todas | Caída incontrolada sobre terceros | Catastrófico |
| FC-02 | F2 | Posición errónea no detectada | Crucero | Salida de la zona operacional sin que nadie lo sepa | Peligroso |
| FC-03 | F2 | Pérdida de GNSS detectada | Crucero / aproximación | Contingencia, aterrizaje en zona no prevista | Mayor |
| FC-04 | F4 | Pérdida del enlace C2 detectada | Todas | Sin supervisión; el dron hace RTL | Mayor |
| FC-05 | F5 | Suelta de carga no ordenada | Crucero | Paquete de 1 kg cae sobre terceros | Peligroso |
| FC-06 | F5 | Paracaídas del paquete no se abre | Suelta | Impacto del paquete a alta velocidad | Peligroso |
| FC-07 | F5 | Carga no liberada | Suelta | Regreso con carga; misión fallida | Menor |
| FC-08 | F6 | Pérdida de contención (fly-away) | Todas | Aeronave fuera de control fuera de la zona | Catastrófico |
| FC-09 | F7 | Estimación de batería errónea | Crucero / regreso | Aterrizaje forzoso en zona no controlada | Peligroso |
| FC-10 | F8 | El piloto no puede intervenir (fallo de GCS) | Todas | El dron sigue en autónomo sin supervisión | Mayor |
| FC-11 | F9 | Disparo del FTS no ordenado | Todas | Caída incontrolada | Catastrófico |
| FC-12 | F9 | FTS no actúa cuando se ordena | Emergencia | Se pierde la última barrera ante un fly-away | Catastrófico (combinado con FC-08) |
| FC-13 | F1 | Pérdida de empuje de un motor, ESC o hélice | Todas | Pérdida de control (un cuadricóptero no tolera la pérdida de un motor) y caída | Catastrófico |
| FC-14 | F7, F1 | Pérdida súbita de energía o fallo térmico de la batería de vuelo | Todas | Caída sin empuje, con posible incendio | Catastrófico |
| FC-15 | F3 | Fallo del companion detectado (pérdida de latido, caída del nodo) | Todas | PX4 pierde Offboard y el C2 por LTE (ADR-003); RTL o espera | Mayor |
| FC-16 | F3 | El companion emite consignas u órdenes erróneas sin que se detecte | Todas | Desvío de la ruta, salida del volumen o orden de suelta indebida | Peligroso (si PX4 y el FTS contienen); si no, es FC-08 o FC-05 |
| FC-17 | F2, F6 | Colisión con obstáculo o con una aeronave tripulada | Despegue, crucero, aterrizaje | Caída sobre terceros; posible fatalidad en aire | Catastrófico |
| FC-18 | F1, F8 | Motores en marcha no ordenados en tierra | Prevuelo, hub | Lesiones al operador del hub | Mayor |
| FC-19 | F4, F8 | La aeronave acepta y ejecuta una orden de mando falsa, repetida o alterada (incluidas las que cortan motores o abren la carga) | Todas | Modo o rumbo no ordenados, terminación de vuelo, desarme forzado o suelta indebida | Catastrófico |
| FC-20 | F5, F6 | Parámetros o ficheros que fijan la contención, la geofence o la zona de suelta alterados sin autorización | Prevuelo, todas | Contención o zona de suelta desplazadas; combinada con otro fallo, FC-08 o FC-05 | Peligroso |

## 3. Justificación, mitigaciones y requisitos

Los AR-021 a AR-027 son nuevos y están sin validar (DOC-03 §2). «SR pendientes» quiere decir que el AR no tiene todavía SR hijo; se derivan en la PSSA.

| ID | Por qué esa severidad | Mitigación | AR que la cubren |
| --- | --- | --- | --- |
| FC-01 | Cuatro kilos con más de 290 J sobre terceros pueden causar una fatalidad; sin paracaídas (D7) nada limita la energía del impacto. ADR-005: la severidad no se rebaja, se declara la limitación | Margen de empuje y de gas (SR-PRP-001/002); FTS; rutas por corredores de muy baja densidad | AR-004, AR-021, AR-022 |
| FC-02 | Sin detección, ni el piloto ni la contención saben que el dron está fuera de sitio; no hay por sí sola energía de impacto | Integridad EKF2; geofence del FTS con GNSS distinto | AR-014 |
| FC-03 | Se detecta y hay contingencia definida; el riesgo es el punto de aterrizaje | Failsafe de GNSS de PX4; puntos de aterrizaje de emergencia (`emergency_landing` en F-02) | AR-014 |
| FC-04 | Se detecta y PX4 ejecuta RTL; sube la carga del piloto | Failsafe de pérdida de enlace; enlace redundante (radio + LTE) a estudiar | AR-013 |
| FC-05 | Un paquete de 1 kg cae sobre terceros: lesiones graves posibles, fatalidad poco probable | Doble condición: orden del companion y autorización de `drop_guard` (ADR-007); confirmación del piloto (ADR-006); inhibición fuera de la zona de suelta | AR-007, AR-008 |
| FC-06 | Impacto de 1 kg a alta velocidad sobre la zona de suelta | Zona sin personas (AS-001); altura mínima de suelta; ensayos del paracaídas | AR-027 |
| FC-07 | Sin efecto en terceros | Detección de carga retenida; margen de batería para volver con carga | AR-009 |
| FC-08 | La aeronave fuera de la zona, sin control y sobre zona poblada equivale a FC-01 | Geofence de PX4 (miembro 1) y del FTS con disparo automático (miembro 2); rutas de baja densidad | AR-010, AR-011, AR-021, AR-024 |
| FC-09 | Aterrizaje forzoso en zona no controlada, con lesiones graves posibles | Estimación por energía consumida y tensión; reserva mínima del 20 % | AR-015 |
| FC-10 | El piloto no puede intervenir, pero el dron sigue con sus failsafes | Tratar como pérdida de C2; GCS de respaldo | AR-017, AR-018 |
| FC-11 | Caída de 4 kg con la aeronave en vuelo normal | Armado en dos pasos y persistencia (SR-FTS-003); lógica de disparo simple | AR-012 |
| FC-12 | Es un fallo latente: solo importa si además ocurre FC-08 | Alimentación y receptor independientes; prueba del FTS en cada prevuelo (SR-FTS-007) | AR-011 |
| FC-13 | Un cuadricóptero sin redundancia pierde el control con un motor; la termina un solo fallo simple | Diseño con margen de propulsión; hélices y ESC de calidad; inspección prevuelo. Sin mitigación efectiva: ver PA-01 | AR-021, AR-022 |
| FC-14 | La pérdida de energía deja caer 4 kg; el fallo térmico añade fuego | Vigilancia de temperatura y de tensión por celda; batería y conectores con margen; ZSA/PRA en la CCA | AR-021, AR-023 |
| FC-15 | Se detecta por el latido y PX4 ejecuta una contingencia; no hay pérdida de control | Latido Offboard (SR-MSN-006, SR-FMS-003); alimentación del companion separada (SR-PWR-002); ADR-003 ya recoge que el companion cae en la cadena del C2 | AR-018 |
| FC-16 | Ver §4: mientras PX4 y el FTS limiten la aeronave, el efecto queda en Peligroso | Geofence y límites de PX4; vigilancia de consigna (SR-MSN-011-D); `drop_guard` para la suelta; FTS | AR-018, AR-024 |
| FC-17 | Sin cámara ni detección y evasión (D4) no hay barrera táctica; el choque con una aeronave tripulada puede ser fatal | Solo estratégica: altura máxima 120 m, zonas prohibidas y de autorización, rutas sobre obstáculos conocidos; se evalúa en DOC-02 (ARC) | AR-005, AR-021, AR-025 |
| FC-18 | Hélices girando cerca del operador: cortes graves posibles, pero afectan a la tripulación, no a terceros | Armado explícito con comprobaciones; procedimientos del hub (DOC-12); hélices retiradas al manipular | AR-026 |
| FC-19 | Equivale a FC-11 por otra vía: PX4 acepta por MAVLink la terminación y el desarme forzado en vuelo sin comprobar el origen (DOC-15, hallazgo 3) | VPN con clave por dispositivo; autenticación hasta la FMU; sin terminación por el C2; identificador y caducidad en las órdenes irreversibles | AR-028 |
| FC-20 | Con el FTS cargado con el mismo volumen, un cambio de parámetros no basta por sí solo para la caída, pero anula una barrera | Parámetros inmutables con el dron armado; hash criptográfico de la configuración; registro | AR-029 |

**Requisitos sin SR hijo:** AR-021 a AR-027 quedan «SR pendientes (PSSA)». AR-021 se apoya ya en SR-MSN-002 y AR-024 en SR-MSN-011-D y SR-FMS-001/002, que hay que reasignar al validarlos.

## 4. El companion y las condiciones catastróficas

La v0.1 decía que el companion no aparecía como fuente directa de condiciones catastróficas. Es incorrecto: un companion que envía consignas erróneas (FC-16) puede sacar al dron de su volumen y así causar FC-08, o ordenar una suelta indebida y causar FC-05. Lo que permite asignarle IDAL C es que **la contención no depende de él**:

- Cada consigna pasa por PX4, que aplica su geofence, sus límites de altura y sus failsafes (ADR-001).
- El FTS decide con su GNSS y su enlace, sin ningún dato del companion (ADR-002).
- La suelta requiere además la autorización de `drop_guard` (ADR-007).

Esa independencia hay que demostrarla en la CCA (modo común: la posición del EKF2 sirve tanto a la geofence de PX4 como a `drop_guard`, y su fallo es FC-02). AR-024 lo convierte en requisito. Si alguna vez el companion pasara a poder anular un límite de PX4, este análisis se reabre.

## 5. Objetivos cualitativos y cumplimiento

Según el resumen del MOC Light-UAS.2510 (PA-02): una condición catastrófica no debe producirse por un fallo simple y debe ser extremadamente improbable; una peligrosa, extremadamente remota; una mayor, remota. No se usan probabilidades numéricas hasta la PASA (DOC-09 §2).

| FC | Cumple el objetivo de fallo simple | Por qué |
| --- | --- | --- |
| FC-01 | No, por diseño | Es una función sin descomponer (ADR-005); PX4 no puede demostrar DAL A |
| FC-08 | Sí, si la CCA lo confirma | Dos miembros independientes: geofence de PX4 y FTS |
| FC-11 | Por confirmar | Depende de que ninguna señal aislada dispare el FTS (SR-FTS-003) |
| FC-12 | Sí, con FC-08 | Necesita dos fallos; latente, por eso la prueba prevuelo |
| FC-13 | **No, por diseño** | Un motor o ESC es un fallo simple y el cuadricóptero no lo tolera |
| FC-14 | **No, por diseño** | Un fallo de la batería o de su conector es simple y no hay segunda fuente |
| FC-17 | Solo por exposición | No hay barrera a bordo; se reduce la probabilidad, no el efecto |
| FC-19 | Por confirmar | Hoy un solo mensaje válido para PX4 basta; depende de SR-SEC-002 y SR-SEC-003 |

FC-13 y FC-14 son huecos nuevos: ADR-005 solo consideraba FC-01 y la opción D (hexáptero u octóptero) la descartaba por peso y coste. Con los objetivos del MOC, lo honesto en este proyecto de aprendizaje es declararlos, como se hizo con F1, y registrarlo en un ADR (PA-01).

## 6. Combinaciones para la PSSA

- FC-08 con FC-12: ya recogida; el FTS es una barrera latente.
- FC-16 con fallo del geofence de PX4 (FC-02 o error de parámetros): la contención queda solo en el FTS.
- FC-15 con FC-04: el companion lleva el módem LTE (ADR-003), así que su caída pierde también el C2.
- FC-04 con FC-03: causa común posible en el entorno RF; hay que ver si un mismo evento (interferencia, jamming) inutiliza LTE y GNSS a la vez.
- FC-14 con todas: una pérdida de energía es causa común de casi todas las barreras a bordo salvo el FTS con batería propia (ZSA/PRA).

## 7. Puntos abiertos

- **PA-01.** FC-13 y FC-14 incumplen el objetivo de fallo simple (§5). Opciones: declarar la limitación como en ADR-005 (A + B), pasar a hexáptero u octóptero (opción D), o reconsiderar el paracaídas de aeronave (D7). Se propone un ADR nuevo tras la revisión de este documento.
- **PA-02.** Cotejar la escala y los objetivos con el texto oficial del MOC Light-UAS.2510 y con ED-280 (§1).
- **PA-03.** Sin datos de densidad de población de las rutas de Toulouse (AS-002): no se puede argumentar todavía la rebaja del riesgo de ADR-005.
- **PA-04.** Los ataques informáticos se tratan en DOC-15. Lo que quede abierto allí (ADR A, B y C, GNSS) puede reabrir FC-05, FC-08 y FC-11.
- **PA-05.** TBD-8 (tiempo de armado sin despegue, AR-026) y TBD-9 (separación vertical sobre obstáculos, AR-025) sin valor.
- **PA-06.** Alimentación de la FMU y del companion: ¿camino único desde el módulo de potencia? Es una pregunta para la ZSA.
