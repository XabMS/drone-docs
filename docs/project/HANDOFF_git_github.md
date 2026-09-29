# Handoff para Claude Code: repositorios Git y GitHub del dron de reparto

> **Estado: ejecutado el 29/09/2026 (documento histórico).** Los cuatro repos existen y `main` está protegida.
> Diferencias respecto a lo que dice el texto de abajo:
>
> - El tag de trabajo de `drone-px4` es `v1.17.0-1.0.0`, no `v1.17.0-drone.1` (luego `v1.17.0-1.0.0`): el validador de versión de PX4 exige que la versión custom sea numérica (`x.y.z`). `drone.repos` fija `v1.17.0-1.0.0`.
> - Se usó HTTPS con `gh` en lugar de SSH.
> - `drone-sim` tiene además un workflow `lint` (es el check que exige su ruleset en PR); `drone-ros` ejecuta `colcon test` solo sobre `drone_core` y `drone_mission` (64 tests; `drone_s1_demo` no tiene tests).
> - El correo personal del usuario se sustituyó por el *noreply* de GitHub también en los `package.xml` (ver la Fase 1) y los repos `drone-ros`, `drone-sim` y `drone-docs` se recrearon limpios para que no quedara en el historial.
> - Verificación en clon limpio: `check_native.sh` con 15 correctas, 0 fallidas y 64 tests; el `nightly` completo tarda unos 23 min en el runner estándar.

Este documento es autosuficiente: quien lo ejecute no tiene el contexto del chat donde se preparó. Se lo da Xabi a Claude Code, que trabaja en su portátil (Ubuntu 24.04), en `~/drone-sim`.

## 1. Objetivo

Pasar el código actual del proyecto (un solo directorio `drone-sim`, sin Git) a **cuatro repositorios públicos en GitHub Free**, con la disciplina de configuración que fija el plan del proyecto (§5 de DOC-09), CI mínima y protección de `main`. Al terminar, un clon limpio de `drone-sim` debe reconstruir el entorno completo con un solo script, con las versiones fijadas por un fichero `drone.repos`.

El proyecto es un aprendizaje personal: un dron de reparto urbano de última milla con ROS 2 Jazzy y PX4 v1.17, con proceso ARP4754A/ARP4761A sin certificación. **No hay nada secreto**, por eso los repos son públicos (decisión del usuario).

## 2. Decisiones ya tomadas (no reabrir)

- Repos **públicos** en GitHub Free. Así las ramas protegidas, los rulesets y los runners estándar de Actions son gratis.
- Cuatro repos ahora: `drone-ros`, `drone-px4`, `drone-sim`, `drone-docs`. `drone-model` (SysML) y `drone-hw` (BOM, esquemas) se crean cuando haya contenido; **no** crear repos vacíos para ellos.
- Sin contenedores. El entorno se instala de forma nativa (solo Ubuntu 24.04, ROS 2 Jazzy). Docker queda como legado.
- Licencia **Apache-2.0** para `drone-ros`, `drone-sim` y `drone-docs` (ya es la de los `package.xml`). `drone-px4` conserva la licencia de PX4 (BSD-3-Clause) tal cual.
- Versiones: PX4 **v1.17.0**, px4_msgs rama `release/1.17`, Micro XRCE-DDS Agent `v2.4.3`.
- PX4 es un **fork** (`drone-px4`), con el módulo `drop_guard` como commits reales en una rama, no como fichero `.patch`.
- Un solo ingeniero: `main` protegida exige pull request y checks en verde, pero **no** aprobaciones de otra persona (no se puede autoaprobar).

## 3. Reglas del plan de configuración (DOC-09 §5) que hay que reflejar

