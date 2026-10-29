# Decisiones de diseño — Semana 3

## 1. Punto de entrada

El proyecto mantiene un único punto de entrada:

```text
IngestaSensores.main()
```

No se crean aplicaciones independientes por semana.

`BancoDePruebas` es una clase auxiliar y no contiene `main`.

## 2. Búsqueda por timestamp

Se utilizan dos estrategias:

- Búsqueda lineal: no requiere ordenamiento.
- Búsqueda binaria: requiere que el arreglo esté ordenado por timestamp.

Los datos sintéticos de `GeneradorDatos` se generan en orden cronológico, por lo que la búsqueda binaria por timestamp cumple su precondición.

## 3. Búsqueda por PM2.5

No se asume que los datos estén ordenados por PM2.5.

Por tanto, la búsqueda binaria por PM2.5 se conserva como experimento para demostrar el efecto de una precondición incumplida.

## 4. Comparación de String

Los identificadores de estación se comparan mediante:

```java
equals()
```

y no mediante:

```java
==
```

porque se necesita comparar contenido.

## 5. Medición

La comparación principal entre algoritmos utiliza el número de comparaciones.

El tiempo en milisegundos se conserva como evidencia experimental, pero no es la única medida utilizada.

## 6. Evolución del proyecto

La Semana 3 agrega una nueva capacidad a la misma plataforma:

```text
Sensores
   ↓
Ingesta
   ↓
Repositorio
   ↓
Búsqueda
   ↓
Medición de eficiencia
```

La Semana 4 podrá extender esta misma arquitectura para estudiar ordenamiento.

## Semana 5: La cadena - Estructuras Dinámicas y Listas Enlazadas

### Mediciones Registradas en BancoDeListas

* **EXP 1 (Polimorfismo e Interfaz):** RepositorioArreglo y RepositorioEnlazado operaron con el mismo ciclo for sin diferencias en su interfaz externa.
* **EXP 2 (Integridad en Eliminación):** Antes de eliminar: 10 elementos. Tras eliminar la posición 5: tamano() reportó 9 y contarRecorriendo() confirmó 9 nodos reales.
* **EXP 3 (Insertar al Inicio - 100.000 lecturas):**
    * Arreglo: 4.999.950.000 movimientos en ~9.193 ms.
    * Lista enlazada: 0 saltos en ~1 ms.
* **EXP 4 (Recorrer por Posición - 50.000 lecturas):**
    * Arreglo: 50.000 accesos en 0 ms.
    * Lista enlazada con obtener(i): 1.249.975.000 saltos acumulados en ~2.899 ms.
* **EXP 5 (Búsqueda Binaria sobre Lista Enlazada - 50.000 lecturas):**
    * Búsqueda lineal: 50.000 saltos.
    * Búsqueda binaria: 749.990 saltos para realizar las mismas 16 comparaciones.

### Preguntas para la Bitácora

1. **¿Por qué insertar al inicio es más económico en una lista enlazada que en un arreglo?**
   En un arreglo, la posición 0 está anclada al inicio del bloque de memoria contigua. Agregar un dato nuevo allí requiere desplazar los n elementos existentes una posición a la derecha, lo que cuesta O(n) movimientos (en la prueba con 100.000 datos fueron casi 5.000 millones de movimientos). En una lista enlazada, la operación cuesta O(1) tiempo constante: basta con crear el nuevo nodo, hacer que su enlace siguiente apunte a primero y actualizar la referencia primero al nuevo nodo, sin mover ningún elemento previo en memoria.

2. **¿Por qué el acceso a una posición arbitraria es más costoso en una lista?**
   El arreglo utiliza memoria contigua y calcula la dirección física directamente en O(1) con la fórmula base + i * tamaño. En cambio, los nodos de una lista enlazada están dispersos en la memoria; para consultar la posición i es indispensable comenzar en primero y avanzar secuencialmente eslabón por eslabón mediante getSiguiente(). Esto vuelve la consulta O(n), lo que en el Experimento 4 causó más de 1.249 millones de saltos de punteros acumulados.

3. **¿Qué diferencia existe entre un experimento de rendimiento y una prueba unitaria?**
   Una prueba unitaria (JUnit) evalúa la corrección lógica y el cumplimiento de casos límite (lista vacía, eliminar nodo único o primero/último) mediante aserciones exactas (assertEquals, assertTrue). Un experimento de rendimiento (BancoDeListas) mide el comportamiento asintótico, la escalabilidad y el costo en recursos computacionales (saltos de nodos, desplazamientos de casilleros y tiempo en milisegundos) a medida que aumenta el volumen de datos n.

4. **¿Qué caso límite consideras más importante probar en eliminar() y por qué?**
   La eliminación en una posición intermedia puenteando los nodos y la actualización de la referencia ultimo. Si al eliminar un nodo del medio se asigna anterior.setSiguiente(null) en lugar de anterior.setSiguiente(aEliminar.getSiguiente()), se rompe la cadena y se pierden todos los nodos posteriores en memoria sin que salte un error de compilación. Además, si se elimina el último elemento y no se actualiza ultimo, las siguientes llamadas a agregarAlFinal() conectarán nodos a un eslabón que ya no forma parte de la lista válida.

5. **Para el histórico de lecturas de sensores, ¿qué estructura elegirías si predominan las inserciones al inicio? ¿Qué elegirías si predominan las consultas por posición? Justifica ambas decisiones.**
* Si predominan inserciones al inicio: Elegiría la lista enlazada (RepositorioEnlazado), ya que cada lectura entrante se registra en O(1) sin el impacto del desplazamiento masivo de memoria que penaliza al arreglo.
* Si predominan consultas por posición: Elegiría el arreglo (RepositorioArreglo), pues garantiza acceso indexado directo en O(1), evitando el costo cuadrático de recorrer la cadena de punteros secuencialmente.