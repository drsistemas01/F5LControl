# Especificación ALMRT — ESP32-C3 Super Mini



## 1. Alcance

El ALMRT (*Altitude Limiter / Motor Run Timer*) es un dispositivo intercalado entre el receptor RC y el ESC de un planeador eléctrico. Limita el lanzamiento por tiempo de motor y por altura, registra los datos del vuelo y permite su verificación posterior.

Esta especificación es autosuficiente. Define qué debe hacer el dispositivo y sobre qué plataforma. No requiere código ni documentos de referencia: el algoritmo del número de serie (§5.1) incluye constantes, convenciones aritméticas y vectores de prueba verificados.

**Plataforma:** ESP32-C3 Super Mini, ESP-IDF nativo, driver propio del sensor.

---

## 2. Cumplimiento normativo

Categoría de referencia: **F5L**. Los parámetros son configurables en compilación para otras categorías.

### 2.1. De SC4_Vol_EDIC_26 Sección 7 — F5L Time & Height Limiter

| Requisito | Valor | Punto |
|---|---|---|
| Corte de motor | 30 s **o** 90 m, lo que ocurra primero | 2.3 |
| Referencia de altura cero | Máx. 2 s tras aplicar alimentación; sin responder a señal de motor durante ese lapso | 2.5 b/e |
| Salida antes del arranque | ≤ 0,9 ms | 2.5 d |
| Umbral de motor-on | 1,2 ms; histéresis permitida | 2.6.1 / 2.6.2 |
| Tasa de muestreo | ≥ 10 muestras/s | 2.3 |
| Precisión de tiempo | ≤ 0,1 s | 2.3 |
| Precisión de altura | ±2,5 m (2.3.1); "mejor que 2 m" (2.1.1 d) | — |
| Error de cómputo ISA | ±0,5 m | 2.4.5 |
| Rango de presión válido | 750 a 1050 hPa | 2.1.1 d.1 |
| Rango de temperatura | 0 a +50 °C | 2.3.1 |
| Rearranque | Prohibido tras el corte definitivo | 2.6.1 c |
| Display | **Opcional** en F5L | 2.1.2 e |
| Indicación de armado | Claramente visible para el competidor al alimentar | 2.5 a |
| Filtrado | Rechazo de transitorios permitido (2.1.1 c); **suavizado/promediado prohibido** (2.4.6) | — |
| Oversampling del sensor | Permitido | 2.4.7 |
| Alimentación | Desde la salida del receptor hacia el ESC | 2.1.2 c |
| Conectores | JR/Futaba, a prueba de desalineación | 2.1.2 |
| Instalación | ESC siempre en serie con el ALMRT | — |
| Blindaje del sensor | Protegido de luz fuerte y de presión ajena a la altura; venteo adecuado | 2.4.8.2 / 3.1.4 |

**Interpretación de la contradicción sobre filtrado.** El punto 2.1.1 c) exige rechazo de transitorios; el 2.4.6 prohíbe filtrado o suavizado. No son incompatibles: describen operaciones distintas.

- **Rechazo** — descartar una muestra físicamente imposible. La muestra del pico es válida, no se toca. **Permitido.**
- **Suavizado** — promediar muestras vecinas. Altera la muestra del pico. **Prohibido.**

Criterio de corte (2.3.4): ningún procesamiento puede modificar la altura máxima tal como la detectaría la muestra individual que representa el pico.

*Nota: 2.1.1 c) menciona "staging" como ejemplo de evento dinámico, terminología de cohetería — es arrastre de la Sección 2 (modelos espaciales). La doble cifra de precisión (2 m / 2,5 m) tiene el mismo origen. Se diseña contra los 2 m.*

### 2.2. De SC4_VOL_F5_ELECTRIC_26 punto 5.5.12.7

Requisitos no cubiertos por la Sección 7 del EDIC:

- Sin telemetría durante el vuelo de competencia
- Sin cambio de valores de ajuste desde el transmisor
- **Almacenamiento del último vuelo: punto de encendido y de apagado del motor, altura y tiempo**
- El dispositivo debe poder verificarse después del vuelo

5.5.12.8 b): **reset manual**; por transmisor está prohibido. En este diseño el reset es el ciclo de alimentación — quitar y reconectar la ficha, presenciado por el cronometrista.

### 2.3. Contradicción normativa a resolver

El rearranque está **habilitado por defecto** por reglamento GVRA, pero EDIC 2.6.1 c) lo prohíbe.

**Resolución:** parámetro de compilación. Valor por defecto *permitido*. Para presentación a homologación FAI debe compilarse en falso.

---

## 3. Configuración

Todos los parámetros son constantes de compilación. Cualquier cambio requiere recompilar y genera una nueva versión de firmware. **No existe mecanismo de configuración en campo.**

### 3.1. Reglamentarios

| Parámetro | Valor | Justificación |
|---|---|---|
| Tiempo máximo de motor | 30 000 ms | Reglamento F5L |
| Altura máxima de motor | 90 m | Reglamento F5L |
| Duración de cuarentena | 10 000 ms | Práctica de la categoría (ver §11.5) |
| Rearranque | Permitido | Reglamento GVRA |

### 3.2. Acelerador

| Parámetro | Valor | Descripción |
|---|---|---|
| Umbral de arranque | 1200 µs | Apagado → encendido |
| Umbral de apagado | 1190 µs | Encendido → apagado (10 µs de histéresis) |
| Umbral inicial | 1150 µs | Límite superior del acelerador inicial válido en el armado |
| Señal de override | 900 µs | Salida con motor apagado (máximo FAI) |
| Rango mínimo de pulso | 800 µs | — |
| Rango máximo de pulso | 2200 µs | — |
| Rango mínimo de frame | 13 500 µs | Cubre receptores Futaba S-FHSS y modos legacy (~14 ms) con margen |
| Rango máximo de frame | 25 000 µs | Límite superior del rango estándar RC |
| Frescura de acelerador | 300 ms | Límite ante pérdida de señal, no latencia esperada |

**Los tres umbrales no son intercambiables.** Cada uno se usa únicamente en la etapa que indica esta especificación.

### 3.3. Presión