- Todo lo que define el sistema vive en Git. Elementos **CC1** = control completo (baseline y cambios con informe de problema); **CC2** = versionado sin proceso formal.
- `main` protegida; una rama de trabajo por cambio; merge solo con CI en verde y autorrevisión con checklist en la PR.
- Tags de baseline con el formato `vFASE.BASELINE.N` (por ejemplo `v1.PDR.0`). Las baselines corresponden a las revisiones SRR, PDR, CDR, TRR/FRR. **No crear ningún tag de baseline ahora**: aún no hay revisión.
- Cada cambio en un elemento CC1 empieza por un issue (informe de problema o petición de cambio) con: descripción, análisis de impacto (requisitos, seguridad, pruebas afectadas), decisión y verificación de cierre.
- Los parámetros de PX4 son configuración: se exportan, se comparan con la baseline y se archivan con el ULog del vuelo.
- Las versiones de PX4, ROS 2, Gazebo, agente XRCE y SO se fijan exactas.
- Tabla de repos del plan: `drone-docs` (CC1), `drone-model` (CC1), `drone-ros` (CC1), `drone-px4` (CC1), `drone-sim` (CC2), `drone-hw` (CC1).

## 3b. Precondiciones (Fase 0): comprobar antes de tocar nada

1. `~/drone-sim/scripts/setup_native.sh` terminó sin errores, y `source ~/drone-sim/env_native.sh && ./scripts/check_native.sh` da **0 fallidas** (`logs/check_native.txt`). Si no, **para y avisa al usuario**: la restructuración debe partir de un estado que funciona.
2. `gh auth status` está autenticado en la cuenta del usuario (si no, pide al usuario que ejecute `gh auth login` con SSH; es interactivo).
3. Lee `~/drone-sim/README_NATIVE.md` y las cabeceras de `scripts/setup_native.sh`, `env_native.sh`, `scripts/check_native.sh` y `scripts/start_sim.sh`: los vas a adaptar.

## 4. Estado actual y reparto

El directorio `~/drone-sim` no es un repo Git. Contiene:

| Ruta actual | Destino |
| --- | --- |
| `ros2_ws/src/drone_core`, `drone_interfaces`, `drone_mission`, `drone_s1_demo` | `drone-ros`, en la raíz del repo (una carpeta por paquete) |
| `ops/` (toulouse y donostia, YAML + GeoJSON) y `missions/` | `drone-ros`, en la raíz (`ops/`, `missions/`). Son datos que cambian el comportamiento del sistema: `config_manager` verifica su hash |
| `px4_patches/drop_guard_px4_v1.17.0.patch` | Rama `drone` del fork `drone-px4` como commits reales (ver §5, Fase 3). El `.patch` no se conserva |
| `px4_patches/sitl_drop_guard_test.py` | `drone-sim/tests/sitl_drop_guard_test.py` |
| `scripts/setup_native.sh`, `check_native.sh`, `start_sim.sh`, `s2_scenarios.py`, `env_native.sh`, `README_NATIVE.md`, `README_S2.md` | `drone-sim` |
| `docker/`, `scripts/build_image.sh`, `run_container.sh`, `check_host.sh`, `collect_results.sh`, `RUNBOOK_S1.md`, `README.md` (S1) | `drone-sim/legacy/`, con una nota en su README de que es del enfoque con contenedor, descartado |
| `.deps/`, `logs/`, `ros2_ws/build`, `install`, `log` | **No se versionan** (ya están en `.gitignore`) |

`drone-docs`: por ahora un esqueleto (README, LICENSE, `.gitignore`, carpeta `requirements/` con un `README` que diga que se usará Doorstop). La documentación viva está en Claude Docs, no en ficheros; **no inventes contenido**.

## 5. Tareas

Trabaja **en un directorio nuevo** (`~/gh-staging/`) y deja `~/drone-sim` intacto hasta verificar (Fase 6). No hagas `git push --force` en ningún momento. Idioma de commits, plantillas y README: español, en imperativo y con mensajes claros.

### Fase 1: identidad y cuenta

- Obtén el login y el id: `gh api user -q '.login, .id'`.
- Configura Git para estos repos con el correo *noreply* de GitHub (`<id>+<login>@users.noreply.github.com`) y `user.name` = el nombre del perfil. Los repos son públicos: **no uses el correo personal del usuario como autor de commits**. Recomienda al usuario activar en GitHub *Settings → Emails → Block command line pushes that expose my email*.
- Los `package.xml` llevaban el correo personal del usuario como mantenedor (práctica normal en ROS). Por ser repos públicos, se cambió al correo *noreply* de GitHub.

### Fase 2: repos vacíos y esqueleto

