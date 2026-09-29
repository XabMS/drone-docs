# Dron de reparto urbano (ROS 2 + PX4): estado

Documento vivo principal: https://claude.ai/code/artifact/957a1dd0-7d9d-44a2-b4a6-64dd19e73c6d (pestañas: principal, "DOC-09 Planes", "DOC-03 Requisitos", "DOC-05 Arquitectura", "DOC-08 Hardware", "DOC-10 Simulación", "DOC-06 ICD", "drop_guard (diseño)")
Código (29/09/2026): repos en GitHub (cuenta XabMS): drone-ros (paquetes ROS 2, ops/, missions/), drone-px4 (fork de PX4, rama drone), drone-sim (instalación nativa, drone.repos, pruebas, legacy/ con lo del contenedor), drone-docs (esta documentación). Sustituyen a drone-sim.zip y a la carpeta ~/drone-sim
Handoff de organización Git/GitHub: project/HANDOFF_git_github.md (ejecutado; histórico)

## Decisiones
- D1 Objetivo: llegar a HW real
- D2 Toulouse como referencia; sistema parametrizable; segunda ciudad de prueba: Donostia
- D3 Punto de entrega por coordenada GNSS
- D4 Sin cámara en v1 (backlog)
- D5 Fases: 1 dron + tierra, 2 multi-dron/U-space, 3 pedidos
- D6 Proceso ARP4754A + ARP4761A, sin certificación (aprendizaje)
- D7 Solo FTS (corte de motores), sin paracaídas de aeronave
- D8 Escala de severidad MOC Light-UAS.2510
- D9 Requisitos en Git con Doorstop + arquitectura en Eclipse Papyrus (SysML 1.6)
- D10 Presupuesto HW < 1.000 € (sin emisora ni cargador previos)
- D11 GitHub + GitHub Actions
- PX4 v1.17.0 (última estable a 25/09/2026) + px4_msgs release/1.17; la doc dice "PX4 v1.17" (corregido el 29/09/2026 donde ponía v1.15+)
- Sin contenedor (decidido 29/09/2026): entorno de simulación en instalación nativa, solo soportada en Ubuntu 24.04 (ROS 2 Jazzy). Se usa el portátil Lenovo IdeaPad con Ubuntu 24.04 (gráfica integrada → headless, SIH por defecto). PC Fedora (RX 7600) descartado por ahora por no ser plataforma soportada de Jazzy
- Entorno nativo: clon de drone-sim en ~/drone/drone-sim con dependencias en .deps/ (fork drone-px4 en el tag fijado en drone.repos, agente XRCE v2.4.3 con wrapper, px4_msgs en workspace aparte, drone-ros clonado en ros2_ws/src/drone-ros); separado de otros proyectos, con los que comparte solo /opt/ros/jazzy. Docker/run_container.sh quedan en drone-sim/legacy (descartado)
- CI (29/09/2026): drone-ros ejecuta build-test (colcon build + 64 tests) en cada PR; drone-sim ejecuta lint en PR y nightly (setup_native.sh completo + check_native.sh, diario y a demanda; cabe en el runner estándar: ~23 min con ccache). drone-px4 sin Actions
- GitHub (29/09/2026, ejecutado): repos públicos en GitHub Free (nada secreto): drone-ros, drone-px4 (fork de PX4 con drop_guard como commits en la rama drone; tag de trabajo v1.17.0-1.0.0, formato exigido por el validador de versión de PX4; no es una baseline), drone-sim, drone-docs; drone-model y drone-hw cuando haya contenido; licencia Apache-2.0 (PX4 conserva BSD-3); drone.repos en drone-sim fija las versiones; main protegida con ruleset (PR + CI, historial lineal, sin aprobaciones ni bypass); commits y package.xml con el correo noreply de GitHub. Aún no hay ninguna baseline (vFASE.BASELINE.N)
- La documentación se versiona en Git (drone-docs/docs) desde el 29/09/2026; los documentos vivos de Claude Docs pasan a ser copia histórica
- DOC-10 y DOC-09 actualizados el 29/09/2026 (sin menciones al contenedor)
- Hub de simulación = terreno de aeromodelismo de Blagnac; coordenadas pendientes (ops usa centro de Toulouse provisional); zonas de ejemplo no reales
- Carga ≤1 kg, MTOM <4 kg, entrega por suelta con paracaídas, autónomo + piloto supervisor 1:1
- Reparto PX4/ROS 2: ROS 2 gestiona toda la secuencia de suelta; PX4 mueve el servo y drop_guard solo veta

## ADR (DOC-05)
- ADR-001 PX4 autoridad final (aceptada)
- ADR-002 FTS independiente (propuesta)
- ADR-003 C2 por LTE/4G vía companion (aceptada)
- ADR-004 Si falla el FTS, motores siguen + RTL (aceptada)
- ADR-005 F1 DAL A: declarar limitación PX4 + rutas de muy baja densidad (aceptada)
- ADR-006 Confirmación de suelta desde QGroundControl por LTE (BVLOS); CH7 de RC solo en ensayos VLOS (aceptada)
- ADR-007 F5 (FDAL B) = payload_manager ROS 2 (C) + módulo drop_guard en fork de PX4 (C) (aceptada)
- ADR-008 FMU-companion por UART (propuesta)
- ADR-009 AR-003: 5 km objetivo de sistema (simulación); prototipo fase 1 ≥1,5 km (aceptada)