| Parámetro | Valor | Descripción |
|---|---|---|
| Estabilización del sensor | 200 ms | Descarte inicial de lecturas |
| Duración de obtención de presión de piso | 1 000 ms | Total, incluida la estabilización |
| Cantidad mínima de muestras de piso | 10 | Si no se alcanza → bloqueo terminal |
| Intervalo mínimo entre muestras de piso | 40 ms | **Recalcular para C3 — ver §14.2** |
| Frescura de presión | 100 ms | Garantiza ≥10 muestras/s del EDIC |
| Timeout de validez de presión de piso | 300 000 ms | Desde la medición hasta el arranque. Si expira → bloqueo terminal |
| Rango válido de presión | 75 000 a 105 000 Pa | EDIC 2.1.1 d.1 |

### 3.4. Plataforma

| Parámetro | Valor |
|---|---|
| Velocidad del bus I2C | 50 kHz (50 % del estándar, por EMI del brushless) |
| Timeout de transacción I2C (`xfer_timeout_ms`) | 10 ms — **recalcular, ver §14.2** |
| Resolución RMT | 1 MHz (1 cuenta = 1 µs) |
| `signal_range_min_ns` | 2 000 ns (tope de hardware ≈ 3,2 µs) |
| `signal_range_max_ns` | 5 000 000 ns |
| Frecuencia LEDC | 50 Hz |
| Resolución LEDC | 14 bits (máximo del C3) |
| Oversampling de presión | ×8 |
| Oversampling de temperatura | ×8 |
| Filtro IIR del sensor | **Desactivado** (es suavizado — EDIC 2.4.6) |
| Modo del sensor | Normal, standby mínimo |

---

## 4. Hardware

### 4.1. Componentes

| Cant. | Componente | Función |
|---|---|---|
| 1 | ESP32-C3 Super Mini | Procesador, USB-C nativo, LED en GPIO8 |
| 1 | Módulo BMP280 3,3 V, 6 pines | Sensor (con pull-ups I2C y capacitor a bordo) |
| 1 | Resistencia 10 kΩ | Serie, entrada del receptor |
| 1 | Diodo Schottky BAT85 o BAT43 | Recorte de entrada a 3V3 |
| 1 | MOSFET N 2N7000 o BS170 (TO-92) | Salida al ESC |
| 1 | Resistencia 1 kΩ | Pull-up de drain al riel del BEC |
| 1 | Resistencia 10 kΩ | Pull-up de gate a 3V3 |
| 1 | Diodo Schottky 1N5817 o 1N5819 | Serie en alimentación |
| 1 | Capacitor electrolítico 470 µF / 10 V | Después del diodo serie |
| 1 | Capacitor cerámico 100 nF | En paralelo con el electrolítico |
| 1 | Buzzer activo | Indicación de rearranque (opcional) |
| 2 | Cables JR/Futaba (prolongación partida) | Entrada y salida |

### 4.2. Mapa de pines

| Función | Pin | Motivo |
|---|---|---|
| Entrada de acelerador | GPIO3 | Libre de rol de arranque. Crítico: la señal puede estar presente al energizar |
| Salida al ESC | GPIO10 | Libre de rol |
| SDA | GPIO0 | Libre de rol |
| SCL | GPIO1 | Libre de rol |
| Buzzer | GPIO4 | Reservado a JTAG; inactivo en operación |
| LED | GPIO8 | LED de placa, lógica invertida |

**No usar:** GPIO2 y GPIO9 (strapping), GPIO20 y GPIO21 (UART0).

**Ignorar la serigrafía de I2C.** Rotula SDA/SCL en GPIO8/GPIO9: GPIO8 es el LED y ambos son strapping. En el C3 el I2C se asigna por matriz a cualquier pin.

### 4.3. Alimentación

El positivo y la masa van **de conector a conector por cable directo**, sin pasar por las pistas de la placa. La corriente del receptor y los servos no circula por el ALMRT.

En la derivación hacia la placa, en este orden: Schottky en serie → 470 µF + 100 nF → pin 5V.

El diodo cumple cuatro funciones:

1. Retiene la alimentación de la placa ante caídas del riel por transitorios de servo (impide brownout)
2. Protege ante conexión de polaridad invertida
3. Impide que, al conectar el USB para leer el vuelo, se alimenten receptor y servos desde el puerto de la PC
4. Permite operar con BEC de 5 V o 5,5 V

El capacitor absorbe el ruido eléctrico de los servos; el cerámico complementa el filtro. Se documentaron combinaciones de ESC y servos capaces de reiniciar el microcontrolador sin este filtro.

**Advertencia heredada de v3.0 §13:** si el BEC es nominal de 5 V pero entrega menos, el diodo introduce una caída adicional contraproducente. Evaluar si la compatibilidad dual 5 V / 5,5 V es necesaria.

### 4.4. Entrada de acelerador

Señal del receptor → resistencia 10 kΩ → GPIO3. Diodo Schottky desde GPIO3 a 3V3, con la banda del lado del 3V3.

Acepta receptores de 3,3 / 4,8 / 5 / 6 V sin cambiar componentes. Corriente máxima con 6 V: 240 µA.

**No se usa pull-up interno.** El VIH del C3 es 0,75 × VDD = 2,475 V; un receptor de 3,3 V ya lo supera. El pull-up interno de 45 kΩ contra los 10 kΩ serie solo reduciría el margen del nivel bajo.

*El máximo absoluto de entrada del C3 es VDD + 0,3 = 3,6 V. El Schottky recorta en ~3,55 V. Un diodo de silicio común recortaría en 4,0 V, fuera de especificación.*

### 4.5. Salida al ESC

GPIO10 → gate del MOSFET. Source a masa. Drain al hilo de señal del ESC. Pull-up de 1 kΩ del drain al riel del BEC. Pull-up de 10 kΩ del gate a 3V3.

**Lógica invertida:** gate alto → salida baja. El firmware escribe el complemento.

*Por qué pull-up de gate y no pull-down:* durante los ~100 ms de arranque el GPIO está en alta impedancia. El pull-up mantiene el MOSFET conduciendo y la salida en bajo — el ESC no recibe señal válida y no arma.