Crea `drone-ros`, `drone-sim` y `drone-docs` con `gh repo create <login>/<nombre> --public --description "..."`, sin plantillas de GitHub (los ficheros los añades tú). Cada repo lleva: `README.md` (qué es, qué nivel de control tiene, cómo se relaciona con los otros), `LICENSE` (Apache-2.0, texto oficial), `.gitignore` adecuado, rama por defecto `main`.

Añade a los tres, en `.github/`:

- `pull_request_template.md`: checklist de autorrevisión. Ítems: issue enlazado; requisitos AR/SR afectados; tests añadidos o actualizados; CI en verde; impacto en seguridad (FHA) revisado; documentación actualizada; revisado al menos 48 h después de escribirlo, según el plan (o marcar la excepción y el motivo).
- `ISSUE_TEMPLATE/problema-o-cambio.yml`: formulario con los cuatro campos del plan (descripción, análisis de impacto, decisión, verificación de cierre) y un desplegable de nivel de control (CC1/CC2) y otro de DAL (A, B, C, D, sin DAL).
- Etiquetas: `CC1`, `CC2`, `problema`, `cambio`, `dal-A`, `dal-B`, `dal-C`, `dal-D`. Hitos (milestones): `SRR`, `PDR`, `CDR`, `TRR`, `FRR`.

### Fase 3: `drone-px4` (fork)

1. `gh repo fork PX4/PX4-Autopilot --fork-name drone-px4 --clone=false` (solo la rama por defecto). Comprueba que el tag `v1.17.0` existe en el fork; si no, súbelo desde un clon completo de upstream.
2. Clona el fork (con submódulos no hace falta para editar; sí para compilar) y crea la rama `drone` desde el tag `v1.17.0`.
3. Aplica el parche `~/drone-sim/px4_patches/drop_guard_px4_v1.17.0.patch` con `git apply --check` y luego `git apply`. Está **comprobado** que aplica limpio sobre v1.17.0 (commit `d6f12ad1c4f70ad3230afd7d86e971421e02fef4`). Afecta a 15 ficheros, entre ellos `src/modules/drop_guard/*`, `msg/DropGuardStatus.msg`, `dds_topics.yaml` y las placas `sitl` y `fmu-v6c`.
4. Divide el resultado en commits con sentido (por ejemplo: módulo `drop_guard`; mensaje y topic DDS; integración en arranque y placas), no uno solo enorme. Si no puedes dividirlo sin romper la compilación, un commit está bien.
5. Sube la rama `drone` y crea el tag anotado `v1.17.0-drone.1` (luego `v1.17.0-1.0.0`) sobre su punta. Es **un tag de trabajo, no una baseline** del plan.
6. **Desactiva GitHub Actions en el fork** (heredaría los workflows de PX4, muy pesados). `gh api -X PUT repos/<login>/drone-px4/actions/permissions -F enabled=false`.
7. `drone-px4` mantiene `main` como espejo de upstream; el trabajo va en `drone`. Añade `DRONE.md` en la rama `drone` explicando esto.

### Fase 4: contenido de `drone-ros` y `drone-sim`

**drone-ros**: copia los cuatro paquetes y `ops/`, `missions/` a la raíz. Comprueba que `colcon` sigue encontrando todo y que los tests (`drone_core` 35, `drone_mission` 29 = 64) pasan en la Fase 6. Actualiza cualquier ruta que asumiera `ros2_ws/src/...` dentro del repo.

**drone-sim**:

- Mueve los scripts según la tabla. `docker/` y lo asociado, a `legacy/`.
- Crea `drone.repos` (formato `vcs`) fijando: `drone-ros` (rama `main`, luego un commit o tag concreto cuando exista), `drone-px4` (`v1.17.0-drone.1` (luego `v1.17.0-1.0.0`)), `px4_msgs` (`release/1.17`), `Micro-XRCE-DDS-Agent` (`v2.4.3`). Este fichero es lo que representará cada baseline.
- Adapta `scripts/setup_native.sh`:
  - el paso `px4` clona **`drone-px4` en el tag/rama fijado**, no `PX4/PX4-Autopilot`;
  - el paso `patch` desaparece;
  - el paso `ws` obtiene `drone-ros` con `vcs import ros2_ws/src < drone.repos` (solo esa entrada) antes de `colcon build`;
  - se mantienen los demás pasos, la idempotencia, `clean_env` y el log en `logs/`.