## PX4 v1.17 — hechos comprobados
- Tópicos DDS con sufijo _vN si MESSAGE_VERSION != 0 (vehicle_status_v1, vehicle_local_position_v1, battery_status_v1, home_position_v1); confirmado en vivo. vehicle_global_position NO lleva sufijo
- QoS PX4: writers BEST_EFFORT + TRANSIENT_LOCAL (depth 0), readers BEST_EFFORT + VOLATILE
- home_position solo se publica al cambiar: se pierde si el agente conecta tarde; PX4 lo refija al armar
- Pausa de QGroundControl = DO_REPOSITION al punto actual: no cambia nav_state (sigue AUTO_LOITER)
- SIH: posición inicial con PX4_PARAM_SIH_LOC_LAT0/LON0/H0; parámetros por env PX4_PARAM_<nombre>
- SIH arranca con `make px4_sitl sihsim_quadx`; px4-rc.sihsim también lee PX4_HOME_LAT/LON/ALT (SIH_LOC_* son float en grados)
- El parche drop_guard aplica limpio sobre el tag v1.17.0 (comprobado 29/09/2026)

## drop_guard
- DropGuardLogic: 38 tests, 100 % sentencias/ramas/decisiones; módulo sobre v1.17.0 (parche 15 ficheros); SITL SIH 23/24
- El parche añade msg/DropGuardStatus.msg: px4_msgs release/1.17 no lo trae → en S3 generar px4_msgs desde el msg/ del fork
- En el portátil, PX4 con drop_guard arranca y publica el tópico DDS drop_guard_status (visto en el log del 29/09/2026)
- Abiertos: valores EPH/EPV reales; algoritmo DG_ZONE_HASH (drone_core ya calcula drop_zone_hash = CRC32 canónico)

## Hito S2 (terminado 25/09/2026)
- Paquetes: drone_core (C++ sin ROS: geo con NED↔ENU, geometría, ops YAML+GeoJSON, misiones, CRC32, herramienta drone_validate), drone_interfaces (MissionState, ActiveConfig, PayloadState, NodeHealth, HealthStatus, EnergyEstimate, MissionCommand.srv, DropPayload.action), drone_mission (MissionStateMachine pura + config_manager + mission_manager + s2.launch.py)
- Misión por tramos con VehicleCommand DDS: ARM, NAV_TAKEOFF, DO_REPOSITION (GOTO confirmado por position_setpoint_triplet, 3 intentos), NAV_RETURN_TO_LAUNCH
- CONTINGENCY si failsafe PX4, modo inesperado >3 s o consigna distinta del objetivo >1 s (detecta Pausa del piloto)
- ops/toulouse, ops/donostia (+geojson), missions/tls_demo_01, dss_demo_01; scripts/s2_scenarios.py
- Verificado en la nube con ROS 2 Jazzy compilado desde fuente + PX4 SITL SIH + XRCE agent: 64 tests; SIM-01, SIM-19 (pausa 1,2 s, RTL 1,4 s), SIM-20 pasan
- Nuevos SR derivados: SR-MSN-011-D (vigilancia de consigna, AR-017), SR-MSN-012-D (home tras armado)
- Desviaciones: sin carga de misión/geofence por MAVLink (SR-MSN-003 → S4), nodos no lifecycle, energía = max_route_m; leg_speed_mps 3 m/s

## Entorno nativo en el portátil (verificado 29/09/2026)
- setup_native.sh completo sin errores; check_native.sh: 15 correctas, 0 fallidas (Ubuntu 24.04.5, Gazebo 8.15.0 = Harmonic, ROS Jazzy, SIH; 64 tests correctos; origen SIH = ops/toulouse.yaml; vehicle_status_v1 con datos, nav_state=4)
- Fallos menores de la primera ejecución de setup_native.sh, ya corregidos en drone-sim: falso aviso del agente XRCE (pipefail) y numpy 2.x tapando al del sistema en .deps/pylibs (ahora numpy<2). Verificado en clon limpio: setup sin avisos y check_native.sh con 15 correctas, 0 fallidas y 64 tests
- Pendiente de probar en el portátil: escenarios completos (s2_scenarios.py sim01, sim19_pause, sim19_rtl, sim20) y sitl_drop_guard_test.py; SIM=gz (Gazebo) con la gráfica integrada

## Hito S1
- RUNBOOK_S1.md (Docker, legado; ahora en drone-sim/legacy); ya no se necesitan sus resultados: los sustituye la comprobación nativa

## Documentos
- DOC-03: AR-001..AR-020, AS-001..005, TBD-1..7; SR v0.1: 45 + 2 derivados
- DOC-06 ICD v0.1; DOC-08 Hardware v0.1; DOC-10 Simulación v0.1 (S2 marcado terminado; §2, §6, §8 y estado actualizados a entorno nativo el 29/09/2026; su párrafo de estado dice que los scripts están pendientes de ejecutar: ya se ejecutaron)

## Preguntas abiertas
- Coordenadas del terreno de Blagnac; GNSS FTS no u-blox; UARTs/GPIO RPi5; 868 MHz duty cycle; MAVSDK; comando MAVLink de confirmación de suelta

## Siguiente
- Probar en el portátil los escenarios completos (s2_scenarios.py) y sitl_drop_guard_test.py; después, hito S3 (suelta real; generar px4_msgs desde el msg/ del fork drone-px4 porque release/1.17 no trae DropGuardStatus.msg)
- Ejecutar los escenarios S2 en el portátil (SIM-01, SIM-19, SIM-20) y la prueba de integración de drop_guard
- S3 (payload_manager + drop_guard desde el fork + px4_msgs del fork + confirmación de suelta)