*Por qué 1 kΩ y no 10 kΩ en el drain:* contra el pull-down interno típico de un ESC (~10 kΩ), un pull-up de 10 kΩ dejaría el nivel alto en la mitad de la tensión del BEC, y el flanco de subida sería lento.

La salida entrega la tensión del BEC, no 3,3 V. Esto garantiza compatibilidad tanto con ESC de lógica 5 V (umbral 3,5 V) como de lógica 3,3 V.

### 4.6. Sensor

VDD a 3V3, GND a GND, SDA a GPIO0, SCL a GPIO1. Montado sobre la placa, dentro del pod del modelo.

El pod cumple el blindaje de luz de EDIC 2.4.8.2 y define el venteo estático de 3.1.4.

El nivel alto del bus lo establecen las resistencias pull-up del módulo a 3,3 V.

### 4.7. Advertencia sobre la placa

La ESP32-C3 Super Mini **no tiene diseño oficial ni fabricante único**. Las variantes difieren en LDO, LED y protecciones.

- Contrastar el pinout con la serigrafía de cada lote
- Leer el modelo del regulador en el encapsulado antes de dar por buenos los 5,5 V del BEC (variantes con ME6211 rondan 6 V de máximo)
- **No hay diodo de protección entre el pin 5V y el VBUS del USB**: nunca alimentar por ambos simultáneamente

---

## 5. Identificación del dispositivo

### 5.1. Número de serie

Hash determinístico de 8 caracteres en base36 sobre los 26 bytes de coeficientes de calibración del BMP280, leídos por I2C desde el registro 0x88.

**Fundamento de la elección.** El C3 tiene MAC única en eFuse, pero **no viene protegida contra escritura de fábrica**: se pueden quemar bits adicionales, y el firmware puede reportar cualquier valor. El hash del sensor es más fuerte: el dato está en el silicio del BMP280, fuera del alcance del firmware. Un verificador que sospeche del firmware carga el suyo y obtiene el valor real.

Como cada dispositivo se ensambla con un único sensor y no se desensambla, el número de serie es único e invariante durante toda su vida útil.

El algoritmo es **invariante**: cualquier reimplementación debe producir exactamente el mismo hash, para preservar la continuidad del registro histórico.

#### 5.1.1. Constantes

| Nombre | Valor |
|---|---|
| OFFSET | `0xCBF29CE484222325` |
| PRIME | `0x100000001B3` |
| ALFABETO | `0-9A-Z` (36 símbolos, en ese orden) |

#### 5.1.2. Convenciones aritméticas

Aritmética entera sin signo de 64 bits con desborde modular (módulo 2⁶⁴). XOR bit a bit. El término `byte[i] × (i+1)` es máximo 6630 y no desborda; el desborde modular aplica al producto por PRIME.

#### 5.1.3. Algoritmo

```
funcion CalcularHashBase36(datos[0..N-1], N) -> cadena de 8 caracteres
    hash = OFFSET                        // uint64
    para i desde 0 hasta N-1:            // índice base 0
        termino = datos[i] * (i + 1)
        hash = (hash XOR termino) * PRIME    // XOR primero, luego multiplicar
        hash = hash mod 2^64
    fin para
    // 8 dígitos base36, del menos significativo al más significativo,
    // escritos de mayor a menor peso (posición 0 = más significativo)
    v = hash
    para p desde 7 hasta 0:
        salida[p] = ALFABETO[v mod 36]
        v = v div 36
    fin para
    devolver salida
fin funcion
```

Notas que eliminan ambigüedad: (1) el orden es XOR y luego multiplicación; (2) el índice es base 0; (3) la reducción equivale a tomar el hash módulo 36⁸, con relleno natural de ceros a la izquierda.

#### 5.1.4. Vectores de prueba

Una implementación correcta debe reproducir exactamente estas salidas sobre 26 bytes de entrada:

| Entrada | Salida esperada |
|---|---|
| `datos[i] = i` (0x00, 0x01, …, 0x19) | `V6MRUXJB` |
| `datos[i] = 0xFF` (los 26 bytes) | `Z71AVNXS` |
| `datos[i] = 0x00` (los 26 bytes) | `TS1HDN0T` |

*Estos tres vectores fueron verificados por ejecución independiente del algoritmo tal como está especificado arriba.*

#### 5.1.5. Manejo de error

Si la transacción I2C falla —NACK, lectura incompleta o timeout— el número de serie es la cadena centinela `ERROR000`.

No se garantiza la unicidad del hash ni que el fabricante produzca coeficientes únicos. Sobre más de 1000 sensores no se registraron colisiones. En caso de colisión, el sensor se descarta y se reemplaza.

### 5.2. Versión de firmware

La versión y los parámetros de la categoría están fijos en el código fuente. Cualquier cambio de parámetros o de comportamiento exige una versión nueva. Versión y parámetros son parte del mecanismo de identificación e integridad.

Como el dispositivo no tiene display, **la revisión de firmware debe estar impresa de forma permanente en el dispositivo** (EDIC 2.1.2).

### 5.3. Reporte serial

Al iniciar, el dispositivo emite por USB CDC un reporte con:

- Versión de firmware y parámetros de configuración activos
- Número de serie del sensor (puede ser `ERROR000`)
- Datos completos del último vuelo registrado, con alturas calculadas y tiempos en segundos

El reporte se emite **exista o no un host conectado**; el proceso continúa sin esperar confirmación. Al terminar se libera el puerto.

Se declara 115200 baudios por continuidad con el procedimiento de verificación existente. En USB CDC el valor es nominal y no afecta la velocidad real.

**El enlace es de solo lectura.** No existen comandos de escritura, por lo que no hay ajustes modificables por ningún medio (satisface EDIC 2.1 d y F5L 5.5.12.7 c).

Con esto se cumple el requisito de poder consultar los datos del último vuelo después del vuelo. El verificador conecta una terminal serial — por ejemplo un teléfono Android con cable OTG y una aplicación de terminal, o cualquier notebook.

**El ALMRT debe desconectarse del modelo antes de conectar el USB.** Esta instrucción debe figurar explícitamente en el procedimiento de verificación.

