# Esphome-Growatt
Reading Growatt Inverter data via the modbus using an MAX485 TTL to RS485 module and a Wemos D1 Mini

![RS485 board](https://github.com/rspring/Esphome-Growatt/assets/6276750/f4176c70-6b30-460e-a3fc-8b44422396bf)

## Connecting to Growatt Inverter
You will need a **MAX485 TTL to RS485 Converter Module** and a **Wemos D1 mini** (or similar such as ESP32 or ESP2866) and a few female jumper wires. And you'll need to find out which two pins on the Growatt Inverter communication connector provide the RS485 modbus communication signals. In the quick guide of my Growatt MIN 6.000TL3-XH inverter I see that the _communication cable installation part_ indicates that pin 3 (green in my setup) and pin 4 (green/white in my setup) are used for RS485 communication: Pin 3 = RS485A1 and pin 4 = RS485B1.

![growatt manual](https://github.com/rspring/Esphome-Growatt/assets/6276750/915d86ba-ba97-40b2-9420-62bad633d7e0)

My inverter came with an empty connector plug including a few ferrule. I just needed a crimping tool to connect the two RS485 communication wires to the ferrule and assemble the connection plug. In the photo you see that I also connected pin 1 (+12V) and pin 15 (ground). In a later stage I would like to replace the USB charger plug and power the D1 mini directly from the converter.

![growatt_connector](https://github.com/rspring/Esphome-Growatt/assets/6276750/969e6089-d822-474e-8849-14d03518689c)

## Connecting to convertor module
These two wires need to be connected from pin 3 (A) to screw A and pine 4 (B) to screw B on the MAX485 TTL to RS485 Converter Module.

## Connecting convertor module to Wemos D1 Mini
The RS485 convertor module is connected with five wires to the Wemos D1 Mini:
Two wires are used to power the module from the D1 mini:

- 3V3 on D1 mini to VCC on module (orange in my setup)
- GND on D1 mini to GND on module (grey in my setup)

and three wires to communicate with the module:

- TX (GPIO1) on D1 mini to DI on module (transmit, green in my setup)
- RX (GPIO3) on D1 mini to RO on module (recieve, yellow in my setup)
- D2 (GPIO4) on D1 mini to RE on module (flow control, orange in my setup)

![d1mini-1](https://github.com/rspring/Esphome-Growatt/assets/6276750/87d6426e-002a-4a0f-ae9b-995ba46e8681)
![module connect](https://github.com/rspring/Esphome-Growatt/assets/6276750/cfba1755-714e-444a-8ed0-c99e878d6ea8)

## Adding new device in ESPHome
After installing ESPHome in Home Assistant it can be reached via the lefthand menu. Adding a new device is easy, just make sure the Wemos D1 Mini is connected via a true USB data cable as some charging cables do not support communication. Follow the process of updating the first firmware to the D1 mini. When it is finished a new tile appears in the overview. Now it is time to edit the code such that it starts listening to the Growatt Inverter:

## Uploading the Growatt code to the Wemos D1 Mini
Below is the exact code I use, and here is some explanation of the code first:

esphome: This is just the name of the device

esp8266: is automatically filled with the correct board type

logger: To disable logging via USB (it uses the same TX and RX pins) the baud_rate is set to 0

api: an encryption key

ota: Is just there such that updates can be done Over-The-Air

wifi: the wifi credentials the device need to connect with. (the actual credentils are read from my secrets.yaml file)

all the rest is just copied from: https://esphome.io/components/sensor/growatt_solar.html
```yaml
esphome:
  name: esphome-growatt
  friendly_name: Growatt

esp8266:
  board: esp01_1m

# Enable logging
logger:
  baud_rate: 0
 
# Enable Home Assistant API
api:
  encryption:
    key: "aZxPOKZhWrd5LkQMH1txjGSXd+vQZC7AU1nMq7TgJr4="

ota:


wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "esphome-growatt"
    password: "SSw2Oy8kDY3r"

captive_portal:
   

uart:
  - id: gw_uart
    baud_rate: 9600
    tx_pin: GPIO1
    rx_pin: GPIO3

modbus:
  uart_id: gw_uart
  flow_control_pin: GPIO4

sensor:
  - platform: growatt_solar
    update_interval: 10s
    protocol_version: RTU2

    inverter_status:
      name: "Status Code"

    phase_a:
      voltage:
          name: "Voltage Phase A"
      current:
          name: "Current Phase A"
      active_power:
          name: "Power Phase A"
    phase_b:
      voltage:
          name: "Voltage Phase B"
      current:
          name: "Current Phase B"
      active_power:
          name: "Power Phase B"
    phase_c:
      voltage:
          name: "Voltage Phase C"
      current:
          name: "Current Phase C"
      active_power:
          name: "Power Phase C"

    pv1:
      voltage:
          name: "PV1 Voltage"
      current:
          name: "PV1 Current"
      active_power:
          name: "PV1 Active Power"

    pv2:
      voltage:
          name: "PV2 Voltage"
      current:
          name: "PV2 Current"
      active_power:
          name: "PV2 Active Power"

    active_power:
      name: "Grid Active Power"

    pv_active_power:
      name: "PV Active Power"

    frequency:
      name: "Frequency"

    energy_production_day:
      name: "Today's Generation"

    total_energy_production:
      name: "Total Energy Production"

    inverter_module_temp:
      name: "Inverter Module Temp"
```

## Connect to Inverter and watch the logging in Home Assistant
Now connect the thing to the Growatt Inverter and power the D1 mini from a usb charger. Back in Home Assistant's ESPHome page, the tile provides the log function:

![log](https://github.com/rspring/Esphome-Growatt/assets/6276750/b616bd28-6c85-4dc5-b73f-8ff885e9e7cc)

## Wow! A new ESPHome device is detected in the integrations panel
This is all the magic to setup the Wemos D1 Mini to listen via the RS458 to TTL module. In Home Assistant's settings -> integration page, the tile ESPhome will now detect the new 'Growatt' device, holding all the sensors:

![esphome](https://github.com/rspring/Esphome-Growatt/assets/6276750/d6d347d6-78ed-4b00-973b-49f3719210bf)
