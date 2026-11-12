# Evidencias del examen de ventas

## Ejecuciones evaluadas

| Escenario | Run ID | comprobar_freshness | construir_resumen |
| --- | --- | --- | --- |
| actualizada (carga hace 2 h) | `manual__2026-11-12T11:45:21.493150+00:00` | success | success |
| desactualizada (carga hace 30 h) | `manual__2026-11-12T11:50:55.305907+00:00` | failed | upstream_failed |

`ejecuciones.txt` también lista otras dos ejecuciones. La de las 11:36 fue un intento previo con la fuente desactualizada y la de las 11:48 una repetición del escenario actualizado. Las que se evalúan son las dos de la tabla.

## Archivos

| Archivo | Qué demuestra |
| --- | --- |
| `01_actualizada_success.png` | Escenario actualizado. Las dos tareas en success con el mismo Run ID. Freshness empieza a las 11:45:25 y build a las 11:45:28, así que freshness corre antes del build |
| `02_desactualizada_estados.png` | Escenario desactualizado en una ejecución nueva. Graph con freshness en failed y build en upstream_failed |
| `03_log_freshness_desactualizada.png` | Log de freshness del run de las 11:50:55. Aparece `INICIO dbt ['source', 'freshness', ...]` y luego el `CalledProcessError` porque dbt terminó con código 1 |
| `04_build_upstream_failed.png` | Build del mismo run en upstream_failed, con duración 00:00:00 y trigger rule all_success. No tiene intento ejecutado ni `INICIO dbt` |
| `05_construir_resumen_sin_ejecuciones_fallidas.png` | Duración del build por ejecución. Solo hay barras en las ejecuciones actualizadas y en las fallidas queda en cero |
| `06_terminal_resumen_y_ejecuciones.png` | Consulta del resumen después del bloqueo y lista de ejecuciones con sus estados |
| `resumen_antes.txt` | `analytics.ventas_diarias` después del escenario actualizado (3 filas, 46.00, 51.25 y 12.40) |
| `resumen_despues.txt` | El mismo resumen después del bloqueo. Las filas se conservan porque build no se ejecutó |
| `ejecuciones.txt` | Salida de `airflow dags list-runs` con run_id, estado y horas de cada ejecución |
| `log_freshness_desactualizada.txt` | Log completo de la tarea `comprobar_freshness` del run de las 11:50:55 |
| `artefactos_dbt_desactualizada.txt` | `sources.json` del freshness de las 11:50:59, con status error, `max_loaded_at` de hace unas 30 h y los umbrales de 12 y 24 h |
| `dbt_log_freshness.txt` | Líneas de `dbt.log` con `ERROR STALE freshness of ventas_raw.ventas` |
| `respuestas.md` | Respuestas a las dos preguntas conceptuales |

## Por qué el motivo STALE no aparece en el log de Airflow

`run_dbt` ejecuta dbt con `subprocess.run` sin capturar la salida, así que dbt escribe directamente en la salida estándar del proceso. Airflow solo redirige al log de la tarea lo que pasa por `sys.stdout` de Python, como el `print` de `INICIO dbt`. Por eso el log de Airflow muestra el inicio y el código de salida 1, y el detalle `ERROR STALE` queda en los artefactos y en el log de dbt (`/state/dbt/target/sources.json` y `/state/dbt/logs/dbt.log`). Las horas coinciden con el log de la tarea, 11:50:58 al inicio y 11:50:59 al fallo.