#### 5.3.1. Formato

```
ALMRT v14.0 - (c) Cristian Leiva

90 metros - 30 Segundos - Rearranque

Sensor           : BMP280

Serie Sensor     : HT7822E6

- Ultimo Vuelo -

Presion Inicial  : 1008.86

Presion Corte    : 998.10

Presion Zoom     : 995.67

Acelerador Inicial: 924

Acelerador Arranque: 1212

Acelerador Corte : 1592

Acelerador Maximo: 1632

Altura Corte     : 90.32

Altura Zoom      : 110.89

Tiempo Inicio    : 22.43

Tiempo Corte     : 10.33

Tiempo Rearranque: 0.00

Tiempo FailSafe  : 0.00
```

---

## 6. Sistema de indicadores

Dos indicadores físicos: **LED** (GPIO8, siempre presente, lógica invertida) y **buzzer** (GPIO4, opcional).

### 6.1. Modos

| Modo | LED | Buzzer |
|---|---|---|
| 0 | Apagado | Apagado |
| 1 | Activo | Inactivo |
| 2 | Activo | Activo |

### 6.2. Patrón de indicación de altura

Representa la altura zoom: la máxima lograda en todo el vuelo con motor más los 10 segundos de cuarentena. Se representa en tres dígitos decimales (centenas, decenas, unidades) emitidos en secuencia y repetidos en loop continuo. La altura es siempre diferencial respecto al piso —no negativa y dentro del rango reglamentario—, por lo que tres dígitos alcanzan.

Un **destello** es el ciclo completo de un estado apagado seguido de un estado encendido. Un **destello largo** es un encendido sostenido de cuatro estados consecutivos sin apagado previo.

- **Dígito 1 a 9:** N destellos consecutivos. El observador los cuenta.
- **Dígito 0 significativo:** un destello largo — cuatro estados encendidos, distinguible por su duración.
- **Dígito 0 no significativo** (ceros a la izquierda): silencio de cuatro estados apagados — misma duración que el destello largo, sin luz, para mantener el ritmo.

Entre cada par de dígitos se inserta una pausa. Al final del tercer dígito, una pausa larga antes de repetir, visualmente distinguible de la pausa entre dígitos.

La duración de cada estado es configurable en compilación y debe ser perceptible a simple vista.

### 6.3. Patrón de indicación de rearranque

Un estado encendido seguido de uno apagado, repetido en loop continuo. Destello simple y veloz, inconfundible con la codificación de dígitos.

### 6.4. Uso por etapa

| Momento | Modo | Contenido |
|---|---|---|
| Al encender | 1 | Altura zoom del vuelo anterior; si `TiempoRearranque > 0` → patrón de rearranque. Permanece hasta el arranque |
| Durante el vuelo activo | 0 | Detenido |
| Post-vuelo sin rearranque | 1 | Altura zoom del vuelo actual |
| Post-vuelo con rearranque confirmado | 2 | Patrón de rearranque, hasta el corte de energía |

La indicación de rearranque persiste en memoria no volátil y reaparece en cada encendido hasta completar un vuelo nuevo sin rearranque.

El LED encendido al alimentar satisface también el requisito de EDIC 2.5 a): indicación claramente visible de que el dispositivo está reseteado y armado.

### 6.5. Jerarquía de verificación de rearranque

Tres medios, en orden de inmediatez:

1. **Buzzer/LED** — audible y visible durante el evento en vuelo y al aterrizar
2. **LED** — visible al encender durante la inspección técnica
3. **Reporte serial** — verificación formal con todos los datos

---

## 7. Sistema de captura de acelerador

### 7.1. Contrato público

La consulta de acelerador retorna exclusivamente:

- **`0`** — no hay información confiable disponible (condición de failsafe, sin excepción)
- **800 a 2200** — valor válido en microsegundos

La decisión de qué hacer ante un `0` corresponde a la etapa que consulta (§11), no a esta función.

### 7.2. Garantías del valor retornado

Cuando el valor es distinto de `0`, cumple simultáneamente:

1. **Integridad del pulso** — la trama contiene exactamente un símbolo. Más de uno indica un flanco espurio que partió el pulso.
2. **Rango físico válido** — entre 800 y 2200 µs.
3. **Frame válido** — período entre flancos de subida consecutivos, dentro del rango configurado.
4. **Frescura** — menos de 300 ms de antigüedad.
5. **Representatividad estadística** — es la mediana de 3 muestras consecutivas válidas.

Si no puede garantizar las cinco, retorna `0`.

**Un pulso y un frame son magnitudes distintas, cada una con su propio rango de validez.** Un pulso pertenece a un frame porque comparten el mismo flanco de subida inicial.

### 7.3. Implementación sobre C3

La captura se realiza con el periférico **RMT en modo recepción** (canales 2 o 3 del C3; los canales 0 y 1 son solo transmisión). El MCPWM no existe en este chip.

**Configuración:**

- Resolución 1 MHz — cada cuenta es 1 µs, unidad natural del PWM de servo
- `signal_range_min_ns` = 2000 ns — filtro de glitches por hardware
- `signal_range_max_ns` = 5 000 000 ns — umbral de cierre de trama

**Sobre el umbral de cierre.** No es la duración del silencio: es cuánto silencio necesita observar el periférico antes de dar la trama por terminada. Debe ser mayor que el pulso más largo (2200 µs) y menor que el silencio más corto (13 500 − 2200 = 11 300 µs). El valor de 5000 µs queda holgado por ambos lados.

**Sobre el filtro de glitches.** Su tope de hardware es ≈3,2 µs (registro de 8 bits contando ciclos del APB a 80 MHz). No puede usarse como umbral de validez de 800 µs — y no debe: el filtro elimina la transición, de modo que un pulso espurio de 400 µs desaparecería sin dejar rastro. Con el filtro en 2 µs ese pulso llega al buffer, se descarta por rango, y puede contabilizarse.

**Ciclo de operación.** El driver detiene el receptor al cerrar cada trama; hay que relanzar la recepción. La ventana disponible es de ~7 ms, margen amplio sobre la latencia de una tarea de prioridad alta.

