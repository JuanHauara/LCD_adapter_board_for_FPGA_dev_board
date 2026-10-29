# Driver de backlight LCD con TPS61165

Driver de corriente constante para el backlight del LCD **SH101JGI40-01017W-03**. Alimentación: **5 V ±5 %**. El trimpot fija la corriente máxima y el PWM en el pin 2 LCD_BACKLIGHT_PWM regula el brillo.

## Componentes con requisitos especiales

| Componente | Requisitos |
|---|---|
| L1 | Blindado; Isat ≥2 A; Irms ≥1 A; DCR ≤0.15 Ω; inductancia efectiva ≥13.2 µH en funcionamiento. |
| D1 | Schottky de 60 V / 2 A. |
| C9, C10 | 50 V, X7R, 1206. Capacidad efectiva conjunta ≥4.7 µF a 12 V. |

**C9 y C10 deben ser de 50 V:** con LED desconectado, la protección actúa alrededor de 38 V.

## Ajuste de corriente con trimpot

```text
VLED− ──┬───────────────────── R1 = 2.2 Ω ─────────────────────── GND
        │
        ├──── R7 = 120 Ω ────[ TRIM1 = 1 kΩ ]──── R9 = 330 Ω ──── GND
        │                           ↑ 
        │                           │ cursor
        └──── R5 = 5.6 kΩ ──────────┴──────────────────────────── FB (pin 6)
```

- **R7 y R9:** Fijan el límite de corriente mínimo y máximo ajustable con el potenciómetro.
- **R5 (5.6 kΩ):** Mantiene la realimentación si el cursor pierde contacto. En ese caso, la corriente disminuye y el backlight sigue encendido.

| Posición del cursor | Corriente nominal con PWM al 100 % |
|---|---:|
| Extremo próximo a R7 | **99.1 mA** |
| Extremo próximo a R9 | **348.6 mA** |

Para setear la corriente de regulación, poner PWM al 100 % conectando el jumper de P3 entre los pines 2 y 3 para llevar el pin 2 (CTRL) a 3,3 V, comenzar con el cursor próximo a R7 y medir la corriente. El ajuste no es lineal.
Ajustar sin superar la corriente admisible por la pantalla.

## Control de brillo

- **LCD_BACKLIGHT_PWM → CTRL (pin 2): 20 kHz, 0 a 3.3 V**.
- Reducir el ciclo útil reduce la corriente y el brillo.
- CTRL bajo durante más de **2.5 ms** apaga el driver. Mantenerlo bajo durante el arranque.

## Validación del prototipo

Pendiente comprobar arranque, corriente, rizado y temperatura. En la PCB, acortar el lazo de conmutación y tomar VLED− junto al terminal superior de R1, separado de las pistas de potencia.

**La etapa no garantiza 350 mA en todas las condiciones.** Con entrada de 4.75 V, backlight de 11 V y eficiencia supuesta del 80 %, el pico calculado ronda 1.2 A, frente al límite mínimo de 0.96 A del TPS61165.

Referencia: [datasheet TPS61165](LCD_BACKLIGHT_LED_driver_TPS61165DBVR.pdf), página 5 para límites eléctricos y página 17 para la referencia de FB.


## Voltages de alimentación con TPS65150

**Realizar todos los ajustes con el LCD desconectado. No conectar ni retirar el FPC con alimentación.**

### Tensiones del panel actual

Medir respecto de GND, preferentemente en el FPC.

| Alimentación | Pin FPC | Objetivo | Límites del panel |
|---|---|---:|---:|
| DVDD digital | 7 | 3,3 V | 3,0–3,6 V |
| AVDD / VS | 43 | 9,6 V | 9,5–9,7 V |
| VGL | 42 | −6,0 V | −6,4 a −5,6 V |
| VGH, salida del MIC5235 | 41 | 18,0 V | 17,6–18,4 V |
| VCOM | 6 y 46 | 4,2 V | 4,0–4,4 V |

### Ajustes disponibles

Comprobar que los componentes montados coincidan con estas redes. Los rangos son nominales, sin tolerancias; **no representan tensiones admisibles para el panel actual**.

