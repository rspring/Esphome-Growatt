# Esphome-Growatt
Reading Growatt Inverter data into Home Assistant via the modbus using a MAX485 TTL to RS485 module and a Wemos D1 Mini. This setup completely eliminates the need to collect data via the cloud (the Chinese Growatt api servers). Also, it is possible to increase the update frequency of the data. My sensors are updating every 10 sec. The setup is also inexpensive: a few euro's for the D1 Mini, and the MAX458 to TTL module costs less than one euro!

## Links to other sources
In my example here I am using the standard [ESPHome Growatt](https://esphome.io/components/sensor/growatt_solar.html) integration. As far as I can see it only supports the solar production part, I don't see any sensors that relate to the use of batteries.

### Battery support
A [feature request](https://github.com/esphome/feature-requests/issues/2331) for battery support for the ESPHome Growatt component already exists. If the standard ESPHome Growatt integration does not work for you, I suggest you to read the following interesting discussion where an [alternative method for reading Growatt's modbus](https://community.home-assistant.io/t/esphome-modbus-growatt-shinewifi-s/369171/51) is proposed by reading values directly from their corrosponding registers. More info about the [Modbus Controller Component](https://esphome.io/components/modbus_controller.html) can be found at the ESPHome site.

<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/f4176c70-6b30-460e-a3fc-8b44422396bf" width="49%">
<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/2457af4b-bfa2-47d0-b440-6da30b7d5244" width="49%">

## Connecting to Growatt Inverter
You will need a **MAX485 TTL to RS485 Converter Module** and a **Wemos D1 mini** (or similar such as ESP32 or ESP2866) to read out the modbus of the Growatt inverter. And a few female jumper wires to make the connections. First you'll need to find out which exact two pins on the Growatt Inverter communication socket provide the RS485 modbus communication signals. In the quick guide of my Growatt MIN 6.000TL3-XH inverter I see that the _communication cable installation part_ shows that pin 3 (RS485A1, green in my setup) and pin 4 (RS485B1, green/white in my setup) can be used for RS485 communication.

![growatt manual](https://github.com/rspring/Esphome-Growatt/assets/6276750/915d86ba-ba97-40b2-9420-62bad633d7e0)

My inverter came with an empty connector plug including a few ferrules. I just needed a crimping tool to connect the two RS485 communication wires to the ferrules and assemble the connection plug. In the photo you can see that I also connected pin 1 (+12V) and pin 15 (ground). In a later stage I would like to replace the USB charger plug and somehow power the D1 mini directly from the converter.

<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/f2925684-c41a-434d-82b5-99c66e6f948a" width="49%">

<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/4a495ea1-2707-4bd1-b569-ee09657c77b1" width="49%">

<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/2fffe42d-2918-4e4f-a3f9-d716ba322246" width="49%">

<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/ef9ddeac-2b55-4d2f-bae7-e8b89fb250b0" width="49%">


## Connecting to MAX485 convertor module
These two wires from pin 3 (A) and pine 4 (B) must be connected to the two screws labeled A and B on the MAX485 TTL to RS485 Converter Module.

## Connecting the MAX485 convertor module to Wemos D1 Mini
The RS485 convertor module must be connected to the Wemos D1 Mini using five wires:
Two wires are used to power the module from the D1 mini:

- 3V3 on D1 mini to VCC on module (orange in my setup)
- GND on D1 mini to GND on module (grey in my setup)

and three wires to communicate with the module:

- TX (GPIO1) on D1 mini to DI on module (transmit, green in my setup)
- RX (GPIO3) on D1 mini to RO on module (recieve, yellow in my setup)
- D2 (GPIO4) on D1 mini to RE on module (flow control, orange in my setup)

![wiring](https://github.com/rspring/Esphome-Growatt/assets/6276750/31487c2e-daa0-4733-9dfb-43a3c70509a7)

<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/87d6426e-002a-4a0f-ae9b-995ba46e8681" width="49%">
<img src="https://github.com/rspring/Esphome-Growatt/assets/6276750/cfba1755-714e-444a-8ed0-c99e878d6ea8" width="49%">

## Adding new device in ESPHome
After installing ESPHome in Home Assistant it can be reached via the lefthand menu. Adding a new device is easy, just make sure the Wemos D1 Mini is connected via a USB _data_ cable (Be aware that some cheap USB charging cables don't support data communication). Follow the process of updating the first firmware to the D1 mini. When it is finished a new tile appears in the overview. This is the time to edit the code such that it starts listening and reading the Growatt Inverter:

## Uploading the Growatt code to the Wemos D1 Mini
Below is the exact code I use, and here is some explanation of the code first:
_esphome:_ This is just the name of the device.

_esp8266:_ This is automatically filled with the correct board type

_logger:_ As hardware serial is used, the logging via USB must be disabled. The serial logging uses the exact same pins we need to communicate with the MAX485TTL to RS485 module. Disabling logging is done by setting its bautrate to zero.

_api:_ The encryption key

_ota:_ Is just there such that updates can be done Over-The-Air.

_wifi:_ the wifi credentials the device need to connect with. (the actual credentils are read from my secrets.yaml file)

all the rest is just copied from: https://esphome.io/components/sensor/growatt_solar.html
```yaml
# Define the name of the ESPHome project
esphome:
  name: esphome-growatt
  friendly_name: Growatt

# Define the exact board being used
esp8266:
  board: esp01_1m

# Disable hardware serial logging as it conflicts with the serial connection
# with the MAX485 TTL converter module by setting baud rate to zero
logger:
  baud_rate: 0
  
# Enable Home Assistant API
api:
  encryption:
    key: "=heregoestheapikey="

# Enable over the air updates (ota)
ota:

# Define how the device connects to Wifi
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "esphome-growatt"
    password: "SSw2Oy8kDY3r"

# Define a captive portal as fallback in case connecting ti Wifi fails
captive_portal:
    
# Define the UART communication settings
# GPIO1 and GPIO3 are the hardware supported serial TX and RX pins
uart:
  - id: gw_uart
    baud_rate: 9600
    tx_pin: GPIO1
    rx_pin: GPIO3

# Define the modbus specific settings
modbus:
  uart_id: gw_uart
  flow_control_pin: GPIO4

# Define the protocol version and the sensors for this device.
# Old inverters use RTU (default). Newer ones use RTU2 (e.g. MIC, MIN, MAX series)
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
Now connect the thing to the Growatt Inverter and power the D1 mini from a usb charger.

![growatt](https://github.com/rspring/Esphome-Growatt/assets/6276750/85ea86f4-ff61-4082-aca1-b4bee8376fbb)

Back in Home Assistant's ESPHome page, the tile provides the log function:

![log](https://github.com/rspring/Esphome-Growatt/assets/6276750/b616bd28-6c85-4dc5-b73f-8ff885e9e7cc)

## Wow! A new ESPHome device is detected in the integrations panel
This is all the magic to setup the Wemos D1 Mini to listen via the RS458 to TTL module. In Home Assistant's settings -> integration page, the tile ESPhome will now detect the new 'Growatt' device, holding all the sensors:

![esphome](https://github.com/rspring/Esphome-Growatt/assets/6276750/d6d347d6-78ed-4b00-973b-49f3719210bf)