**Reconstrucción del instante del flanco de subida:**

```
t_flanco = t_cierre − ancho − (signal_range_max_ns / 1000)
```

**Modelo productor/consumidor.** Una tarea productora valida cada trama y publica `{ancho, t_flanco}` en un buffer rodante de las últimas 3 muestras válidas consecutivas. Si una muestra no supera la validación, el conteo de consecutivas se reinicia. La consulta verifica frescura y devuelve la mediana sin demora adicional.

El presupuesto de 300 ms es el límite ante pérdida de señal, no la latencia esperada. En régimen la latencia es de un frame.

### 7.4. Nota sobre la mediana y el retardo

En régimen la ventana de tres ya está llena: entra una muestra, sale la mediana. El retardo es de **un frame**, no de tres. Los tres frames son el llenado inicial, que ocurre segundos antes del lanzamiento.

Un retardo de un frame satisface EDIC 4.2.2 c) —la señal al ESC sigue a la señal de comando— con el mismo orden de retardo que introduce cualquier receptor.

**La mediana no se aplica por magnitud de salto.** Con una llave de dos posiciones el acelerador salta de 1000 a 2000 en un frame, y eso es legítimo. La mediana protege contra un valor espurio aislado, no contra cambios grandes.

---

## 8. Sistema de salida al ESC

### 8.1. Generación

Periférico **LEDC**, 50 Hz, 14 bits (máximo del ESP32-C3).

| Magnitud | Valor |
|---|---|
| Paso | 1,22 µs |
| Duty para 1000 µs | 819 |
| Duty para 2000 µs | 1638 |
| Fórmula | `duty = ancho_µs × 16384 / 20000` |
| Corrección por MOSFET inversor | `duty_escrito = 16383 − duty` |

Se escribe una vez y el hardware repite el pulso indefinidamente, sin intervención de la CPU. Solo se reescribe cuando el valor cambia.

*La resolución de 1,22 µs es peor que los 0,5 µs del hardware AVR anterior y no tiene consecuencia: el ESC no distingue esa diferencia y el EDIC exige 0,1 s de precisión temporal.*

### 8.2. Señal de override

Cuando el dispositivo debe mantener el motor apagado, emite el **acelerador inicial**. Este valor es 900 µs por defecto (máximo FAI para motor apagado, EDIC 2.5 d) y se reemplaza por el valor real obtenido en el armado.

### 8.3. Estado durante el arranque del firmware

Antes de que LEDC quede configurado, el GPIO está en alta impedancia. El pull-up de gate mantiene el MOSFET conduciendo y la salida en bajo. El ESC no recibe pulsos válidos y no arma. Este es el estado seguro y coincide con la lógica invertida en operación.

---

## 9. Medición de presión y cálculo de altura

### 9.1. Contrato de lectura

Mismo contrato que el acelerador: retorna **`0`** si no hay lectura confiable, o la presión compensada en Pascales.

Una lectura es válida si es un número real (no NaN) y está entre 75 000 y 105 000 Pa (EDIC 2.1.1 d.1). Las que no cumplen ambas condiciones cuentan como fallidas.

**Frescura:** menos de 100 ms de antigüedad. Esto garantiza la cadencia mínima de 10 muestras/s del EDIC.

### 9.2. Sensor

**BMP280 exclusivamente.** Direcciones 0x76 o 0x77 según el pin SDO del módulo; el firmware detecta cuál responde validando el Chip ID (registro 0xD0, valor 0x58).

**Driver propio, sobre ESP-IDF.** La compensación se implementa según el procedimiento publicado por Bosch en la hoja de datos del sensor. Para homologación, el EDIC exige la hoja de datos del fabricante como documentación de respaldo; un driver que sigue ese procedimiento es directo de defender.

**Configuración:** modo normal, muestreo continuo y autónomo, oversampling de presión ×8, de temperatura ×8, filtro IIR desactivado, standby mínimo. La lectura solo cosecha el último dato disponible.

El oversampling está expresamente permitido por EDIC 2.4.7. El filtro IIR desactivado no es opcional: es suavizado, prohibido por 2.4.6.

La temperatura se mide internamente para la compensación, por lo que la presión devuelta ya está compensada.

### 9.3. Rechazo de transitorios

Permitido y exigido por EDIC 2.1.1 c). Se descarta una muestra cuya tasa de cambio contra la anterior sea físicamente imposible para un planeador.

El criterio debe ser por **tasa de cambio físicamente imposible**, no por desviación estadística respecto de un promedio — esto último sería suavizado encubierto. El umbral debe documentarse: es lo que un evaluador va a querer ver.

**Prohibido:** promediar, media móvil, o cualquier operación que altere la muestra que representa el pico.

### 9.4. Cálculo de altura

Fórmula ISA del EDIC 2.4.1:

**h = 44330,76923 × (1 − (P / P₀)^0,190266669)**

Donde P es la presión medida y P₀ la presión de referencia del piso, ambas en Pascales. El cálculo es siempre diferencial respecto al piso.

Constantes: K1 = 44330,76923 m; K2 = 0,190266669.

**Las variables se implementan como `float` de 32 bits. Nunca `double`.** El ESP32-C3 no tiene FPU y emula el punto flotante por software; la doble precisión es sustancialmente más lenta. La precisión de `float` es suficiente para el ±0,5 m que exige EDIC 2.4.5.

*El ATmega328P tampoco tenía FPU, así que esto no es una regresión: el C3 emula 32 bits a 160 MHz contra 8 bits a 16 MHz.*

---

## 10. Persistencia

Se utiliza **NVS** sobre la flash interna. El almacenamiento por clave elimina la necesidad de asignar direcciones manualmente.

### 10.1. Datos persistidos

