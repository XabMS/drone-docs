# DOC-08 Estudio de hardware (borrador v0.1)

Sep 25, 2026 · @Xabier

## 1. Criterios de selección

Cada componente se evalúa contra los requisitos y principios ya aprobados; el precio decide solo entre opciones que los cumplen.

| Criterio | Origen |
| --- | --- |
| Soporte oficial en PX4 (placa y drivers) | ADR-001, Planes §5 (versiones fijadas) |
| IMU redundante en la FMU | FC-01, FC-02 |
| FTS sin ningún componente compartido con la cadena principal | ADR-002, CCA |
| GNSS del FTS de otro fabricante o modelo que GNSS1 | CMA (DOC-05 §5) |
| Enlace C2 por LTE en el companion | ADR-003 |
| Carga de 1 kg con MTOM inferior a 4 kg | AR-001, AR-002 |
| Presupuesto total inferior a 1.000 € | D10 |

## 2. Comparativa por componente

Precios de lista en USD de la web del fabricante a septiembre de 2026; en euros, con IVA y envío, serán parecidos o algo mayores.

**Controlador de vuelo (FMU)**

| Opción | Precio | A favor | En contra | Veredicto |
| --- | --- | --- | --- | --- |
| [Pixhawk 6C Mini](https://holybro.com/products/pixhawk-6c-mini) | desde $131 ($150 con módulo de potencia) | Mismo procesador STM32H743 e IMU redundantes que la 6C; compacta | Menos puertos | **Recomendada** |
| [Pixhawk 6C](https://holybro.com/products/pixhawk-6c) | $199 ($218 con PM02) | IMU redundantes, PX4 preinstalado, más puertos | +$70 respecto a la Mini | Alternativa si faltan puertos |
| [Pixhawk 6X](https://holybro.com/products/pixhawk-6x) | $269 solo módulo; $389–423 en kit | Ethernet para el companion, IMU triple | Rompe el presupuesto | Descartada en fase 1 |

**Ordenador de a bordo (companion)**

| Opción | Precio | A favor | En contra | Veredicto |
| --- | --- | --- | --- | --- |
| [Raspberry Pi 5, 4 GB](https://www.raspberrypi.com/news/more-memory-driven-price-rises/) | $75–85 (precios al alza en 2026 por la memoria) | ROS 2 Jazzy en Ubuntu 24.04, gran comunidad, UART y USB para LTE | Necesita BEC de 5 V / 5 A y refrigeración | **Recomendado** |
| Raspberry Pi 5, 8 GB | $95–125 | Más margen para percepción futura | No hace falta sin cámara (D4) | Descartado por ahora |
| Jetson Orin Nano | Muy por encima del presupuesto | GPU para visión | Precio y consumo | Para cuando haya cámara |

**Airframe y propulsión**

| Opción | Precio | A favor | En contra | Veredicto |
| --- | --- | --- | --- | --- |
| [Holybro X500 V2 ARF](https://holybro.com/products/x500-v2-kits) | $329 | Frame de 610 g, motores 2216 920 KV, ESC y hélices montados; carga de 1,5 kg al 70 % de gas; bien soportado en PX4 | Autonomía corta con carga (ver §4) | **Recomendado para fase 1** |
| X500 V2 solo frame | $135 | Más barato | Motores y ESC aparte; más trabajo y riesgo | Solo si ya tienes propulsión |

**Enlace C2 (LTE)**

| Opción | Precio | Veredicto |
| --- | --- | --- |
| Dongle USB [Quectel EC25-EU](https://www.amazon.com/Quectel-EC25-EU-Module-150Mbps-Thailand/dp/B0FKMF93XS) (bandas europeas) | \~$40–60 | **Recomendado**: módem industrial muy usado con Linux |
| Tarjeta mini PCIe EC25 + placa adaptadora | \~$40–70 | Más robusto mecánicamente, más montaje |

**GNSS**

| Uso | Opción | Precio |
| --- | --- | --- |
| GNSS1 (FMU) | [Holybro M10 GPS](https://holybro.com/products/m10-gps) con brújula IST8310 | $44 |
| GNSS del FTS | Módulo M10 de otro fabricante (p. ej. Matek o Beitian) | \~$20–35 (estimado) |

Nota: GNSS1 y GNSS del FTS llevarían el mismo chip u-blox M10 aunque sean de fabricantes distintos. Es un modo común parcial que la CMA tiene que justificar o eliminar (por ejemplo, con un chip de otro fabricante en el FTS).

**FTS (componentes, precios estimados)**

| Elemento | Opción | Precio estimado |
| --- | --- | --- |
| MCU | Placa RP2040 o STM32 pequeña | \~$5–15 |
| Receptor | ExpressLRS en banda distinta de la emisora de seguridad (p. ej. 868 MHz si la emisora es 2,4 GHz) | \~$15–25 |
| Batería propia | LiPo 1S–2S pequeña | \~$10 |
| Corte de potencia | Interruptor MOSFET de alta corriente (≥ 80 A) | \~$25–40 |

## 3. Lista de materiales recomendada

El kit completo sale por unos $1.070, algo por encima del límite. Si ya tienes emisora RC y cargador, baja a unos $930 y entra en el presupuesto.

| Elemento | Selección | Precio aprox. (USD) |
| --- | --- | --- |
| Airframe + propulsión | Holybro X500 V2 ARF | 329 |
| FMU | Pixhawk 6C Mini + módulo de potencia | 150 |
| GNSS1 | Holybro M10 GPS | 44 |
| Companion | Raspberry Pi 5, 4 GB | 85 |
| Accesorios del companion | microSD, disipador, BEC 5 V / 5 A | 30 |
| C2 | Dongle LTE Quectel EC25-EU | 50 |
| Mando de seguridad | Emisora + receptor ExpressLRS 2,4 GHz | 90 |
| FTS | MCU, receptor 868 MHz, GNSS, batería, corte MOSFET | 105 |
| Batería de vuelo | LiPo 4S 5000 mAh | 55 |
| Cargador | Cargador LiPo balanceado | 50 |
| Carga útil | Servo de suelta + soporte impreso + paracaídas del paquete | 40 |
| Varios | Cables, conectores, amortiguadores, impresiones 3D | 40 |
| **Total** |  | **\~1.070** |
| **Total sin emisora ni cargador** |  | **\~930** |

Coste recurrente aparte: tarjeta SIM de datos para el LTE.

**Ajuste al presupuesto:** sin emisora ni cargador propios, el total (unos $1.070) supera el límite en un 5–10 %. Hay tres ahorros posibles sin tocar la seguridad, a decidir al comprar: Raspberry Pi 5 de 2 GB, suficiente sin cámara (unos $20 menos); cargador básico (unos $20 menos); y emisora ExpressLRS de gama de entrada (unos $20 menos). Con los tres, el total queda alrededor de $1.010.

**Compra por lotes (reparte el gasto y sigue el plan de verificación):**

| Lote | Cuándo | Contenido | Aprox. (USD) |
| --- | --- | --- | --- |
| 1. Banco HIL | Tras la TRR de simulación | FMU, GNSS1, companion y accesorios, LTE | \~360 |
| 2. Plataforma | Tras superar HIL | X500 ARF, batería, cargador, emisora | \~525 |
| 3. Seguridad y carga | Antes de la FRR | FTS completo, mecanismo de suelta, varios | \~185 |

## 4. Presupuesto de masa y energía

Con este hardware el dron pesa unos 3,0 kg con carga y cumple AR-002, pero su radio útil con 1 kg ronda 1,5–2 km, no los 5 km de AR-003. Es la conclusión más importante del estudio.

**Masa estimada**

| Bloque | Masa (g) |
| --- | --- |
| Airframe ARF (frame 610 g + motores, ESC y hélices) | \~990 |
| Batería 4S 5000 mAh | \~520 |
| Aviónica (FMU, módulo de potencia, GNSS, receptor RC) | \~95 |
| Companion + BEC + LTE | \~120 |
| FTS completo | \~120 |
| Mecanismo de suelta | \~60 |
| Cableado y varios | \~100 |
| **Vacío en orden de vuelo** | **\~2.000** |
| Carga útil | 1.000 |
| **Total al despegue** | **\~3.000 (límite 4.000)** |

**Autonomía (estimación de primer orden)**

Holybro da unos 18 min en estacionario con batería de 5000 mAh y sin carga (masa estimada \~1,55 kg). Escalando con la teoría de cantidad de movimiento (tiempo ∝ masa^-1,5):

| Configuración | Masa | Estacionario estimado |
| --- | --- | --- |
| Especificación Holybro | \~1,55 kg | 18 min |
| Nuestro dron sin carga (regreso) | \~2,0 kg | \~12 min |
| Nuestro dron con 1 kg (ida) | \~3,0 kg | \~7 min |

Una misión de 5 km necesita \~7 min de ida cargado, \~7 min de vuelta y \~2 min de despegue, descenso y suelta. Solo la ida ya consume toda la batería. Con un 20 % de reserva, el radio alcanzable sale de unos 1,5 km, quizá 2 km porque el vuelo en avance a velocidad moderada es algo más eficiente que el estacionario. Hay que confirmarlo con ensayos (DOC-11).

**Opciones**

| Opción | Efecto |
| --- | --- |
| A. Radio reducido para el prototipo físico (\~1,5 km); 5 km se mantiene en simulación | Sin coste; AR-003 se divide en objetivo del sistema y límite del prototipo |
| B. Bajar la carga a 0,5 kg | Algo más de radio (\~2 km estimado), pero cambia la misión |
| C. Otro airframe más adelante (hélices mayores, 6S, batería Li-ion o VTOL de ala fija) | Cumple AR-003; fuera del presupuesto de fase 1 |

Decidido (ADR-009): opción A. Se valida todo el sistema a escala con el X500 y la opción C queda para una segunda plataforma.

## 5. Consecuencias y preguntas abiertas

**ADR-008 queda resuelta por el hardware:** la Pixhawk 6C Mini no tiene Ethernet, así que FMU y companion se conectan por UART: un puerto TELEM para uXRCE-DDS a alta velocidad (921.600 baudios o más) y otro para MAVLink hacia el LTE. Si en fase 2 el ancho de banda no basta, la migración natural es la Pixhawk 6X con Ethernet.

**ADR-009 (aceptada, 25/09/2026):** separar AR-003 en un objetivo de sistema (5 km, validado en simulación) y un límite del prototipo físico (\~1,5 km, a confirmar en ensayos).

**Preguntas abiertas**

- [ ] Emisora RC y cargador: no se tienen, entran en la compra (25/09/2026).
- [ ] Opción A del §4 aceptada (ADR-009).
- [ ] ¿Quieres que busque para el FTS un GNSS con chip de otro fabricante (no u-blox) para eliminar el modo común?

**Fuentes**

- [Holybro Pixhawk 6C Mini](https://holybro.com/products/pixhawk-6c-mini)
- [Holybro Pixhawk 6C](https://holybro.com/products/pixhawk-6c)
- [Holybro Pixhawk 6X](https://holybro.com/products/pixhawk-6x)
- [Holybro X500 V2 kits](https://holybro.com/products/x500-v2-kits)
- [Holybro M10 GPS](https://holybro.com/products/m10-gps)
- [Raspberry Pi: subidas de precio por la memoria (feb. 2026)](https://www.raspberrypi.com/news/more-memory-driven-price-rises/)
- [Tom's Hardware: precios de Raspberry Pi 5 (feb. 2026)](https://www.tomshardware.com/raspberry-pi/raspberry-pi-5-price-increases-drastically-as-ai-shortage-bites-16gb-version-now-usd205-second-price-increase-in-three-months-over-70-percent-more-expensive-than-original-msrp)
- [Quectel EC25-EU USB dongle (Amazon)](https://www.amazon.com/Quectel-EC25-EU-Module-150Mbps-Thailand/dp/B0FKMF93XS)

Los precios de los componentes del FTS, la emisora, la batería, el cargador y los varios son estimaciones sin fuente concreta.
