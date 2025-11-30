# CFSLinux-RBTree

Este proyecto es una simulación del **Completely Fair Scheduler (CFS)** de Linux, que utiliza un **Árbol Rojo-Negro (Red-Black Tree)** como su estructura de datos principal para gestionar la cola de procesos. El objetivo es demostrar visualmente cómo el CFS logra una planificación justa y eficiente de los procesos en un sistema.

## Características

- **Implementación de Árbol Rojo-Negro:** Una estructura de datos de árbol rojo-negro genérica y completamente funcional.
- **Simulador CFS:** Una clase `CFS_Scheduler` que emula la lógica del planificador de Linux, incluyendo el manejo de `vruntime` y prioridades (`nice`).
- **Visualización Interactiva:** Una interfaz gráfica construida con SFML que muestra el estado del árbol rojo-negro y el proceso en ejecución en tiempo real.
- **Panel de Control Interactivo:** Permite pausar, reanudar, cambiar la velocidad de la simulación y añadir nuevos procesos dinámicamente.
- **Suite de Benchmarks:** Mide el rendimiento (tiempo, comparaciones, memoria) de la implementación del árbol rojo-negro y exporta los resultados a CSV.

---

## Limitaciones y Mejoras Futuras

A pesar de que la implementación del planificador basado en Completely Fair Scheduler (CFS) con un Árbol Rojo-Negro logra simular adecuadamente los principios fundamentales de equidad y ordenación por virtual runtime (vruntime), el proyecto aún presenta varias limitaciones por su simplicidad. Estas restricciones ofrecen oportunidades concretas para una evolución más fiel al funcionamiento de un sistema operativo real.

### Principales Limitaciones

*   **Modelo de procesos limitado a tareas CPU-bound:** Actualmente, los procesos no transitan por estados propios de un sistema operativo, como `sleeping`, `waiting for I/O` o `blocked`. La ausencia de estos estados reduce el realismo de la simulación y limita la representación del comportamiento dinámico que CFS debe gestionar en escenarios reales.
*   **Parámetros de planificación estáticos:** El período de planificación es fijo y no se ajusta en función de la carga del sistema. En implementaciones reales, CFS adapta este parámetro para equilibrar latencia e interactividad, lo que no es replicado en la versión actual.
*   **Ausencia de multiprocesamiento y balanceo de carga:** La simulación contempla únicamente un CPU y no modela escenarios multinúcleo ni mecanismos de `load balancing`. Esto restringe el estudio de la distribución de cargas entre hilos de ejecución, un componente esencial en versiones modernas de CFS.
*   **Visualización limitada y poco escalable:** Si bien el árbol Rojo-Negro se representa gráficamente, la visualización no escala adecuadamente a grandes cantidades de procesos y carece de métricas adicionales, como diferencias de `vruntime` o representaciones temporales de la planificación. En cargas elevadas, varios nodos pueden superponerse y dificultar la interpretación de la estructura, afectando la experiencia del usuario y la utilidad analítica del visualizador.

### Mejoras Futuras Propuestas

*   **Incorporación de estados reales de proceso:** Ampliar el modelo para incluir estados como `ready`, `sleeping`, `I/O wait` y `running` permitiría simular comportamientos más complejos e imitar el manejo de procesos que caracterizan a CFS en sistemas operativos modernos.
*   **Ajuste dinámico de parámetros del scheduler:** Implementar un `scheduling period` adaptable según la cantidad de procesos activos permitiría replicar el comportamiento real de CFS, mejorando la equidad y la latencia percibida por los procesos interactivos.
*   **Extensión hacia escenarios multinúcleo:** Incorporar múltiples instancias del planificador, junto con mecanismos de balanceo de carga y migración de procesos, permitiría estudiar configuraciones SMP y analizar uno de los retos más relevantes para planificadores modernos.
*   **Mejoras en la visualización y análisis temporal:** Incluir funciones como zoom, paneles de métricas, líneas de tiempo tipo Gantt y visualización de diferencias de `vruntime` permitiría comprender mejor la evolución del sistema, facilitando el uso educativo y analítico del simulador.

Este proyecto establece una base clara para el estudio de la planificación justa en sistemas operativos y abre diversas líneas de trabajo que pueden ser exploradas en investigaciones posteriores. Las mejoras propuestas ofrecen un marco concreto para profundizar en aspectos avanzados del CFS, ampliar el realismo de la simulación y fortalecer su utilidad académica. Se espera que futuros desarrollos puedan tomar estas recomendaciones como guía para extender el alcance y rigor del modelo presentado.
