# Proyecto 1 - Entrega 1 - Modelo de optimización para la operación de distribución en Seneca Libre

**Integrantes:**

- Carlos Casadiego
- Mafe De La Hoz
- Sebastián Ospino

**Curso:** ISIS-3302 - Modelado, Optimización y Simulación - 202402  
**Universidad de los Andes**

---

## Presuposiciones

- Todos los vehículos del mismo tipo son idénticos en todos los aspectos.
- Las distancias y tiempos de envío entre distintos puntos son iguales para vehículos del mismo tipo, salvo para drones, que tienen restricciones adicionales.
- Los transportistas tienen un descanso obligatorio de 10 horas entre turnos de trabajo.
- Los turnos de trabajo para todos los transportistas son de 14 horas, iniciando a las 5 a.m. y terminando a las 7 p.m.
- La optimización se realiza a las 5 a.m. de un día cualquiera, considerando la necesidad de enviar un conjunto de productos. Es posible asignar envíos para días posteriores.
- Una ruta circular en la ciudad no puede durar más de 14 horas.
- Existen estaciones de recarga de combustible y energía cerca de todos los puntos de entrega, pero el tiempo de inactividad por recarga depende de la distancia de la ruta. Los drones deben recargar cada 25% de la distancia que recorrería un vehículo de combustión.
- Los vehículos eléctricos solo se recargan una vez al inicio del día, durante un tiempo definido.
- Los vehículos parten y vuelven al mismo centro de distribución, sin pasar por otros centros en la ruta.
- Los puntos de recarga de energía sirven tanto para drones como para vehículos eléctricos.
- Se ignoran problemas de tráfico u otros imprevistos en las consideraciones de distancias y tiempos entre nodos geográficos.

---

## Conjuntos

- \( V \): Conjunto de vehículos.
- \( T \): Conjunto de transportistas.
- \( P \): Conjunto de productos (20,000 envíos/día).
- \( X \): Conjunto de nodos geográficos.
- \( L \): Conjunto de tipos de lugares (punto de entrega, recarga de combustible, recarga de energía, centro de distribución).
- \( TV \): Conjunto de tipos de vehículos (combustión, eléctricos, drones).
- \( A \): Conjunto de arcos entre nodos, donde \( A = \{(i,j) \,|\, i, j \in X\} \).

---

## Parámetros

### Ubicación y Distancias

- \( \text{UbiX}_i \): Coordenadas del nodo \( i \) en grados decimales.
- \( \text{Dist}_{ijv} \): Distancia entre nodo \( i \) y \( j \) para vehículo \( v \).
- \( \text{Tiempo}_{ijv} \): Tiempo de viaje entre nodo \( i \) y \( j \) para vehículo \( v \).
- \( \text{LugX}_i \): Tipo de lugar en nodo \( i \).

### Capacidades y Consumos

- \( \text{Cap}_v \): Capacidad de carga del vehículo \( v \) (kg).
- \( \text{Pes}_p \): Peso del producto \( p \) (kg).
- \( \text{Dest}_p \): Nodo de destino del producto \( p \).
- \( \text{Consumo}_v \): Consumo de combustible o energía por km del vehículo \( v \).
- \( \text{Rango}_v \): Distancia máxima sin recarga del vehículo \( v \).

### Costos

- \( V_{\text{carga}} \): Costo por minuto de carga.
- \( T_{\text{kg}_v} \): Tiempo de carga por kg para vehículo \( v \) (min/kg).
- \( C_{\text{km}_v} \): Costo por km recorrido para vehículo \( v \).
- \( C_{\text{hora}} \): Costo por hora de operación.
- \( C_{\text{fuel}_v} \): Costo por unidad de combustible o energía para vehículo \( v \).
- \( F_{\text{mantenimiento}_v} \): Costo de mantenimiento por km para vehículo \( v \).

### Horarios y Turnos

- \( \text{Turno} \): Duración del turno laboral (14 horas).
- \( \text{Descanso} \): Período de descanso obligatorio (10 horas).

---

## Variables de Decisión

- \( x_{vij} \in \{0,1\} \): 1 si el vehículo \( v \) viaja del nodo \( i \) al nodo \( j \), 0 en caso contrario.
- \( y_{vp} \in \{0,1\} \): 1 si el vehículo \( v \) transporta el producto \( p \), 0 en caso contrario.
- \( s_{vi} \geq 0 \): Tiempo de llegada del vehículo \( v \) al nodo \( i \).
- \( l_{vi} \geq 0 \): Carga del vehículo \( v \) al llegar al nodo \( i \).
- \( w_{vt} \in \{0,1\} \): 1 si el transportista \( t \) es asignado al vehículo \( v \), 0 en caso contrario.

---

## Función Objetivo

Minimizar el costo total de operación:

$$
\begin{aligned}
\min \ \text{Costo Total} =\ & \sum_{v \in V} \left[ V_{\text{carga}} \times T_{\text{kg}_v} \times \left( \sum_{p \in P} \text{Pes}_p \times y_{vp} \right) \right] \\
& + \sum_{v \in V} \left[ C_{\text{km}_v} \times \sum_{(i,j) \in A} \text{Dist}_{ijv} \times x_{vij} \right] \\
& + \sum_{v \in V} \left[ F_{\text{mantenimiento}_v} \times \sum_{(i,j) \in A} \text{Dist}_{ijv} \times x_{vij} \right] \\
& + \sum_{v \in V} \left[ C_{\text{fuel}_v} \times \text{Consumo}_v \times \sum_{(i,j) \in A} \text{Dist}_{ijv} \times x_{vij} \right] \\
& + \sum_{v \in V} \left[ C_{\text{hora}} \times \left( \sum_{(i,j) \in A} \text{Tiempo}_{ijv} \times x_{vij} + T_{\text{recarga}_v} \right) \right]
\end{aligned}
$$

