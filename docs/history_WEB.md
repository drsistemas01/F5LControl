# Historial de cambios — Web F5LControl

> Cambios de la **aplicación web**, del más reciente al más antiguo.
>
> **Referencias:** ✨ Mejora · 🔧 Arreglo · 🎨 Visual

| Versión | Tipo | Cambio |
|:---:|:---:|---|
| **3.1.14** | 🔧 | **Detección de aterrizaje más robusta:** ahora calcula el aterrizaje aunque el **barómetro tenga ruido cerca del piso** o el modelo se **levante/mueva justo después de tocar**. Antes esos vuelos quedaban en **"baja confianza"** sin tiempo de vuelo. Usa un "piso robusto" (percentil) que ignora un pozo/deriva transitorios y tolera más variación cerca del suelo. Sigue **conservadora**: un vuelo cortado arriba (sin descenso a piso) no inventa aterrizaje. |
| **3.1.13** | ✨ | En **Avanzadas (Solo para expertos)** se agregó un deslizador para ajustar el **PWM mínimo del ESC** (900 a 1000 µs). Es lo que se manda al ESC en reposo/failsafe; si tu ESC no arma con el mínimo en 1000, bajalo a 900. Requiere firmware **3.1.7+**. Probar **SIN hélice**. |
| **3.1.12** | ✨ | En la sección **Pantalla** se puede **habilitar/deshabilitar el "reset con el botón"** y ajustar el **tiempo de mantener** (1 a 5 s). Requiere firmware 3.1.6+. |
| **3.1.11** | 🔧 | **Detección de aterrizaje** más robusta: ahora también detecta el aterrizaje cuando la grabación **se corta justo al tocar** (el avión bajó y el registro termina abajo, sin unos segundos quieto en el piso). Antes esos vuelos quedaban sin aterrizaje y sin tiempo de vuelo. |
| **3.1.10** | ✨ | En "Buscar actualizaciones", cada bloque (Web y firmware) muestra su descripción y **el botón de actualizar aparece solo si hay una versión nueva**. Al **cerrar** el cartel del OTA, el equipo **se reinicia** para arrancar con el firmware nuevo. |
| **3.1.9** | 🎨 | La actualización por Bluetooth ahora muestra un **cartel (pop-up) con la barra de progreso** para **ambas** vías (repositorio y por archivo), con el aviso de **no cerrar ni desconectar**. |
| **3.1.8** | ✨ | Botón **"Descargar app web"**: guarda el `index.html` (con la versión en el nombre) para usarlo sin internet o compartirlo. |
| **3.1.7** | 🔧 | La **barra de progreso** de la actualización por Bluetooth ahora se ve al actualizar **desde "Buscar actualizaciones"** (antes estaba dentro de la sección "Solo para expertos" y quedaba oculta). |
| **3.1.6** | ✨ | "Buscar actualizaciones" ahora reconoce las versiones **BETA**: se muestran con una etiqueta 🧪 y como **opcional**, para que decidas vos si la instalás o no (nunca se baja sola). |
| **3.1.5** | 🎨 | Distintivo **"Actualización por Bluetooth (OTA)"** en el pie de la página, indicando que el proyecto soporta actualización de firmware por Bluetooth. |
| **3.1.4** | 🔧 | "Buscar actualizaciones" y la descarga del firmware ahora se leen desde **GitHub Pages** del repo (más estable) en vez de `raw.githubusercontent`, que a veces daba un error temporal (503). Sigue siendo el mismo repo público. |
| **3.1.3** | 🔧 | El veredicto **válido / nulo** del vuelo ya no se marca como nulo por un **bache momentáneo** del acelerador: un corte se cuenta solo si se sostiene. Un rearranque real (sostenido) se sigue detectando como nulo por reglas F5L. |
| **3.1.2** | 🎨 | La actualización de firmware **por archivo** se movió a la sección "Solo para expertos". La actualización **automática desde el repositorio** sigue en "Buscar actualizaciones". |
| **3.1.1** | ✨ | Cuando hay una versión de firmware **más nueva en el repositorio**, la app la baja sola y la actualiza por Bluetooth con un botón (además del selector manual de archivo). |
| **3.1.0** | ✨ | La app ahora puede **actualizar el firmware del equipo por Bluetooth**: elegís el archivo `firmware.bin`, se envía con barra de progreso y, si algo falla o se corta, el equipo sigue con la versión actual. |
| **3.0.22** | 🎨 | El botón **"Consultar log de inicio"** ahora está dentro del recuadro de "Estado del equipo" (que se muestra al conectar), a la derecha; el resultado aparece justo debajo y se puede ocultar con una ✕. |
| **3.0.21** | 🔧 | Se quitó la **altura sobre el nivel del mar** de "Consultar barómetro": era solo una estimación (dependía de la presión del día) y podía confundir. Se mantiene la altura relativa al encendido. |
| **3.0.20** | ✨ | Nueva sección **"Consultar log de inicio"**: muestra cómo fue el último arranque del equipo (barómetro, receptor, mínimo, memoria y resultado), con marcas de OK/falla. |
| **3.0.19** | ✨ | En "Consultar barómetro" se agrega la **altura sobre el nivel del mar (aproximada)**, además de la altura relativa al encendido. |
| **3.0.18** | ✨ | La altura de autoencendido de la pantalla ahora se puede ajustar hasta 10 metros. |
| **3.0.17** | ✨ | La app ahora está en **español e inglés**: elegís el idioma con el botón 🌐 de arriba (arranca en el del dispositivo y recuerda tu elección). |
| **3.0.16** | 🎨 | Textos más claros en la sección Pantalla. |
| **3.0.15** | ✨ | El aviso de nueva versión indica el tipo de cambio y su importancia. |
| **3.0.14** | 🎨 | En el gráfico, el tiempo comienza en el despegue. |
| **3.0.13** | 🎨 | El gráfico ya no muestra alturas negativas. |
| **3.0.12** | 🔧 | Mejor detección del aterrizaje en vuelos prolongados. |
| **3.0.11** | ✨ | Detección del aterrizaje más confiable. |
| **3.0.10** | ✨ | Detección del despegue y separación de los tiempos de vuelo y de motor. |
| **3.0.9** | 🎨 | Instrucciones de conexión más claras. |
| **3.0.8** | 🔧 | Recuperación de la configuración cuando llega incompleta. |
| **3.0.7** | 🎨 | Botón "Cómo conectar" más compacto. |
| **3.0.6** | 🔧 | La ventana "Cómo conectar" se cierra correctamente. |
| **3.0.5** | ✨ | Instrucciones de conexión en una ventana independiente. |
| **3.0.4** | 🎨 | Estado del equipo más fácil de interpretar. |
| **3.0.3** | 🎨 | Panel de estado más compacto. |
| **3.0.2** | ✨ | Nuevo panel de estado del equipo. |
| **3.0.1** | ✨ | Configuración de la pantalla desde la web. |
| **3.0** | 🎨 | Indica si el equipo posee pantalla. |
| **2.9.23** | 🔧 | Corrección del aviso de actualización. |
| **2.9.22** | 🔧 | La web carga siempre la versión más reciente. |
| **2.9.21** | ✨ | El botón Ignorar recuerda la elección. |
| **2.9.20** | ✨ | El aviso de actualización se verifica al conectar. |
| **2.9.19** | ✨ | Verificación automática de la versión de la web. |
| **2.9.18** | ✨ | Aviso de actualización e información del autor. |
| **2.9.17** | 🎨 | El tiempo del gráfico comienza en el arranque del motor. |
| **2.9.16** | 🔧 | Se corrigió un reintento que afectaba la conexión. |
| **2.9.15** | 🔧 | Recuperación de la configuración incompleta. |
| **2.9.14** | 🎨 | Opciones útiles activadas de forma predeterminada. |
| **2.9.13** | 🎨 | Escala de altura protegida e indicación de vuelo válido/nulo mejor ubicada. |
| **2.9.12** | 🎨 | Tiempo de vuelo también expresado en minutos. |
| **2.9.11** | ✨ | Búsqueda manual de actualizaciones. |
| **2.9.10** | ✨ | Más pines disponibles en la configuración. |
| **2.9.9** | 🎨 | Gráfico más limpio. |
| **2.9.8** | 🔧 | El recorte del gráfico se mantiene al ampliar. |
| **2.9.7** | 🎨 | Resumen del vuelo mejor organizado. |
| **2.9.6** | ✨ | Cálculo del tiempo de planeo. |
| **2.9.5** | ✨ | Cálculo de la altura por inercia más confiable. |
| **2.9.4** | 🎨 | Gráfico de mayor altura. |
| **2.9.3** | ✨ | Cálculo de la altura ganada por inercia. |
| **2.9.2** | 🎨 | Ayudas de configuración y consola oculta. |
| **2.9.1** | 🎨 | Indicación de pérdidas de señal en el gráfico. |
| **2.9** | ✨ | Análisis del corte y validez del vuelo. |
| **2.8.3** | 🔧 | La web recupera el estado de diagnóstico. |
| **2.8.2** | 🎨 | Estado de diagnóstico siempre visible. |
| **2.8.1** | ✨ | Control del diagnóstico desde la web. |
| **2.8** | ✨ | Gráfico con altura y señales del motor. |
| **2.7.1** | ✨ | El umbral del motor se obtiene del equipo. |
| **2.7** | 🎨 | Altura del gráfico coloreada según el motor. |
| **2.6.5** | 🎨 | La sección de configuración se inicia contraída. |
| **2.6.4** | 🎨 | Botón "Borrar todos" siempre visible. |
| **2.6.3** | ✨ | Borrado completo y aviso de memoria casi llena. |
| **2.6.2** | 🔧 | Aviso cuando la memoria no está disponible. |
| **2.6.1** | ✨ | Barra de uso de la memoria. |
| **2.6** | ✨ | Archivos de vuelo con información de las señales. |
| **2.5.1** | 🎨 | Opciones avanzadas reorganizadas. |
| **2.5** | 🎨 | Escala automática de altura. |
| **2.4** | 🔧 | El monitor de señal muestra datos solo con señal válida. |
| **2.3** | ✨ | Límites del reglamento protegidos. |
| **2.2** | ✨ | Configuración de los valores de corte. |
| **2.1** | ✨ | Selección de pines desde la web. |
