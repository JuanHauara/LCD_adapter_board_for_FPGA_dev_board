# Driver de backlight LCD con TPS61165

Circuito de corriente constante para el backlight del LCD **SH101JGI40-01017W-03**. Entrada de **5 V**, control PWM y cinco niveles de corriente.

**Estado:** calculado para prototipo; pendiente de comprobar arranque, rizado y temperatura. Las fuentes con TPS65150 quedan fuera de este documento.

## Condiciones de diseño

- Entrada: **5 V ±5 %**. Reservar aproximadamente **1 A**.
- Backlight: hasta **11 V** entre VLED+ y VLED−; la salida suma unos 0,2 V de sensado.
- Corriente: **90,9 a 261,1 mA nominales**, con PWM al 100 %.

## Componentes

Referencias funcionales; adaptar los designadores al esquemático.

| Elemento | Valor o modelo | Requisitos |
|---|---|---|
| U1 | **TPS61165DBVR** | SOT-23 de 6 pines. |
| L1 | **22 µH**, blindado | Isat ≥1,5 A, preferiblemente 2 A; Irms ≥1 A; DCR ≤0,15 Ω; inductancia efectiva ≥13,2 µH en funcionamiento. |
| D1 | **Vishay SS26SHE3_B/H** | Schottky 60 V / 2 A, SMA, LCSC **C511359**. |
| C1A, C1B | **2 × 10 µF / 16 V**, X5R, 0805 | En paralelo entre +5 V y GND. |
| C4 | **100 nF / 16 V**, X7R, 0805 | Desacoplo junto a VIN y GND del integrado. |
| C2A, C2B | **2 × 4,7 µF / 50 V**, X7R, 1206 | Entre VLED+ y GND. Capacidad efectiva conjunta ≥4,7 µF a 11,2 V, incluyendo tolerancia y temperatura. |
| C3 | **220 nF / 16 V**, X7R, 0805 | Entre COMP y GND; compensación del regulador. |
| R0 | **1.8 Ω**, 0805, 0,125 W | Siempre conectada entre FB y GND. |
| R1 a R4 | **5 × 6.8 Ω**, 0805, 0,125 W | Cada resistencia en serie con su jumper; ramas en paralelo con R0. |
| RPD | **100 kΩ** | CTRL a GND: apagado mientras el MCU no controla la señal. |

**C2A y C2B debe ser de 50 V:** con LED desconectado, la protección actúa alrededor de 38 V.

## Selección de corriente

```text
VLED− / FB ──┬── R0 = 1.8 Ω ───────── GND
             ├── R1 = 6.8 Ω ── JP1 ── GND
             ├── R2 = 6.8 Ω ── JP2 ── GND
             ├── R3 = 6.8 Ω ── JP3 ── GND
             ├── R4 = 6.8 Ω ── JP4 ── GND
             ├── R5 = 6.8 Ω ── JP5 ── GND
             └── R6 = 6.8 Ω ── JP6 ── GND
```

Fórmula: **página 17, “Current Program”, ecuación (7)** del PDF local, revisión SLVS790C:

`I_LED = V_FB / R_SET`

Con PWM al 100 %, `V_FB = 0.2 V`:

`I_LED [A] = 0.2 × (1/1.8 + n/6.8)`

`n` es la cantidad de jumpers cerrados; cualquier combinación es válida.

| Jumpers cerrados | Corriente nominal |
|---:|---:|
| 0 | **111mA** |
| 1 | **140.5mA** |
| 2 | **169.9mA** |
| 3 | **199.3mA** |
| 4 | **228.7mA** |
| 5 | **258.2mA** |
| 6 | **287.6mA** |

R0 disipa **22.2mW** y cada rama **5.9mW** nominales. **Cambiar jumpers sin alimentación.** Sin jumpers, el driver habilitado entrega 111mA; no queda apagado.

## Control de brillo

- Señal prevista: **PA8 / J2_20 de la DevEBox → CTRL**.
- PWM recomendado para este diseño: **20 kHz**, niveles de **0 a 3.3 V**.
- PWM al 100 %: corriente de la tabla. Reducir el ciclo útil reduce corriente y brillo.
- CTRL bajo durante más de **2.5 ms** apaga el driver. Mantenerlo bajo al arrancar el MCU.

