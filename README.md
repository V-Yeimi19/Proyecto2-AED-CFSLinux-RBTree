# CFSLinux - Simulador de Scheduler CFS con Red-Black Tree

Este proyecto implementa una simulación del **Completely Fair Scheduler (CFS)** de Linux, utilizando un **Árbol Rojo-Negro (Red-Black Tree)** como estructura de datos subyacente para gestionar los procesos de manera eficiente.

## 📋 Descripción del Proyecto

El objetivo principal es demostrar cómo el kernel de Linux utiliza un árbol balanceado para mantener los procesos ordenados por su tiempo de ejecución virtual (`vruntime`). El CFS busca maximizar la utilización de la CPU y asegurar que todos los procesos reciban una cantidad justa de tiempo de procesador.

El proyecto incluye:
1.  Una implementación completa de un **Red-Black Tree** en C++.
2.  Una clase `CFS_Scheduler` que simula la lógica del planificador.
3.  Una interfaz visual (simulación) y un menú interactivo para probar las estructuras.
4.  Benchmarks para analizar el rendimiento (tiempo, comparaciones y memoria).

## 🛠️ Implementación

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

## 📊 Análisis de Complejidad

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

## 🚀 Uso

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
