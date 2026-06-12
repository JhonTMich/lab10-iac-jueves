# Monitoreo

En este laboratorio exploraremos monitoreo con herramientas disponibles


## Aplicaciones
```bash
docker compose up -d --build
```

## Servicios y URLs
| Servicio       | URL                         | Notas                                  |
|----------------|-----------------------------|----------------------------------------|
| Frontend       | http://localhost:8080       | Hello World + botones de tráfico/carga |
| Backend (API)  | http://localhost:3001       | `/api/hello`, `/metrics`, `/load`      |
| Grafana        | http://localhost:3000       | admin / admin                          |
| Prometheus     | http://localhost:9090       | datasource ya provisionado             |
| Loki           | http://localhost:3100       | datasource ya provisionado             |
| Alloy (UI)     | http://localhost:12345      | estado del recolector de logs          |
| cAdvisor       | http://localhost:8081       | métricas por contenedor                |
| node-exporter  | http://localhost:9100/metrics | métricas del host                    |

## Configuraciones
- **Datasources** Prometheus y Loki (provisionados automáticamente).
- Logs etiquetados por Alloy con `tier=application` o `tier=infrastructure`.

## Actividad
- El **dashboard** (paneles de CPU + logs de app e infra).
- La **alarma** de CPU > 50%.

## Reset
```bash
docker compose down -v   # borra también dashboards/alarmas creados
```

> Nota de versiones: el tag `prom/prometheus:latest` apunta aún a la rama 2.x (LTS),
> por eso fijamos `v3.8.1`. Promtail EOL (2026-03-02); el recolector de logs
> es Grafana Alloy.

# Respuestas a las preguntas

1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos /metrics?

- Porque se manejan dos tipos de información, prometheus y /metrics solo entienden de números para saber cuándo ocurre un problema. Sin embargo, no almacenan texto. Loki es necesario para almacenar los logs, que son los que explican el porqué del problema.

2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

- Aporta reproducibilidad, cualquiera puede levantar el entorno completo desde cero en otra máquina y Grafana ya sabrá exactamente dónde conectarse sin que nadie tenga que entrar a la interfaz a hacer clics, recordar contraseñas o escribir URLs manualmente.

3. El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?

- Muestran valores distintos porque el "host" representa a la máquina completa, mientras que el "contenedor" está aislado y muestra únicamente el consumo específico de esa pieza de software.
Para alertar sobre una aplicación concreta se debe usar la CPU del contenedor. Si se usara la del host, la alarma podría dispararse porque otra aplicación consumió recursos, dando un falso positivo.

4. ¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?


- Evaluation interval es la frecuencia con la que Grafana revisa la métrica y pending period es el tiempo de gracia que la condición debe mantenerse activa antes de enviar la alerta real. "la CPU debe estar por encima de 50% durante 30 segundos seguidos". Sirve para evitar que un pico de procesamiento de un solo segundo dispare falsas alarmas.