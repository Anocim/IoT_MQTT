#IoT_MQTT

## 🚀 Guía de Instalación en Ubuntu Linux

### 1. Instalar y configurar el Broker (Eclipse Mosquitto)

```bash
# Actualizar repositorios e instalar Mosquitto y sus clientes CLI
sudo apt update
sudo apt install -y mosquitto mosquitto-clients

# Habilitar el puerto 1883 y permitir conexiones anónimas locales para la demo
echo -e "listener 1883\nallow_anonymous true" | sudo tee /etc/mosquitto/conf.d/demo.conf

# Reiniciar el servicio y comprobar que está activo
sudo systemctl restart mosquitto
sudo systemctl status mosquitto
```
*(Pulsa `q` para salir de la pantalla de estado).*

---

### 2. Instalar el Cliente Visual (MQTTX)



**Opción A — Vía paquete `.deb` oficial:**
Descarga el instalador para Linux (`amd64.deb`) desde [mqttx.app](https://mqttx.app/) y ejecuta:
```bash
sudo apt install ./MQTTX_*_amd64.deb
```

---

### 3. Instalar Node.js v22 LTS y Node-RED

Las versiones actuales de Node-RED requieren **Node.js v22.9 o superior** (la versión incluida por defecto en los repositorios estándar de Ubuntu suele quedar obsoleta).

```bash
# 1. Instalar curl y limpiar versiones antiguas de Node.js
sudo apt install -y curl
sudo apt remove -y nodejs npm
sudo apt autoremove -y

# 2. Añadir el repositorio oficial de NodeSource para Node.js 22 LTS e instalar
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verificar que la versión instalada es v22.x
node -v

# 3. Instalar Node-RED de forma global
sudo npm install -g --unsafe-perm node-red
```

---

## ▶️ Verificación y Demostración en Vivo

### Verificación rápida por terminal
Abre dos terminales para comprobar que Mosquitto enruta mensajes localmente:
* **Terminal 1 (Suscriptor):** `mosquitto_sub -h localhost -t "prueba/ubuntu"`
* **Terminal 2 (Publicador):** `mosquitto_pub -h localhost -t "prueba/ubuntu" -m "Broker operativo"`

### Configuración del Dashboard (Node-RED)
1. Arranca Node-RED ejecutando `node-red` en la terminal y abre `http://127.0.0.1:1880`.
2. Instala el módulo visual desde `Menú (☰) > Manage palette > Install` buscando **`@flowfuse/node-red-dashboard`**.
3. Crea dos flujos conectados a `localhost:1883`:
   * `mqtt in` (`fabrica/motor/temperatura`) $\rightarrow$ Nodo **Gauge** (Reloj).
   * `mqtt in` (`fabrica/motor/estado`) $\rightarrow$ Nodo **Text** (Indicador de estado).
4. Pulsa **Deploy** y abre el panel visual en `http://127.0.0.1:1880/dashboard`.

### Coreografía de la Demo (MQTTX)
1. Abre **MQTTX** y configura una conexión hacia `localhost:1883` llamada `Sensor_Motor`.
2. Antes de conectar, despliega **Last Will and Testament** y configura:
   * **Last-Will Topic:** `fabrica/motor/estado`
   * **Last-Will Payload:** `OFFLINE (Fallo Crítico)`
   * **Last-Will QoS:** `1` | **Last-Will Retain:** `True`
3. Pulsa **Connect** y ejecuta las pruebas:
   * **Telemetría:** Publica `45` en `fabrica/motor/temperatura` (QoS 0) para mover la aguja en Node-RED.
   * **Retained Message:** Publica `ONLINE` en `fabrica/motor/estado` marcando la casilla **Retain**. Abre una pestaña de incógnito en el dashboard para demostrar la persistencia inmediata de estado.
   * **Disparo de LWT:** Cierra forzadamente MQTTX (`killall mqttx` en terminal) sin desconectar limpiamente. Tras vencer el *Keep Alive*, el broker publicará automáticamente el estado `OFFLINE (Fallo Crítico)` en el dashboard.