- Adapta `env_native.sh` (`OPS_DIR` y `MISSIONS_DIR` pasan a apuntar a `ros2_ws/src/drone-ros/ops` y `.../missions`) y `check_native.sh`, `start_sim.sh` y `README_NATIVE.md`. `s2_scenarios.py` y `sitl_drop_guard_test.py` no deben depender de rutas del layout antiguo.
- Actualiza `.gitignore` para el layout nuevo.
- **Corrige dos fallos de `setup_native.sh` detectados en la primera ejecución** (el entorno funciona; son de aviso y de higiene):
  1. Falso aviso `AVISO: MicroXRCEAgent no arranca`. Con `set -o pipefail`, `MicroXRCEAgent -h` sale con código 1 (imprime el uso y sale), y eso hace fallar la tubería aunque `grep` encuentre `Usage`. Sustituye la línea por `{ "${DEPS}/bin/MicroXRCEAgent" -h 2>&1 || true; } | grep -q "^Usage" || warn "MicroXRCEAgent no arranca; revisa el log."` (el agente funciona: lo confirma `check_native.sh`).
  2. `pip install --target .deps/pylibs pymavlink pyulog` instaló **numpy 2.5.3** en `.deps/pylibs`, que va delante en `PYTHONPATH` y tapa al numpy 1.x del sistema, contra el que están compiladas ROS y scipy (pip avisó del conflicto con scipy 1.11.4). Cámbialo por `python3 -m pip install --quiet --upgrade --target "${DEPS}/pylibs" "numpy<2" pymavlink pyulog`. En el equipo del usuario, para arreglarlo sin repetir la instalación: `rm -rf ~/drone-sim/.deps/pylibs && ./scripts/setup_native.sh pylibs`, y después repetir `check_native.sh`.

### Fase 5: CI

Todos los runners son `ubuntu-24.04`, estándar (gratis en repos públicos).

- **`drone-ros`, `.github/workflows/ci.yml`** (en `pull_request` y `push` a `main`): instalar ROS 2 Jazzy con `ros-tooling/setup-ros`, clonar `px4_msgs` en `release/1.17` y compilarlo (cachearlo), instalar `libyaml-cpp-dev` y `nlohmann-json3-dev`, `colcon build`, `colcon test`, `colcon test-result --verbose`. Debe dar **64 tests correctos**. Comprueba que el `-Werror` con el que se compiló en el hito S2 sigue sin dar avisos; si hay flags concretos en los `CMakeLists.txt`, respétalos.
- **`drone-sim`, `.github/workflows/nightly.yml`** (`schedule` diario y `workflow_dispatch`): `scripts/setup_native.sh` completo y `scripts/check_native.sh`. Es largo (la compilación de PX4 tarda más de 30 min): usa `ccache` con `actions/cache` (la caché es de 10 GB por repo, no cachees `.deps` entero) y `timeout-minutes` razonable. Si no cabe en el runner estándar, deja el workflow solo con `workflow_dispatch` y documenta el motivo.
- **`drone-docs`**: sin CI por ahora.
- Los nombres de los jobs de CI son los que exigirá el ruleset de `main` (Fase 6).

### Fase 6: protección de `main`, subida y verificación

Orden estricto: **primero** el push inicial de cada repo a `main`, **después** el ruleset (con `main` protegida no podrías hacer el primer push).