| Dato | Descripción |
|---|---|
| Presión inicial | Promedio de al menos 10 lecturas de presión de piso al arrancar. Referencia de todos los cálculos de altura |
| Presión de corte | Presión mínima registrada durante el vuelo con motor. Se inicializa a la presión inicial |
| Presión zoom | Presión mínima durante el vuelo con motor y los 10 s de cuarentena. Se inicializa a la presión inicial |
| Acelerador inicial | Valor de motor apagado del piloto. 900 µs por defecto, reemplazado al completar el armado |
| Acelerador de arranque | Valor en el momento del arranque |
| Acelerador de corte | Último valor antes del corte. Se inicializa al valor de arranque |
| Acelerador máximo | Máximo registrado durante el vuelo. Se inicializa al valor de arranque |
| Tiempo de inicio | Timestamp del arranque, en milisegundos |
| Tiempo de corte | Milisegundos desde el arranque hasta el corte |
| Tiempo de rearranque | Milisegundos desde el arranque hasta el rearranque. 0 si no hubo |
| Tiempo de failsafe | Milisegundos desde el arranque hasta el primer failsafe. 0 si no hubo |

### 10.2. Momento de escritura

Los datos se persisten **al finalizar la cuarentena**. El registro sobrevive al ciclo de alimentación y se sobrescribe recién en el vuelo siguiente. Alimentar el dispositivo nunca borra el registro: si lo hiciera, sería imposible cumplir F5L 5.5.12.7 e).

### 10.3. Precaución de IRAM

El ESP32-C3 es monocore. Durante una escritura a flash se detienen las interrupciones no marcadas como seguras para IRAM, por decenas de milisegundos en el peor caso.

**El callback de RMT y todo lo que invoque deben estar en IRAM** (`ESP_INTR_FLAG_IRAM`). Los datos constantes que el manejador use requieren `DRAM_ATTR` — el compilador puede ubicar en flash una constante aunque no esté marcada `const`.

*Mitigación de arquitectura:* la escritura ocurre al cerrar la cuarentena, con el vuelo con motor ya terminado y la salida al ESC fija por hardware. Solo es crítico si el rearranque está habilitado, porque en ese caso el acelerador sigue vigilándose después del corte.

---

## 11. Etapas operativas

El dispositivo ejecuta una secuencia lineal de etapas, sin retorno. El **bloqueo terminal** es una condición sin salida: el dispositivo mantiene el acelerador en el valor de acelerador inicial indefinidamente hasta que se corte la energía.

La etapa es una variable de estado compartida entre tareas. El bloqueo terminal es un estado que todas las tareas respetan, no un bucle infinito.

### 11.0. Acciones ante un valor `0` de las funciones de adquisición

| Etapa | Magnitud | Acción ante `0` |
|---|---|---|
| Presión de piso (11.1) | Presión | Descartar la muestra: no cuenta para el promedio |
| Armado / Arranque (11.2 / 11.3) | Acelerador | No se cumple la condición de avance; se sigue esperando (sujeto al timeout de validez de piso) |
| Vuelo con motor (11.4) | Acelerador | Failsafe: corte (override) + registro de tiempo de failsafe |
| Vuelo con motor (11.4) | Presión | Corte (override) + registro de tiempo de failsafe |
| Cuarentena (11.5) | Presión | Descartar la muestra |
| Post-vuelo (11.6) | Acelerador | Failsafe post-rearranque: override + registro de failsafe (1ª vez) |

### 11.1. Inicialización

**Configuración de hardware.** Pines, RMT, LEDC, I2C, NVS. Salida al ESC en 900 µs (override FAI).

**Lectura de NVS.** Carga los datos del último vuelo registrado.

**Indicación del vuelo anterior.** Modo 1 (solo LED). Si `TiempoRearranque > 0` → patrón de rearranque; si no → altura zoom del vuelo anterior, redondeada al entero más cercano y clampeada a 0–999. Permanece activa hasta el arranque.

**Verificación del sensor y número de serie.** Intenta el BMP280 en 0x76 y 0x77. Lee los 26 bytes de calibración desde 0x88 y calcula el número de serie. Si el sensor no responde, o falla la lectura de calibración → `ERROR000`.

**Reporte serial.** Emite el reporte incluyendo el número de serie. No bloqueante: no espera host.

**Bloqueo terminal.** Si el número de serie es `ERROR000` → bloqueo terminal inmediato.

**Establecimiento de referencia de altura.** Estabiliza el sensor 200 ms descartando lecturas. Luego, durante el resto de los 1000 ms, obtiene todas las muestras válidas posibles respetando el intervalo mínimo entre muestras. Si no se alcanzan 10 muestras válidas → bloqueo terminal. El promedio es la presión de referencia; `PresionCorte` y `PresionZoom` se inicializan a ese valor.

Inmediatamente arranca el contador de validez de presión de piso.

**Restricción de EDIC 2.5 b/e:** toda esta etapa debe completarse dentro de los 2 s desde la aplicación de alimentación, y durante ella el dispositivo no debe responder a ninguna señal de motor. Ver §13.1 sobre el presupuesto de arranque.

### 11.2. Armado

Espera que la consulta de acelerador retorne un valor distinto de `0` y menor al **umbral inicial**. El contador de validez sigue corriendo; si expira → bloqueo terminal.

Al obtenerlo, ese valor reemplaza al acelerador inicial (900 µs) y se replica inmediatamente a la salida del ESC.

### 11.3. Arranque

Espera acelerador **mayor al umbral de arranque**. El contador de validez sigue corriendo; si expira → bloqueo terminal.

Al detectar el arranque:

- Se registra el timestamp de inicio
- Se registran acelerador de arranque, de corte y máximo, todos al valor de arranque
- `PresionCorte` y `PresionZoom` se inicializan a `PresionInicial`
- Tiempos de corte, rearranque y failsafe se inicializan en cero
- El servicio de indicadores se detiene (modo 0)

### 11.4. Vuelo con motor

Monitoreo continuo:

**Acelerador.** Se replica al ESC en tiempo real. Se actualizan acelerador máximo y de corte.

**Failsafe RC.** Si la consulta retorna `0` → corte, registro de tiempo de failsafe, fin de etapa.

**Presión.** Muestreo continuo; actualiza `PresionCorte` y `PresionZoom` con el mínimo. Si la lectura retorna `0` → corte, registro de failsafe, fin de etapa.

**Corte reglamentario** — cualquiera de los tres termina la etapa:

- Altura calculada ≥ altura máxima configurada
- Tiempo transcurrido ≥ tiempo máximo configurado
- Acelerador ≤ umbral de apagado (el piloto corta)

