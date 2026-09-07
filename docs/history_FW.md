# Historial de cambios — Firmware F5LControl

> Cambios del **firmware**, del más reciente al más antiguo.
>
> **Referencias:** ✨ Mejora · 🔧 Arreglo · 🎨 Visual

| Versión | Tipo | Cambio |
|:---:|:---:|---|
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