| Salida | Red de realimentación | Rango nominal completo | Preajuste del trimpot |
|---|---|---:|---:|
| VS | Superior 56 kΩ; inferior 4,7 kΩ + 390 Ω + trimpot 5 kΩ | 7,51–13,75 V; intervalo previsto: 8–13 V | ≈2,50 kΩ para 9,6 V |
| VGL | VGL–FBN: 390 kΩ; FBN–REF: 56 kΩ + trimpot 50 kΩ | −8,45 a −4,46 V; intervalo previsto: −8 a −5 V | ≈22,85 kΩ para −6 V |
| VCOM | VS–IN: 56 kΩ; IN–GND: 39 kΩ + 2,2 kΩ + trimpot 5 kΩ | 4,069–4,340 V con VS = 9,6 V | ≈2,36 kΩ para 4,2 V |

Al aumentar la resistencia del trimpot, VS disminuye, VGL se hace menos negativa y VCOM aumenta. El sentido horario depende del conexionado. Ajustar por la tensión medida.

VGH es fija: ≈17,98 V con 120 kΩ + 15 kΩ entre OUT y ADJ del MIC5235, y 10 kΩ entre ADJ y GND. Esta versión no tiene trimpot de VGH.

### Procedimiento de ajuste

1. Sin alimentación ni LCD, comprobar orientación del FPC, cortocircuitos y divisores. Mantener RESET bajo y retroiluminación apagada.
2. Alimentar con limitación de corriente y comprobar las ramas conmutadas de 3,3 V y 5 V.
3. Ajustar **VS a 9,60 V primero**.
4. Comprobar la bomba positiva: ≈22,4 V con divisor 820 kΩ / 47 kΩ. Se mide en C4 del circuito del datasheet, que alimenta al MIC5235; no es la salida de 18 V.
5. Ajustar **VGL a −6,00 V**, verificar **VGH ≈18,00 V** y ajustar **VCOM a 4,20 V al final**.
6. Verificar arranque, rizado y sobreimpulsos con osciloscopio, además de regulación con carga de prueba. Apagar, esperar la descarga de las salidas, conectar el LCD y comprobar nuevamente bajo carga.

### Precauciones del circuito

- **No barrer todo el rango de VS sin revisar previamente la bomba positiva.** Cambiar VS afecta su funcionamiento. La entrada y el enable del MIC5235 no deben superar 24 V, incluidos los picos.
- El rango de VGL depende también de VS y de la carga: no asumir que alcance −8 V con VS = 8 V. Su divisor termina en REF, no en GND.
- Reajustar VCOM después de cualquier cambio de VS.
- No confundir C15 = 270 pF, paralelo a los 56 kΩ de VS, con COMP, pin 22 = 2,2 nF. Estos índices corresponden al datasheet.
- El pin VGH del TPS se utiliza para habilitar el MIC5235. Los 18 V del FPC proceden de OUT del MIC5235.
- Si el TPS queda en protección enclavada, corregir la causa y descargar su entrada hasta 0 V antes de reintentar.

### Alimentación y RESET

El switch y los MOSFETs conmutan 3,3 V y 5 V, pero **no garantizan por sí solos su secuencia**.

Orden propuesto: DVDD estable → alimentación del TPS → fuentes analógicas estables → liberar RESET. Garantizar DVDD primero requiere supervisión o Power Good.

DLY1 y DLY2 de 22 nF proporcionan ≈5,3 ms nominales por etapa. VCOM y la habilitación del MIC5235 parten del mismo evento; no queda garantizado que VGH llegue a 18 V antes que VCOM.

Mantener RESET, FPC44, bajo durante el arranque, incluso mientras el MCU/FPGA esté sin configurar. Liberarlo con las fuentes estables y después de más de 1 ms desde DVDD estable.

Antes de apagar, desactivar retroiluminación y señales y afirmar RESET. **No aplicar señales digitales con DVDD apagado.** El switch tampoco garantiza el orden de descarga.

El datasheet del panel no proporciona una secuencia completa de alimentación: validar las rampas reales de encendido y apagado.
