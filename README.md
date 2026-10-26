# **Optimizador de rutas de reparto**

Calcula la ruta más corta para repartir en 8 puntos desde un depósito, con y sin límite de carga en el camión, y la compara contra la forma intuitiva de planearla.

## Resultados

| |  |
| --- | --- |
| Sin límite de carga, ruta óptima | 56.99 km |
| Con límite de carga, ruta óptima | 63.21 km |
| Con límite de carga, ruta secuencial | 106.45 km |

Las dos conclusiones salen de comparar esos tres números entre sí.

**La restricción es barata. Planear mal es caro.** Ponerle al camión un límite de 120 unidades encarece el reparto 6.22 km, un 10.9%. Planear ese mismo camión con la regla intuitiva lo encarece 43.24 km, un 68%. El mal plan cuesta siete veces más que la restricción

**Y la regla intuitiva empeora más rápido que el problema.** Sin límite de carga, la ruta óptima le gana a la secuencial por 18.4%, con límite de carga, le gana por 40.6%, agregar una restricción no volvió al atajo un poco peor, sino, lo volvió el doble de peor

Los dos óptimos están verificados contra las 40,320 rutas posibles, así que no son estimaciones

## El problema

Si tienes que repartir en 8 lugares y los visitas en el orden que parece lógico, vas a manejar mucho más de lo necesario. Y si además el camión no aguanta todo de un viaje, el problema cambia de forma.

La trampa de la regla intuitiva "voy al que esté más cerca" es que se siente correcta en cada paso y falla en el conjunto. Deja los puntos lejanos para el final y eso obliga a un viaje largo de regreso.

Con límite de carga, esa misma trampa se vuelve peor por una razón nueva: el camión se llena antes de llegar al grupo lejano, y entonces el grupo lejano queda partido entre dos viajes. Cada mitad paga su propio traslado largo.

Las dos soluciones se ven así:

- **Secuencial:** club, centro comercial, casa amigo · planta, hospital, parque · escuela, museo
- **Óptima:** club, planta · escuela, centro comercial · casa amigo, parque, museo, hospital

La secuencial llena el camión desde el primer viaje (115, 110 y 80 unidades). La óptima hace lo contrario: deja el primer viaje ligero (80, 110 y 115), despacha el grupo cercano en dos vueltas cortas, y se guarda capacidad para barrer todo el grupo del sur en un solo recorrido. No parte el grupo lejano, y ahí está el ahorro.

## Cómo funciona

**Datos.** Un depósito y 8 paradas, con coordenadas tomadas de Google Maps y una demanda por parada.
La demanda total es 305 unidades contra una capacidad de 120, así que el reparto no cabe en menos de tres viajes

**Distancias (Haversine).** Calcula la distancia sobre la superficie de la esfera, no usé Pitágoras sobre las coordenadas ya que un grado de longitud no mide lo mismo en todas las latitudes: aquí son unos 105 km y en el ecuador 111

**Matriz de distancias.** Las 81 distancias se calculan una vez y se guardan en una tabla de 9×9, el algoritmo consulta la tabla en lugar de recalcular senos y raíces en cada paso, y queda independiente de cómo se midan las distancias, al agregar la demanda como cuarto dato de cada punto, la función de distancia no necesitó un solo cambio

**Heurística voraz con recargas.** Siempre al punto no visitado más cercano que todavía quepa en el camión. Cuando ya no cabe nada, regresa al depósito, recarga y sigue

**Búsqueda exhaustiva.** Recorre las 40,320 permutaciones de las 8 paradas. Para cada orden, las recargas no son una decisión, el límite de carga las determina solo, porque le conviene regresar al depósito lo más tarde posible

Eso último vale justificarlo, porque de ahí depende que el óptimo sea un óptimo de verdad. Una visita extra al depósito cambia un tramo directo
x → y por x → 0 → y, y sobre una esfera siempre se cumple que D[x][0] + D[0][y] ≥ D[x][y]: el camino por un tercer punto nunca es más corto que el directo. Es la desigualdad del triángulo, y Haversine la respeta, entonces regresar antes de que se llene el camión solo puede sumar kilómetros, y recorrer todos los órdenes regresando únicamente cuando el límite obliga sí cubre la solución óptima. No se queda nada fuera.

## Cómo correrlo

Abre el notebook en Google Colab y corre las celdas de arriba a abajo. No hay nada que instalar: el algoritmo unicamente usa `math` e `itertools`, que vienen con Python.

Para usar otras coordenadas, cambia la lista `puntos` por tus propias coordenadas y demandas, y ajusta `CAPACIDAD`. Google Maps da las coordenadas con clic derecho sobre el mapa

Dos reglas al elegir los datos: la demanda total tiene que superar la capacidad, si no la restricción nunca se activa; y ninguna demanda individual puede pasar de la capacidad, porque entonces esa parada es inalcanzable y el problema no tiene solución

## Verificación

Cada resultado se mide dos veces por caminos distintos. El algoritmo acumula la distancia mientras construye la ruta; después, un bloque aparte recorre la ruta terminada y vuelve a sumar los tramos. Si las dos cifras no coinciden, entonces el acumulador está mal. También se revisa que la ruta ganadora contenga las nueve ubicaciones y que ningún viaje se pase de la capacidad.

Esto no es adorno. Una versión con un error de lógica daba 60.71 km: un número más bajo que el óptimo real, perfectamente creíble al lado de los demás, y producido por un código que corría sin un solo error de sintaxis, la revisión lo detectó en un paso, porque la ruta ganadora solo contenía tres de las nueve ubicaciones

## Limitaciones

**Las distancias son en línea recta.** Haversine mide de punto a punto sobre la esfera, no por calles. El manejo real es mayor, así que todos los números son optimistas, los porcentajes de mejora se sostienen porque las rutas que se comparan se miden igual

**La búsqueda exhaustiva no escala.** Con 8 paradas son 40,320 rutas y tarda menos de un segundo. Con 12 paradas serían 479 millones. Para más puntos hace falta un solver de optimización.

**Un solo vehículo.** El camión recarga en el depósito y vuelve a salir, pero no toma en cuenta si hay flota  y tampoco toma en cuenta ventanas de horario ni costos

## Qué sigue

- **v3:** múltiples vehículos, función objetivo en pesos (combustible + horas + costo fijo por camión), ventanas de horario por parada, distancias reales por calle, y la ruta dibujada sobre un mapa.
