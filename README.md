# Proyecto Sensor UV – Monitoreo de Radiación Solar

Estudiante: Diana Sahian Quiñonez Yañez
No. de Control: 22211637

---


## 1️⃣ Propósito del Sistema

El propósito de este proyecto es:

- Capturar datos de radiación UV usando un sensor simulado en micro:bit.
- Transmitir los datos en tiempo real mediante el protocolo MQTT.
- Almacenar y persistir los datos en InfluxDB para análisis histórico.
- Visualizar y monitorear los datos con Grafana en tiempo real.
- Generar alertas automáticas si los niveles de radiación UV exceden un umbral seguro.

El sistema permite **monitoreo remoto**, análisis histórico y la posibilidad de **tomar decisiones basadas en datos** sobre la exposición al sol.

---

## 2️⃣ Arquitectura del Sistema

La arquitectura sigue un modelo **IoT típico basado en stack de telemetría**:

```
[Sensor UV (micro:bit simulado)] 
         │ publica datos via MQTT
         ▼
   [Broker MQTT (Mosquitto en EC2)]
         │ recibe y distribuye mensajes
         ▼
[Telegraf en EC2] --> lee mensajes MQTT
         │ transforma en formato JSON
         ▼
[InfluxDB 2.x en EC2] --> persiste datos en bucket "microbit_data"
         │ consulta histórica
         ▼
[Grafana] --> visualización de datos y alertas
`````
- **Sensor UV:** micro:bit simulado en VSCode genera datos de radiación.  
- **MQTT Broker:** Mosquitto recibe mensajes de los sensores y los envía a Telegraf.  
- **Telegraf:** Agente que consume MQTT y escribe los datos en InfluxDB.  
- **InfluxDB:** Base de datos de series de tiempo optimizada para datos IoT.  
- **Grafana:** Herramienta de visualización que crea dashboards y alertas.

---

## 3️⃣ Uso del Stack

**MQTT:**  
- Protocolo ligero de publicación/suscripción.  
- Permite transmitir datos del sensor a Telegraf de manera eficiente.  

**Telegraf:**  
- Agente que integra MQTT con InfluxDB.  
- Permite transformar datos JSON en métricas de series de tiempo.  

**InfluxDB:**  
- Base de datos de series de tiempo.  
- Almacena los valores de UV con marca de tiempo y metadatos del sensor.  

**Grafana:**  
- Dashboard de monitoreo.  
- Permite visualizar niveles de UV, históricos y generar alertas cuando se superan umbrales.

---

## 4️⃣ Instalación y Configuración

### MQTT Broker (Mosquitto)
```bash
sudo apt update
sudo apt install mosquitto mosquitto-clients
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
`````
### InfluxDB 2.x
```bash
sudo apt install influxdb
sudo systemctl enable influxdb
sudo systemctl start influxdb
`````
1. Crear bucket `microbit_data` y usuario/organization `iot-lab`.  
2. Generar token con permisos **write/read** para Telegraf y Grafana.

![INFLUXDB](https://github.com/DiSahian/Sensor-UV-para-alertas-de-radiacion-solar/blob/5067f23d1371f34ce745a080a8abd657548e4581/Captura%20de%20pantalla%202025-10-19%20201249.png)

### Telegraf
```bash
sudo apt install telegraf
```

- Configurar `telegraf.conf:`
```
[[inputs.mqtt_consumer]]
servers = ["tcp://100.110.166.3:1883"]
topics = ["lab/microbit/uv"]
qos = 1
data_format = "json"
name_override = "microbit_data"

[[outputs.influxdb_v2]]
urls = ["http://100.110.166.3:8086"]
token = "_JbcU2KhKRjcarRLMU1fGc3pTpaDEEyRGARlEuwHgqNOqtAcSHjhI8PbzK4WVQDIaUPt5fPXm5yAEPkqMAVkyw=="
organization = "iot-lab"
bucket = "microbit_data"
```
- Reiniciar Telegraf:
```
sudo systemctl restart telegraf
```

![TELEGRAF](https://github.com/DiSahian/Sensor-UV-para-alertas-de-radiacion-solar/blob/5067f23d1371f34ce745a080a8abd657548e4581/Captura%20de%20pantalla%202025-10-19%20201731.png)

### Grafana
```
sudo apt install grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```
- Acceder a http://100.110.166.3:3000.
- Agregar data source InfluxDB con bucket microbit_data.
- Crear dashboards y alertas.

![GRAFANA](https://github.com/DiSahian/Sensor-UV-para-alertas-de-radiacion-solar/blob/5067f23d1371f34ce745a080a8abd657548e4581/Captura%20de%20pantalla%202025-10-19%20195859.png)

## 5️⃣ Uso del Sistema

1. Ejecutar sensor en VSCode:
```
python sensor_uv.py
```

![SENSOR UV](https://github.com/DiSahian/Sensor-UV-para-alertas-de-radiacion-solar/blob/5067f23d1371f34ce745a080a8abd657548e4581/Captura%20de%20pantalla%202025-10-19%20201322.png)

2. Verificar suscriptor MQTT (opcional):
```
python suscriptor.py
```

![SUSCRIPTOR](https://github.com/DiSahian/Sensor-UV-para-alertas-de-radiacion-solar/blob/5067f23d1371f34ce745a080a8abd657548e4581/Captura%20de%20pantalla%202025-10-19%20201309.png)

3.Telegraf consume datos y los escribe en InfluxDB automáticamente.
4. Abrir Grafana:
- Visualizar datos en tiempo real en dashboards.
- Configurar alertas UV (ej. > 8).

## 6️⃣ Diagrama de Arquitectura

![DIAGRAMA DE LA ARQUITECTURA](https://github.com/DiSahian/Sensor-UV-para-alertas-de-radiacion-solar/blob/9fb817539f2e8d1f99e2b1073afb183f9bc6db66/arquitectura.png)
