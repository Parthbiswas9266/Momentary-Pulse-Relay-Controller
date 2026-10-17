# ESP32 Smart Pump Controller

An open-source smart pump controller built with **ESP32, ESPHome, and Home Assistant**.

The project brings remote pump control, electrical monitoring, configurable automatic shutoff, and scheduled operation into a single smart-home system. It is designed to work alongside an existing pump control panel by using momentary relay pulses to emulate its physical Start and Stop push buttons.

> **Project status:** Bench-tested with a sample load. Testing on the actual borewell installation is still pending.

## Features

- **Momentary Start/Stop control:** Uses separate relays to emulate the existing panel's push buttons.
- **Electrical monitoring:** Reads voltage, current, power, energy consumption, frequency, and power factor through a PZEM-004T energy meter.
- **Current-based running status:** Uses current thresholds to determine whether the pump is running, including changes made through the physical panel.
- **Configurable auto-off timer:** Set the operating duration from Home Assistant.
- **Daily schedule:** Configure scheduled start and stop times.
- **Independent controls:** Enable or disable the timer, schedule, and bypass mode separately.
- **Home Assistant integration:** Exposes controls and measurements through ESPHome.
- **ESP32-based design:** Built with commonly available hardware and configurable YAML.

## Hardware

| Component | Purpose |
|---|---|
| ESP32 development board | Main controller |
| Two-channel relay module | Emulates Start and Stop push buttons |
| PZEM-004T 100A energy meter | Electrical measurements |
| Existing pump control panel | Primary motor control and protection |
| Home Assistant server | Dashboard, configuration, and automation |

### ESP32 pin configuration

| GPIO | Function |
|---|---|
| GPIO25 | Start relay |
| GPIO26 | Stop relay |
| GPIO16 | UART RX for PZEM-004T |
| GPIO17 | UART TX for PZEM-004T |

The relay outputs are configured with inverted logic for the relay module used during development. Verify the relay's actual inactive state before connecting it to the pump panel.

## How it works

### 1. Pump control

Home Assistant sends a Start or Stop command to the ESP32. The controller energizes the corresponding relay for approximately one second and then releases it.

The relay's normally open dry contacts are intended to be connected in parallel with the corresponding physical push button, provided the existing panel's control circuit is compatible.

### 2. Electrical monitoring

The PZEM-004T communicates with the ESP32 over UART through the ESPHome Modbus integration.

The current-based running indicator uses hysteresis:

- Below 1 A: pump considered off.
- Above 6 A: pump considered on.
- Between 1 A and 6 A: retain the previous state.

These thresholds should be adjusted for the actual motor and its operating conditions.

### 3. Automatic shutoff

When enabled, the timer stops the pump after the configured duration. A timer-triggered lockout prevents the scheduled start routine from restarting the pump before the lockout is cleared.

Manual Start can clear the lockout and begin a new operating cycle.

### 4. Bypass mode

Bypass mode uses the controller's commanded state instead of the current-based running indicator. Because it ignores measured current, it cannot reliably reflect pump operations initiated directly at the physical panel while bypass mode is enabled.

## Software

- ESPHome
- Home Assistant
- ESP32 Arduino-compatible hardware is not required; this project uses the ESP-IDF framework.
- PZEM-004T communication through ESPHome's `pzemac` and Modbus components.

## Installation

1. Install ESPHome and add an ESP32 device.
2. Copy the project configuration into the device YAML file.
3. Configure Wi-Fi credentials using ESPHome secrets.
4. Verify GPIO assignments, relay logic, and PZEM wiring.
5. Compile and flash the firmware.
6. Add the ESPHome device to Home Assistant.
7. Test Start/Stop pulses and sensor readings with the motor isolated from hazardous conditions before connecting the controller to the actual installation.

Keep Wi-Fi credentials and API encryption keys out of public commits. Use `secrets.yaml` for private configuration values.

## Safety

**This project is experimental and is not a certified motor controller or safety system.**

- Mains voltage can cause fatal electric shock. Installation and modifications to mains wiring must be performed by a qualified person.
- Preserve the pump panel's original overload, dry-run, and other protective devices.
- Retain an independent physical Stop control.
- Verify relay isolation, contact ratings, and inactive behavior before connecting to the control panel.
- The software timer cannot guarantee shutoff if the ESP32 loses power, becomes unresponsive, or loses control of the relay.
- Do not use this project as the sole means of preventing flooding, dry running, or other hazardous conditions.

## Roadmap

- [ ] Test on the actual borewell installation
- [ ] Validate automatic shutoff and restart behavior
- [ ] Validate daily scheduling and timer priority
- [ ] Document the complete wiring diagram
- [ ] Add installation photos and a Home Assistant dashboard screenshot
- [ ] Publish a verified example configuration

## Contributing

Suggestions, bug reports, and improvements are welcome. Please include the ESPHome version, hardware configuration, and relevant logs when reporting issues.

## License

Released under the MIT License. See [LICENSE](LICENSE).