Al salir, el acelerador se fija en el valor inicial.

*El corte por altura lo detecta la tarea de presión; el corte por tiempo y por acelerador, la tarea de acelerador. La rutina de corte debe ser atómica: dos tareas pueden alcanzarla.*

### 11.5. Cuarentena

10 segundos sin posibilidad de rearranque. El acelerador se mantiene en el valor inicial.

Continúa el muestreo de presión actualizando `PresionZoom` si se registra un nuevo mínimo.

Al finalizar se persisten todos los datos en NVS.

*Nota normativa: la ventana de 10 s proviene de la práctica de la categoría. El texto de la Sección 7 define la ventana de detección de pico hasta el cruce de la altura requerida (2.5), lo cual deja sin cerrar el caso de corte por tiempo por debajo de 90 m — que es el habitual. Los 10 s también figuran en 4.2.3 del EDIC, pero ese apartado describe el procedimiento de F5J. Ver §14.1.*

### 11.6. Post-vuelo

El acelerador se mantiene en el valor inicial. Los indicadores pasan a modo 1 mostrando la altura zoom del vuelo actual.

**Sin rearranque habilitado:** bloqueo terminal.

**Con rearranque habilitado:** el rearranque es un mecanismo de seguridad que permite al piloto recuperar el control del acelerador a costa de invalidar el vuelo.

Cuando el acelerador regresa a un valor menor al umbral inicial, el piloto puede optar por usar el mecanismo. Si luego sube por encima del umbral de arranque, el rearranque queda confirmado:

- Se registra el tiempo de rearranque
- Se persiste en NVS
- Los indicadores pasan a modo 2 con patrón de rearranque, para alertar al juez
- A partir de ahí el piloto tiene control directo del acelerador, replicado sin restricciones

El tiempo de rearranque se registra **una única vez** (primer rearranque confirmado); las activaciones posteriores no lo modifican. El tiempo de failsafe también se registra una única vez.

**Failsafe post-rearranque.** Si la consulta retorna `0` → acelerador inicial al ESC, y se persiste el failsafe solo la primera vez. El dispositivo sigue sensando. Si la señal regresa por encima del umbral inicial, se mantiene el acelerador inicial hasta que el piloto baje por debajo de ese umbral, momento en que recupera el control directo. El ciclo se repite hasta el apagado.

---

## 12. Arquitectura de firmware

### 12.1. Tareas

| Tarea | Prioridad | Función |
|---|---|---|
| Acelerador | Alta | Consume tramas RMT, valida, publica mediana, replica al ESC, evalúa umbrales |
| Presión | Media | Cosecha el sensor, valida, publica con timestamp, detecta corte por altura |
| Indicadores | Baja | Genera los patrones de LED y buzzer |

El callback de RMT corre en contexto de interrupción, en IRAM: encola la trama y relanza la recepción. No bloquea.

Las esperas son bloqueos de FreeRTOS —la tarea se suspende y el planificador cede la CPU—, no espera activa.

### 12.2. Modelo de adquisición

Cada productor publica `{valor, timestamp, válido}`. Cada consumidor verifica la antigüedad contra su presupuesto de frescura y obtiene el valor o el centinela `0`.

*Esto reemplaza el esquema de "función con presupuesto y reintentos internos" del diseño AVR, que existía porque en un bucle único una lectura lenta bloqueaba todo el sistema. Con tareas independientes ese riesgo no existe, y el presupuesto pasa a ser un criterio de frescura, que es lo que realmente importa.*

### 12.3. Fuente de tiempo

El temporizador de sistema, derivado del reloj principal y por lo tanto del cristal externo. **No usar nada apoyado en el oscilador RC interno.**

Con 10 ppm, el error sobre 30 s es de 0,3 ms. El retardo dominante es la detección del arranque: hasta un frame (25 ms). Ambos muy dentro de los 0,1 s del EDIC.

### 12.4. Configuración del proyecto

Todo viaja en el binario; la placa no se modifica físicamente.

| Opción | Valor | Motivo |
|---|---|---|
| `CONFIG_BOOTLOADER_LOG_LEVEL` | Mínimo | Reduce el tiempo de arranque |
| `CONFIG_LOG_DEFAULT_LEVEL` | Mínimo | Ídem |
| `CONFIG_ESPTOOLPY_FLASHMODE` | QIO | Casi duplica la velocidad de carga desde flash |
| `CONFIG_ESP32C3_RTC_CLK_CAL_CYCLES` | 0 | Omite la calibración del reloj lento |
| `CONFIG_BOOTLOADER_SKIP_VALIDATE_ON_POWER_ON` | **NO usar** | Ahorra tiempo pero omite la verificación de integridad del firmware, que es parte de lo que se homologa |

---

## 13. Limitaciones conocidas

### 13.1. Presupuesto de arranque

Los 2 s de EDIC 2.5 b) se cuentan desde la aplicación de alimentación, no desde que corre el firmware. El arranque del C3 —bootloader de ROM, bootloader de segunda etapa, verificación de imagen, inicialización— consume parte de ese presupuesto.

Con la configuración de §12.4, hay un caso reportado en ESP32-C3 de ~96 ms de arranque total. Contra los 1200 ms de la referencia de altura, quedan unos 700 ms de margen.

**Sin esa configuración el arranque puede consumir el presupuesto entero.**

### 13.2. Validez de la presión de piso ante cambios atmosféricos

La presión de piso se mide una única vez; su validez se degrada con el tiempo. El timeout configurable obliga al piloto a reiniciar si no arranca dentro del plazo.

### 13.3. Unicidad del número de serie

La unicidad del hash no está garantizada matemáticamente y el fabricante no garantiza coeficientes de calibración únicos. En la práctica, sobre más de 1000 sensores no se registraron colisiones. En caso de colisión, descartar y reemplazar el sensor.

### 13.4. Tasa de muestreo ante falla del bus I2C

Ante una falla generalizada y reiterada del bus podría no sostenerse la tasa mínima de 10 muestras/s del EDIC. En condiciones normales la tasa supera ampliamente el mínimo.

