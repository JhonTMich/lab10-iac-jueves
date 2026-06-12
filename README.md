# Validación

### Levantar stack
```bash
docker compose up -d --build
```

### Comprobar servicios esten `up`
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

1. **Verificar recolección de Logs y Métricas base:**
   * Ingresa al Frontend (`http://localhost:8080`) y presione el botón **"Saludar (API)"** varias veces.
   * En dashboard de Observabilidad en Grafana (`http://localhost:3000`).
   * En el panel de *Logs de aplicación* deben aparecer los eventos recientes en formato JSON.

2. **Validar la Alarma de CPU (> 50%):**
   * En el Frontend, pulsar el botón **"Generar carga de CPU (30s)"**.
   * En el dashboard en Grafana observar cómo la gráfica del panel *CPU contenedor backend (%)* supera el umbral rojo del 50%.
   * Ir al menú de Grafana: **Alerting → Alert rules**.
   * `CPU backend > 50%` pasará a estado **Pending** durante 30 segundos, y si la carga se mantiene, cambiará a estado **Firing**.

3. **Validar el ciclo cerrado (Webhook):**
   * Una vez que la alarma pase a estado *Firing*, Grafana enviará automáticamente una notificación vía Webhook al backend (`http://backend:3001/alerts`).
   * En el panel de *Logs de infraestructura* se debe haber registrado el evento de recepción de la alerta.

# Explicación componentes
Prometheus se encarga de recolectar y almacenar las métricas numéricas temporales (como el % de CPU). Grafana Alloy funciona extrae los logs de texto directamente del entorno de Docker y los envía a Loki, la base de datos especializada en almacenarlos e indexarlos. Grafana centraliza la observabilidad, actúa como el tablero visual que consulta a Prometheus y Loki para graficar los datos combinados, además de encargarse de evaluar las reglas para disparar las alarmas automáticamente. 

# Respuestas a las preguntas

#### 1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos /metrics?

- Porque se manejan dos tipos de información, prometheus y /metrics solo entienden de números para saber cuándo ocurre un problema. Sin embargo, no almacenan texto. Loki es necesario para almacenar los logs, que son los que explican el porqué del problema.

#### 2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

- Aporta reproducibilidad, cualquiera puede levantar el entorno completo desde cero en otra máquina y Grafana ya sabrá exactamente dónde conectarse sin que nadie tenga que entrar a la interfaz a hacer clics, recordar contraseñas o escribir URLs manualmente.

#### 3. El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?

- Muestran valores distintos porque el "host" representa a la máquina completa, mientras que el "contenedor" está aislado y muestra únicamente el consumo específico de esa pieza de software.
Para alertar sobre una aplicación concreta se debe usar la CPU del contenedor. Si se usara la del host, la alarma podría dispararse porque otra aplicación consumió recursos, dando un falso positivo.

#### 4. ¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?


- Evaluation interval es la frecuencia con la que Grafana revisa la métrica y pending period es el tiempo de gracia que la condición debe mantenerse activa antes de enviar la alerta real. "la CPU debe estar por encima de 50% durante 30 segundos seguidos". Sirve para evitar que un pico de procesamiento de un solo segundo dispare falsas alarmas.