# Respuestas conceptuales

## 1. ¿Por qué usar `_loaded_at` y no `fecha_venta` para este control?

Porque miden cosas distintas. `fecha_venta` es tiempo de negocio, es decir cuándo ocurrió la venta. `_loaded_at` es tiempo de sistema, es decir cuándo llegó el registro a `raw`. El riesgo que se quiere controlar es que la carga se retrase y el resumen se reconstruya con información vieja, y eso solo se ve en `_loaded_at`.

`fecha_venta` no sirve para esto. En los tres escenarios las ventas son del 10 al 12 de enero de 2026 y lo único que cambia es `_loaded_at` (hace 2, 18 o 30 horas). Con `fecha_venta` el control daría el mismo resultado en los tres casos, siempre error, aunque la carga fuera de hace 2 horas. Además una venta antigua puede cargarse hoy, y por eso una fecha de negocio vieja no quiere decir que la carga esté atrasada. Por último `fecha_venta` es `DATE` sin hora, así que no alcanza para medir umbrales de 12 y 24 horas. `_loaded_at` es `TIMESTAMPTZ` en UTC y se compara directamente con la hora actual.

## 2. ¿Por qué las pruebas de integridad no garantizan actualidad?

Porque revisan las filas que existen y no cuándo llegaron. `unique`, `not_null` y `valores_ventas` confirman que no hay identificadores repetidos, que no faltan valores y que cantidad y precio son válidos. Un conjunto de datos de hace 30 horas cumple todo eso igual que uno de hace 2 horas. En este ejercicio los seis registros son idénticos en todos los escenarios, por eso el build pasa las pruebas aunque la fuente esté desactualizada.

Además `dbt build` no ejecuta `dbt source freshness`. Por eso el DAG corre primero `comprobar_freshness` y deja `construir_resumen` aguas abajo con la regla `all_success`. Si freshness falla por pasar de 24 horas, el build queda `upstream_failed` y `analytics.ventas_diarias` conserva el resumen anterior en vez de reconstruirse con datos viejos.