El driver de I2C del C3 devuelve `ESP_ERR_TIMEOUT` ante bus ocupado o falla de hardware; el timeout es un argumento de la llamada. Dispone además de `glitch_ignore_cnt`, filtro de hardware sobre las líneas, aplicable al ruido del brushless. Si el bus queda trabado, la recuperación manual es pulsos de reloj hasta liberar SDA, condición de stop, y reinicialización del driver.

### 13.5. Resolución de la salida PWM

1,22 µs por paso, contra 0,5 µs del hardware AVR anterior. Sin consecuencia funcional: el ESC no distingue esa diferencia.

---

## 14. Puntos abiertos

### 14.1. Ventana de detección de pico

La ventana de 10 s posterior al corte proviene de la práctica de la categoría, no del texto de la Sección 7. Conviene consultarlo al EDIC antes de presentar a homologación, porque define qué altura se registra.

### 14.2. Recálculo de presupuestos de tiempo

Los siguientes valores están heredados de la plataforma AVR con la biblioteca `Wire` y deben recalcularse con mediciones sobre C3:

- Intervalo mínimo entre muestras de piso (40 ms)
- Timeout de transacción I2C (10 ms)
- Frescura de presión (100 ms)

### 14.3. Tratamiento del reinicio anómalo

`esp_reset_reason()` distingue encendido limpio de brownout, watchdog de tareas, watchdog de interrupciones y pánico.

Un reinicio en vuelo haría que el dispositivo tome referencia de altura cero en el aire y rehabilite el motor. El hardware de §4.3 cubre la causa de alimentación; falta definir el tratamiento de las causas de software.

Opción planteada, pendiente de elaborar: bloqueo terminal ante cualquier causa distinta de encendido limpio, con patrón de indicación propio y registro en NVS.

---

## 15. Sin verificar

Lo siguiente sostiene decisiones de esta especificación y **no fue confirmado**:

| Ítem | Impacto si no se cumple |
|---|---|
| Tasa efectiva del BMP280 con ×8 y standby mínimo | Si no supera 10 Hz con margen, revisar oversampling y presupuesto de ruido |
| Tiempo de arranque real en la placa concreta | Si excede ~800 ms, acortar la ventana de presión de piso |
| Límite de entrada del LDO de la Super Mini | Variantes con ME6211 rondan 6 V; el BEC entrega 5,5 V |
| Si el driver de RMT del C3 ubica su ISR en IRAM por defecto | Determina el trabajo de §10.3 |
| Deriva térmica del BMP280 montado en el pod | Estimada en ~25 cm sobre un presupuesto de 2 m |

Ninguno invalida la elección de plataforma. Son puestas a punto.

---

## 16. Prompt de generación de código

*Genera TODO el proyecto siguiendo esta especificación al pie de la letra y con el código más eficiente posible para el hardware definido.*

*Fuente de verdad: esta especificación y este prompt mandan siempre. Si te entrego un proyecto de referencia, úsalo solo como guía; ante cualquier diferencia, gana la especificación y el prompt. Los valores por defecto son los de las tablas de la sección 3. Esta especificación es autosuficiente: el algoritmo de número de serie (5.1) incluye constantes, convenciones aritméticas y vectores de prueba verificados, por lo que no necesitás código de referencia para reproducirlo.*

*NO portes código del proyecto AVR anterior. La funcionalidad es la misma; la implementación debe aprovechar el hardware nuevo. Cualquier patrón que exista solo por limitaciones del ATmega328P debe descartarse.*

*El código de referencia NO es modelo de estilo. Aunque use retornos tempranos u otros patrones, no los imites: el estilo lo define este prompt.*

*Antes de empezar, hacé TODAS las preguntas que tengas. Una vez que arrancás, no preguntes más: ante una ambigüedad menor tomá la interpretación más fiel a la especificación y documentá el supuesto como comentario en el código.*

*Antes de escribir el primer archivo, analizá el sistema completo, identificá los datos que se producen y consumen entre módulos, y documentá las decisiones de optimización. Luego, para cada archivo: (1) citá los puntos de la especificación que cubre, (2) verificá que no haya redundancias, (3) escribí el código, (4) verificá que cada garantía esté implementada y que no haya operaciones redundantes. Todo ese análisis va como comentarios en el código, no en el chat.*

*Plataforma: ESP-IDF nativo sobre ESP32-C3 Super Mini. No uses el framework Arduino ni bibliotecas de terceros para el sensor: el driver del BMP280 se implementa según el procedimiento de compensación de la hoja de datos de Bosch.*

*El sensor es exclusivamente BMP280, en 0x76 o 0x77. Validá el Chip ID (registro 0xD0 = 0x58).*

*Marcá con IRAM_ATTR el callback de RMT y todo lo que invoque; usá DRAM_ATTR en los datos constantes que ese camino toque. Asigná la interrupción con ESP_INTR_FLAG_IRAM. Dejá un comentario explicando por qué.*

*Usá float de 32 bits, nunca double: el ESP32-C3 no tiene FPU.*

*La versión del proyecto es la 14.0; el nombre del proyecto debe contenerla y debe verse reflejada internamente donde corresponda.*

*Estructura: cada etapa operativa en su propio módulo; las funciones comunes se agrupan por afinidad funcional. La lectura de acelerador es conceptualmente distinta de la salida de acelerador: no tienen afinidad entre sí. No uses #define ni directivas de compilador para valores de configuración; usá constantes.*

*Estilo OBLIGATORIO: programación estructurada en todo el código — un solo punto de salida por función, sin retornos tempranos, control de flujo solo con secuencia, selección e iteración. Únicas excepciones: las ISR y los callbacks del driver. Verificá función por función antes de entregar.*

*Conceptos a respetar sin reinterpretar: pulso y frame son magnitudes distintas con su propio rango; un pulso pertenece a un frame por compartir el flanco de subida inicial. Los umbrales de acelerador (inicial, de arranque, de apagado) no son intercambiables. Las funciones de adquisición devuelven 0 ante fallo y la política ante ese 0 es por etapa (sección 11). Filtrar por rechazo de transitorios está permitido; suavizar o promediar la presión está prohibido.*

*Comentarios en español.*
