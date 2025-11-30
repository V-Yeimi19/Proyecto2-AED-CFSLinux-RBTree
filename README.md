# CFSLinux - Simulador de Scheduler CFS con Red-Black Tree

Este proyecto implementa una simulación del **Completely Fair Scheduler (CFS)** de Linux, utilizando un **Árbol Rojo-Negro (Red-Black Tree)** como estructura de datos subyacente para gestionar los procesos de manera eficiente.

## Descripción del Proyecto

El objetivo principal es demostrar cómo el kernel de Linux utiliza un árbol balanceado para mantener los procesos ordenados por su tiempo de ejecución virtual (`vruntime`). El CFS busca maximizar la utilización de la CPU y asegurar que todos los procesos reciban una cantidad justa de tiempo de procesador.

## Características
- **Implementación de Árbol Rojo-Negro:** Una estructura de datos de árbol rojo-negro genérica y completamente funcional.
- **Simulador CFS:** Una clase `CFS_Scheduler` que emula la lógica del planificador de Linux, incluyendo el manejo de `vruntime` y prioridades (`nice`).
- **Visualización Interactiva:** Una interfaz gráfica construida con SFML que muestra el estado del árbol rojo-negro y el proceso en ejecución en tiempo real.
- **Panel de Control Interactivo:** Permite pausar, reanudar, cambiar la velocidad de la simulación y añadir nuevos procesos dinámicamente.
- **Suite de Benchmarks:** Mide el rendimiento (tiempo, comparaciones, memoria) de la implementación del árbol rojo-negro y exporta los resultados a CSV.

## Implementación

### 1. Red-Black Tree (`RB_tree.h`)
El núcleo del proyecto es la estructura de datos `RB_tree`. Es un árbol binario de búsqueda auto-balanceado que garantiza que la altura del árbol sea logarítmica con respecto al número de nodos.

**Métodos Principales:**
*   **`add_leaf(T key)`**: Inserta un nuevo elemento y rebalancea el árbol (recoloreo y rotaciones) para mantener las propiedades Red-Black.
*   **`delete_leaf(T key)`**: Elimina un elemento y restaura el balance del árbol.
*   **`find(T key)`**: Busca si un elemento existe en el árbol.
*   **`sucesor(T key)` / `predecesor(T key)`**: Encuentra el siguiente o anterior elemento en orden, respectivamente.
*   **`get_min_node()`**: Retorna el nodo con el valor mínimo; esdecir, el más a la izquierda (operación crítica para el CFS).

### 2. CFS Scheduler (`CFS_Scheduler.h`)
El planificador utiliza el `RB_tree` para almacenar objetos de tipo `Process`.

**Lógica del Programa de Planificación:**
*   **Ordenamiento**: Los procesos se ordenan en el árbol basándose en su `vruntime` (virtual runtime).
*   **Selección**: En cada paso (`tick`), el scheduler selecciona el proceso con el **menor `vruntime`** (el nodo más a la izquierda del árbol) para ejecutarlo.
*   **Time Slice**: Se calcula un "time slice" dinámico para cada proceso basado en su peso (prioridad `nice`).
    *   Fórmula: `time_slice = scheduling_period * (peso_proceso / peso_total)`
*   **Actualización de Vruntime**: A medida que un proceso se ejecuta, su `vruntime` aumenta.
    *   Fórmula: `vruntime += delta_exec * (NICE_0_LOAD / peso_proceso)`
*   **Preemption**: Si el proceso actual consume su time slice o si aparece un nuevo proceso con menor `vruntime`, el proceso actual se reinserta en el árbol y se selecciona el nuevo mínimo.

### 3. Proceso (`Process.h`)
Cada proceso tiene atributos clave:
*   `pid`: Identificador del proceso.
*   `nice`: Valor de prioridad que oscila de -20 a 19. Menor valor implica mayor prioridad; es decir, más peso.
*   `vruntime`: Tiempo de ejecución virtual acumulado.
*   `burst_time`: Tiempo total de CPU requerido.

## Análisis de Complejidad

### Complejidad Temporal

| Operación | Complejidad | Justificación |
| :--- | :---: | :--- |
| **Insertar Proceso** | **O(log n)** | Inserción en RB Tree + Rebalanceo (rotaciones O(1)). |
| **Eliminar Proceso** | **O(log n)** | Eliminación en RB Tree + Rebalanceo. |
| **Seleccionar Próximo (Pick Next)** | **O(log n)** | Encontrar el mínimo (`get_min_node`) toma O(log n) en el peor caso (altura del árbol).  |
| **Buscar Proceso** | **O(log n)** | Búsqueda binaria en un árbol balanceado. |

Donde **n** es el número de procesos en el sistema (runqueue).

### Complejidad Espacial 

*   **O(n)**: Se requiere espacio lineal para almacenar los `n` nodos del árbol. Cada nodo almacena el objeto `Process`, punteros (`left`, `right`, `parent`) y el color.

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



## Uso

### Compilación
El proyecto utiliza la librería SFML (para la visualización).

```bash
mkdir build
cd build
cmake ..
make
```

### Ejecución
```bash
./RB-Tree
```

### Menú Interactivo
Al ejecutar el programa, verás un menú con las siguientes opciones:

1.  **Operaciones Básicas**: Insertar, buscar y eliminar nodos manualmente en el árbol.
2.  **Benchmarks**: Ejecutar pruebas de rendimiento para medir tiempo, comparaciones y uso de memoria con grandes volúmenes de datos (10k, 50k, 100k, 200k elementos).
3.  **Simulación CFS**:
    *   **Simulación con precarga**: Corre una demo con procesos predefinidos.
    *   **Simulación personalizada**: Permite configurar procesos manualmente.