1. Hace el commit inicial y `git push -u origin main` en `drone-ros`, `drone-sim` y `drone-docs`.
2. En `drone-ros` y `drone-sim`, crea un ruleset para la rama por defecto vía `gh api repos/<login>/<repo>/rulesets -X POST --input ruleset.json`, con: exigir pull request (0 aprobaciones), exigir el check de CI del repo, bloquear force-push y borrado, historial lineal. Sin actores de bypass. En `drone-docs` solo PR obligatoria, sin checks.
3. Verifica el ruleset: un `git push` directo a `main` debe **fallar**; el flujo por PR debe funcionar. Haz una PR de prueba trivial (por ejemplo, un cambio de README) en `drone-ros`, comprueba que el CI corre en verde y se puede fusionar, y elimínala.
4. **Verificación del resultado**: clona `drone-sim` **limpio** en `~/drone/drone-sim` (no dentro de `~/drone-sim`), y ejecuta `./scripts/setup_native.sh` (con `SKIP_PX4_DEPS=1` si el sistema ya tiene las dependencias de PX4) y luego `RUN_TESTS=1 ./scripts/check_native.sh`. Debe dar **0 fallidas y 64 tests**. Avisa al usuario de que la compilación de PX4 tarda más de 30 min. Deja `~/drone-sim` sin tocar como copia de seguridad; el usuario decidirá cuándo borrarlo.

## 6. Criterios de aceptación

- Cuatro repos públicos: `drone-ros`, `drone-px4`, `drone-sim`, `drone-docs`, con licencia, README y plantillas.
- `drone-px4`: rama `drone` con drop_guard como commits, tag `v1.17.0-drone.1` (luego `v1.17.0-1.0.0`), Actions desactivadas.
- `drone.repos` en `drone-sim` fija las cuatro dependencias con versión exacta.
- CI de `drone-ros` en verde con 64 tests. Ruleset activo y comprobado en `drone-ros` y `drone-sim`.
- Un clon limpio de `drone-sim` reproduce el entorno y pasa `check_native.sh` sin fallos.
- Ningún commit lleva el correo personal del usuario como autor.

## 7. Restricciones

- No toques nada de otros proyectos. Solo se comparte `/opt/ros/jazzy`.
- No hagas `git push --force`, no reescribas historia publicada, no borres repos ni ramas remotas sin preguntar.
- No instales nada con `sudo` que no esté ya en `setup_native.sh` sin avisar.
- Si algo no cuadra con este documento (por ejemplo, el parche no aplica, el tag no existe en el fork, el nombre del repo ya está cogido), **para y pregunta al usuario** en lugar de improvisar.
- No inventes contenido de documentación ni requisitos: DOC-xx vive en Claude Docs.

## 8. Qué necesita del usuario

- `gh auth login` si no está autenticado (interactivo).
- Confirmar el nombre de usuario u organización de GitHub donde crear los repos, si no es la cuenta autenticada.
- Decidir si el correo de mantenedor de los `package.xml` se queda como está.

## 9. Informe final

Termina con un resumen corto para el usuario: URLs de los cuatro repos, el commit y el tag de `drone-px4`, resultado de la verificación en clon limpio (tests y `check_native.sh`), lo que dejaste fuera y por qué, y una lista de decisiones que tomaste por tu cuenta. El usuario lo pegará en el chat original para actualizar la documentación del proyecto.

## 10. Hechos comprobados que te ahorran tiempo

- El parche `drop_guard_px4_v1.17.0.patch` aplica limpio sobre el tag `v1.17.0` de PX4 (probado con `git apply --check`); ese tag es el commit `d6f12ad1c4f70ad3230afd7d86e971421e02fef4`.
- SIH arranca con `make px4_sitl sihsim_quadx`, y `px4-rc.sihsim` también lee `PX4_HOME_LAT/LON/ALT`; `SIH_LOC_*` son float en grados. El simulador por defecto en el portátil es SIH; `SIM=gz` usa Gazebo.
- `px4_msgs release/1.17` **no** trae `DropGuardStatus.msg` (lo añade el parche): en el hito S3 habrá que generar `px4_msgs` desde el `msg/` del fork. No lo resuelvas ahora, pero déjalo anotado en el README de `drone-sim`.
- `setup_native.sh` compila PX4 y el agente en un entorno limpio (sin ROS ni rutas de otros proyectos), porque mezclar los de ROS con los del agente XRCE causa conflictos de librerías. Conserva `clean_env` y el wrapper `.deps/bin/MicroXRCEAgent`.
- Los tópicos de PX4 1.17 llevan sufijo `_vN` cuando el mensaje está versionado (`vehicle_status_v1`, `vehicle_global_position_v1`...). `check_native.sh` ya resuelve el nombre real.
- Tests esperados: `drone_core` 35 y `drone_mission` 29 (64 en total).
