# My Smart Home

## Global architecture

```
                                            +---------+
                                      +---> | NodeRed | <-+
+-----------+       +-------------+   |     +---------+   |   +----------+      +---------+
|  Devices  | <---> | MQTT Broker | <-+                   +-> | InfluxDB | +--> | Grafana |
+-----------+       +-------------+   |     +------+      |   +----------+      +---------+
                                      +---> | Apps | <----+
                                            +------+ 
```

## Directory structure

```
.
├── grafana -> JSON exports of dashboards
├── nodered -> JSON exports of flows
└── README.md
```

## Ambient sensing

### Xiaomi Mijia Bluetooth Thermometer 2 (LYWSD03MMC)

Installed in the different rooms to monitor temperature and humidity.

#### Flow

```
+------------+        +-----------------+
| LYWSD03MMC |   1    | OpenMQTTGateway |   2   +-------------+   3   +----------+    4   +---------+
|    ATC     | +--->  |     ESP32       | +---> | MQTT Broker | +---> | InfluxDB |  +---> | Grafana |
+------------+        +-----------------+       +-------------+       +----------+        +---------+
```

1. Using ATC custom firmware the thermometers send the data over Bluetooth advertisement packets
2. An ESP32 ([M5Atom](https://docs.m5stack.com/#/en/core/atom_matrix) here) running [OpenMQTTGateway](https://github.com/1technophile/OpenMQTTGateway) receives and publishes over MQTT the data
3. A Nodered flow subscribed to the topic `OpenMQTTGateway_ESP32_ATOM_BLE_IR/BTtoMQTT/+` receives and store the data in InfluxDB under the measurement `LYWSD03MMC_ATC`
4. Grafana dashboards "Ambient sensing" and "LYWSD03MMC" query the database to show graphs

#### How to flash ATC MiThermometer

Based on instructions from [ATC's Youtube video](https://youtu.be/NXKzFG61lNs)
1. Download [custom firmware binary](https://github.com/atc1441/ATC_MiThermometer/raw/master/ATC_Thermometer.bin)
2. Open [ATC's Telink Flasher](https://atc1441.github.io/TelinkFlasher.html) on Chrome Android App
3. Click "Connect" and find the device in the list
4. Click "Do Activation"
5. Select the firmware downloaded in step 1 using "Browse..."
6. Click "Start Flashing"

Additional optional config for a crude display
1. Connect to the device with the newly flashed firmware
2. Under "Smiley" click "Off"
3. Under "Show battery in LCD" click "Disabled"

#### Data storage

```
Schema
---------
LYWSD03MMC_ATC,id=[MAC address] tempc=[°C] hum=[%] batt=[%] volt=[V] timestamp
```
