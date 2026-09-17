# Historial de cambios — Firmware F5LControl

> Cambios del **firmware**, del más reciente al más antiguo.
>
> **Referencias:** ✨ Mejora · 🔧 Arreglo · 🎨 Visual

| Versión | Tipo | Cambio |
|:---:|:---:|---|
| **3.1.7** | ✨ | El **PWM mínimo del ESC** (el valor de **reposo/failsafe/arranque** que se le manda al ESC apagado) ahora es **configurable de 900 a 1000 µs** (default **900**). Algunos ESC (con FW BLHELI)**no arman** si el mínimo está en 1000; bajándolo a 900 sí lo toman. Se ajusta desde la web en la sección **Avanzadas (Solo para expertos)** con un deslizador. **Aplica al reiniciar.** Probar **SIN hélice**. |
| **3.1.6** | ✨ | **Reset controlado con el botón** (solo versión con pantalla): manteniendo el botón **2 s** (configurable) **con la pantalla encendida** (o sea, en el piso), el equipo **cierra y graba el vuelo**, pone el motor en mínimo y **se reinicia** — sin tener que desconectar la batería. Muestra **"RESET" + barra de progreso** mientras lo mantenés, y **"LISTO"** al cumplirse (soltás y reinicia). Habilitar/tiempo quedan listos para configurar desde la web. |
| **3.1.5** | 🔧 | **Seguridad:** al **resetear** el equipo con la batería puesta, el motor ya **no da un tirón**. Antes, tras el reset, la salida al ESC quedaba ~1 s en un estado indefinido y algunos ESC lo tomaban como acelerador; ahora esa salida se pone en **mínimo/apagado apenas arranca** el firmware. (Recomendado además un pull-down de 10k en la señal del ESC para cubrir el instante previo al firmware.) |
| **3.1.4** | 🔧 | El **corte por sensor** ahora tolera un glitch corto del barómetro: en vez de cortar a las 3 lecturas malas (~150 ms), corta solo si el sensor sigue fallando **~1,5 s seguidos**. Así un bache momentáneo (ruido del motor al arrancar) ya no corta el vuelo, pero un sensor realmente colgado igual corta. |
| **3.1.3** | 🔧 | Tres mejoras de pantalla/registro: (1) la pantalla de **resultado** ahora se maneja **por nivel** — por debajo de la altura de aviso y **sin motor** queda encendida, se apaga solo con motor y **vuelve a encender** al apagarlo (un toque de motor por debajo del umbral ya no la deja apagada). (2) **Reintento de detección del OLED** al arrancar (hasta 3 veces): un glitch del bus I²C al encender ya no deja sin pantalla todo el vuelo. (3) Se elimina la **falsa "pérdida de señal" de ~10 s** que aparecía en el registro después de cada corte (la ventana de protección ahora sigue leyendo el receptor). |
| **3.1.2** | 🔧 | Dos correcciones de seguridad/robustez: (1) tras un **rearranque**, si se perdía y volvía la señal con el acelerador al mínimo, podía darse un **breve impulso de motor**; ahora el filtro se reinicia y la salida sube suave desde el mínimo. (2) El estado de "motor encendido" ahora es **único y con antirrebote**: un bache momentáneo del acelerador ya no prende la pantalla a mitad de corrida ni provoca una escritura a memoria con el motor girando. |
| **3.1.1** | 🎨 | Durante la actualización por Bluetooth, la pantalla muestra **UPDATE** con una **barra de progreso** y el porcentaje recibido. Si la actualización se corta, la pantalla vuelve al estado de Bluetooth. |
| **3.1.0** | ✨ | El equipo ahora puede **actualizar su firmware por Bluetooth** desde la web, sin cable. La actualización se inicia solo a pedido y, si algo falla o se corta, el equipo sigue con el firmware anterior. |
| **3.0.25** | 🔧 | El firmware ocupa bastante menos espacio (alrededor de 400 KB menos): se quitó el Wi-Fi, que no se usaba (el equipo funciona solo con Bluetooth). Sin cambios de funcionamiento. |
| **3.0.24** | ✨ | El cero de altura se vuelve a ajustar solo cada vez que la lectura se aleja del cero (con el equipo quieto, también en Bluetooth), hasta que das motor. Al arrancar el motor el cero queda congelado y desde ahí se mide la altura del vuelo. |
| **3.0.23** | ✨ | Guarda un reporte del último arranque (barómetro, receptor, mínimo, memoria y resultado) que se puede consultar desde la web. En los avisos de error, la pantalla muestra además un logo de Bluetooth chico indicando que el equipo quedó accesible por Bluetooth. |
| **3.0.22** | 🎨 | En la pantalla de listo para volar, la altura en metros se actualiza sola cada segundo con la lectura actual del sensor. |
| **3.0.21** | 🔧 | Al encender, el equipo reintenta detectar el sensor de altura antes de dar el aviso de falta de barómetro, para evitar falsos avisos por un fallo momentáneo de conexión o arranque. |
| **3.0.20** | 🎨 | La pantalla de listo para volar muestra la altura en metros (número grande) junto al tilde de OK, y ya no muestra el dato del acelerador. |
| **3.0.19** | ✨ | La altura a la que la pantalla se enciende sola al aterrizar ahora llega hasta 10 metros y viene configurada en 10 metros por defecto. |
| **3.0.18** | 🔧 | Al encender, el cero de altura se calcula de forma más robusta para que el equipo parta lo más cerca posible de 0 m y no muestre desvíos de varios metros estando quieto. El equipo sigue quedando listo para vuelo en 2 segundos. |
| **3.0.17** | 🔧 | La pantalla de resultado ya no queda detenida en un dato al aterrizar (alterna siempre la altura máxima y la altura adicional). La altura de autoencendido predeterminada pasa a 3 m, y una vez activada la pantalla solo se apaga al volver a encender el motor. |
| **3.0.16** | ✨ | Se agregó compatibilidad con un nuevo sensor de altura (BME280). |
| **3.0.15** | 🎨 | La pantalla de Bluetooth muestra el nombre del equipo. |
| **3.0.14** | ✨ | Se mejoró el ingreso al modo Bluetooth y se agregaron avisos en pantalla cuando el equipo no puede iniciar el vuelo. |
| **3.0.13** | 🎨 | Al finalizar el vuelo, la pantalla muestra la altura máxima y la altura adicional ganada tras el corte del motor. |
| **3.0.12** | 🔧 | Se corrigió que la pantalla quedara congelada al reiniciar el equipo. |
| **3.0.11** | ✨ | Se optimizó el guardado de datos para cuidar la memoria del equipo. |
| **3.0.10** | 🔧 | Se corrigió la pérdida de datos en vuelos prolongados. |
| **3.0.9** | 🔧 | La altura máxima de la pantalla coincide con la de la aplicación web. |
| **3.0.8** | 🔧 | Se solucionó el inicio del equipo en placas con pantalla. |
| **3.0.7** | 🔧 | Primer arreglo del inicio con pantalla (mejorado en la 3.0.8). |
| **3.0.6** | ✨ | El vuelo se guarda al confirmarse el despegue, conservando los segundos previos. |
| **3.0.5** | ✨ | Guardado de vuelos orientado a cuidar la memoria (revisado en la 3.0.10). |
| **3.0.4** | 🔧 | Recuperación automática de la memoria dañada para poder volar. |
| **3.0.3** | ✨ | Calibración del acelerador más compatible. |
| **3.0.2** | 🎨 | La aplicación web puede ver el estado del receptor. |
| **3.0.1** | 🎨 | Nuevas pantallas de estado y de resultado del vuelo. |
| **3.0** | ✨ | Soporte inicial para pantalla. |
| **2.6.20** | 🔧 | Respuesta más segura ante la pérdida de señal del receptor. |
| **2.6.19** | 🔧 | Registro de vuelos más robusto. |
| **2.6.18** | ✨ | Respuesta más rápida del control. |
| **2.6.17** | 🔧 | Corrección de la lectura de señal en dos pines de entrada. |
| **2.6.16** | 🔧 | Inicio más confiable con el sensor de altura. |
| **2.6.15** | ✨ | Más pines disponibles para las conexiones. |
| **2.6.14** | ✨ | Compatibilidad con nuevos sensores de altura. |
| **2.6.13** | ✨ | Inicio del equipo más rápido (2 segundos). |
| **2.6.12** | ✨ | La descarga incluye el motivo del corte del motor. |
| **2.6.11** | ✨ | La configuración de diagnóstico queda guardada. |
| **2.6.10** | 🔧 | El motor ya no funciona entrecortado durante el registro. |
| **2.6.9** | 🔧 | Corrección del formateo de la memoria. |
| **2.6.8** | ✨ | Altura de referencia más estable al encender. |
| **2.6.7** | ✨ | Lecturas de altura más estables. |
| **2.6.6** | 🎨 | Indicaciones más claras con el LED durante el vuelo. |
| **2.6.5** | 🔧 | Corrección de la calibración del acelerador. |
| **2.6.4** | 🔧 | Corrección de la lectura de la señal del receptor. |
| **2.6.3** | 🔧 | Corrección de la señal de salida hacia el motor. |
| **2.5** | ✨ | Límites de la categoría fijos y monitoreo por Bluetooth. |
| **2.4** | ✨ | Registro de vuelos en la memoria. |
| **2.3** | ✨ | Lógica completa de corte del motor. |
| **2.2** | ✨ | Control de la salida hacia el motor. |
| **2.1** | ✨ | Lectura de la señal del receptor. |
| **2.0** | ✨ | Estructura inicial del firmware. |