Donde \( T_{\text{recarga}_v} \) es el tiempo total de recarga del vehículo \( v \).

---

## Restricciones

1. **Entrega de Productos**

   - Cada producto debe ser entregado por un único vehículo:

     $$
     \sum_{v \in V} y_{vp} = 1, \quad \forall p \in P
     $$

   - Un vehículo solo puede entregar un producto si visita su destino:

     $$
     y_{vp} \leq \sum_{i \in X} \delta_{i,\text{Dest}_p} \times \sum_{j \in X} x_{vij}, \quad \forall v \in V,\ \forall p \in P
     $$

     Donde:

     $$
     \delta_{i,\text{Dest}_p} = \begin{cases}
     1, & \text{si } i = \text{Dest}_p \\
     0, & \text{en otro caso}
     \end{cases}
     $$

2. **Capacidad de Vehículos**

   - La carga total asignada a cada vehículo no debe exceder su capacidad:

     $$
     \sum_{p \in P} \text{Pes}_p \times y_{vp} \leq \text{Cap}_v, \quad \forall v \in V
     $$

3. **Conservación de Flujo**

   - Para cada vehículo, el flujo que entra y sale de cada nodo debe ser el mismo, excepto en el inicio y fin:

     $$
     \sum_{j \in X} x_{vji} = \sum_{j \in X} x_{vij}, \quad \forall i \in X,\ \forall v \in V
     $$

4. **Inicio y Fin en el Centro de Distribución**

   - Cada vehículo debe iniciar y terminar en el centro de distribución:

     $$
     \sum_{j \in X} x_{v s_v j} = 1, \quad \sum_{i \in X} x_{v i s_v} = 1, \quad \forall v \in V
     $$

     Donde \( s_v \) es el nodo del centro de distribución para el vehículo \( v \).

5. **Tiempo de Ruta y Turno Laboral**

   - El tiempo total de ruta no debe exceder el turno laboral:

     $$
     \sum_{(i,j) \in A} \text{Tiempo}_{ijv} \times x_{vij} + T_{\text{recarga}_v} \leq \text{Turno}, \quad \forall v \in V
     $$

6. **Período de Descanso**

   - Después de completar un turno, el transportista debe descansar el período obligatorio antes de iniciar otro turno:

     $$
     N_v \geq \frac{\text{Tiempo de Ruta}_v}{\text{Turno}} + \frac{\text{Descanso}}{24}, \quad \forall v \in V
     $$

     Donde \( \text{Tiempo de Ruta}_v = \sum_{(i,j) \in A} \text{Tiempo}_{ijv} \times x_{vij} + T_{\text{recarga}_v} \).

7. **Restricciones de Recarga de Vehículos**

   - Los vehículos deben recargar según su rango máximo:

     $$
     \sum_{(i,j) \in A} \text{Dist}_{ijv} \times x_{vij} \leq \text{Rango}_v \times \text{NumRecargas}_v, \quad \forall v \in V
     $$

   - Si la distancia entre dos nodos excede el rango sin recarga, el vehículo debe visitar un punto de recarga. Esto se garantiza incluyendo nodos de recarga en la ruta cuando sea necesario.

8. **Asignación de Transportistas**

   - Cada vehículo es operado por un único transportista:

     $$
     \sum_{t \in T} w_{vt} = 1, \quad \forall v \in V
     $$

   - Un transportista no puede operar más de un vehículo al mismo tiempo:

     $$
     \sum_{v \in V} w_{vt} \leq 1, \quad \forall t \in T
     $$

9. **Consistencia de Tiempos de Llegada**

   - Los tiempos de llegada deben ser coherentes con los tiempos de viaje:

     $$
     s_{vj} \geq s_{vi} + \text{Tiempo}_{ijv} - M \times (1 - x_{vij}), \quad \forall (i,j) \in A,\ \forall v \in V
     $$

     Donde \( M \) es un número suficientemente grande.

10. **Restricciones de Horario de Trabajo**

    - Los transportistas inician a las 5 a.m. y terminan a las 7 p.m.:

      $$
      5 \leq s_{v s_v} \leq 19 - \text{Tiempo de Ruta}_v, \quad \forall v \in V
      $$

11. **Duración Máxima de la Ruta**

    - Una ruta no puede durar más de 14 horas:

      $$
      \text{Tiempo de Ruta}_v \leq 14, \quad \forall v \in V
      $$

12. **Restricciones Específicas para Drones**

    - Los drones deben recargar cada 25% de la distancia que recorrería un vehículo de combustión:

      $$
      \sum_{(i,j) \in A} \text{Dist}_{ijv} \times x_{vij} \leq 0.25 \times \text{Rango}_{\text{combustión}}, \quad \forall v \in V_{\text{drones}}
      $$
---

## Modelo Dinámico

Si las condiciones cambian (por ejemplo, llegan nuevos productos para enviar), se recalcula la asignación óptima de recursos para el nuevo conjunto de productos pendientes de envío, tomando como referencia temporal el momento del cambio. Se evalúa si esta nueva planificación es más óptima que la original para los productos no enviados, o si es mejor mantener la planificación original y optimizar únicamente la asignación de recursos para los nuevos productos.

