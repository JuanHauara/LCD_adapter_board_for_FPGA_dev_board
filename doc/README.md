# Driver de backlight LCD con TPS61165

Driver de corriente constante para el backlight del LCD **SH101JGI40-01017W-03**. Alimentación: **5 V ±5 %**. El trimpot fija la corriente máxima y el PWM regula el brillo.

## Componentes

| Elemento | Valor o modelo | Requisitos |
|---|---|---|
| U1 | **TPS61165DBVR** | SOT-23 de 6 pines. |
| L1 | **22 µH**, blindado | Isat ≥2 A; Irms ≥1 A; DCR ≤0.15 Ω; inductancia efectiva ≥13.2 µH en funcionamiento. |
| D1 | **Vishay SS26SHE3_B/H** | Schottky 60 V / 2 A, SMA, LCSC **C511359**. |
| C1A, C1B | **2 × 10 µF / 16 V**, X5R, 0805 | En paralelo entre +5 V y GND. |
| C4 | **100 nF / 16 V**, X7R, 0805 | Desacoplo junto a VIN y GND del integrado. |
| C2A, C2B | **2 × 4.7 µF / 50 V**, X7R, 1206 | Entre VLED+ y GND. Capacidad efectiva conjunta ≥4.7 µF a 12 V. |
| C3 | **220 nF / 16 V**, X7R, 0805 | Entre COMP y GND; compensación del regulador. |
| R1 — sensado | **2.2 Ω**, 1 %, **1 W**, 2512 | Entre VLED− y GND. |
| R2 — superior | **120 Ω**, 1 %, 0.125 W | Entre VLED− y el extremo superior de VR1. |
| VR1 | **1 kΩ**, multivuelta | Trimpot tipo 3296; cursor directamente a FB, pin 6. |
| R3 — inferior | **330 Ω**, 1 %, 0.125 W | Entre el extremo inferior de VR1 y GND. |
| R4 — respaldo | **5.6 kΩ**, 1 %, 0.125 W | Entre FB y VLED−. |
| RPD | **100 kΩ** | CTRL a GND: mantiene apagado el driver si la entrada queda sin controlar. |

**C2A y C2B deben ser de 50 V:** con LED desconectado, la protección actúa alrededor de 38 V.

## Ajuste de corriente con trimpot

```text
VLED− ──┬── R1 = 2.2 Ω ─────────────────────────── GND
        │
        ├── R2 = 120 Ω ──[ VR1 = 1 kΩ ]── R3 = 330 Ω ── GND
        │                       ↑ cursor
        │                       │
        └── R4 = 5.6 kΩ ────────┴── FB (pin 6)
```

**VLED− y FB son nodos distintos.**

- **R2 y R3:** fijan conjuntamente los límites de corriente ajustables con el potenciómetro: mínimo con el cursor hacia R2 y máximo hacia R3.
- **R4 (5.6 kΩ):** mantiene la realimentación si el cursor pierde contacto. En ese caso, la corriente disminuye y el backlight sigue encendido.

| Posición del cursor | Corriente nominal, PWM al 100 % |
|---|---:|
| Extremo próximo a R2 | **99.1 mA** |
| Extremo próximo a R3 | **348.6 mA** |

Para ajustar, poner PWM al 100 %, comenzar con el cursor próximo a R2 y medir la corriente. El ajuste no es lineal.

Las tolerancias pueden llevar el extremo alto por encima de **350 mA**. Ajustar sin superar la corriente admisible por la pantalla.

## Control de brillo

- **LCD_BACKLIGHT_PWM → CTRL (pin 2): 20 kHz, 0 a 3.3 V**.
- Reducir el ciclo útil reduce la corriente y el brillo.
- CTRL bajo durante más de **2.5 ms** apaga el driver. Mantenerlo bajo durante el arranque.

## Validación del prototipo

Pendiente comprobar arranque, corriente, rizado y temperatura. En la PCB, acortar el lazo de conmutación y tomar VLED− junto al terminal superior de R1, separado de las pistas de potencia.

**La etapa no garantiza 350 mA en todas las condiciones.** Con entrada de 4.75 V, backlight de 11 V y eficiencia supuesta del 80 %, el pico calculado ronda 1.2 A, frente al límite mínimo de 0.96 A del TPS61165.

Referencia: [datasheet TPS61165](LCD_BACKLIGHT_LED_driver_TPS61165DBVR.pdf), página 5 para límites eléctricos y página 17 para la referencia de FB.

