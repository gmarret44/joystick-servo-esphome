# ESP32 Joystick Servo Controller

An ESP32-based project that reads an analog joystick (X/Y axes + push button) and drives a servo motor in real time based on the joystick's X axis, using [ESPHome](https://esphome.io/) and Home Assistant.

## Hardware

- Generic ESP32 board (`esp32dev`)
- 2-axis analog joystick module (VRX, VRY, SW)
- Servo motor (standard PWM, 50Hz)

### Wiring

| Component        | Pin    |
|-------------------|--------|
| Joystick VRX (X)  | GPIO34 |
| Joystick VRY (Y)  | GPIO35 |
| Joystick SW (button) | GPIO26 |
| Servo signal      | GPIO27 |
| Joystick VCC      | 3V3    |
| Joystick GND      | GND    |
| Servo VCC         | 5V (external supply recommended for larger servos) |
| Servo GND         | GND (common ground with the ESP32) |

> ⚠️ GPIO34 and GPIO35 are input-only (ADC1) pins on the ESP32 — they cannot be used as outputs, which is expected here since they only read the joystick's analog axes.
>
> ⚠️ If your servo draws more current than the ESP32's 3V3/5V regulator can supply (common with larger servos), power it from a separate 5V supply and tie its ground to the ESP32's ground.

## Features

- Reads the joystick's X axis (`GPIO34`) and maps it in real time to a servo position, from -100% to +100%
- Reads the joystick's Y axis (`GPIO35`) as a standalone sensor (not currently wired to an action — free to extend)
- Exposes the joystick's push button as a binary sensor (`Joystick Click`)
- Servo auto-detaches after each movement (`auto_detach_time: 1s`) to avoid overheating or buzzing when idle
- Local web server enabled on port 80 for quick debugging without Home Assistant

## Setup

1. Go to `esp32-firmware/`.
2. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your Wi-Fi credentials and a generated API encryption key.
3. Flash with ESPHome:
   ```bash
   esphome run joystick.yaml
   ```
4. The device will appear automatically in Home Assistant via the ESPHome integration, exposing the X/Y axis sensors and the button as entities.

## Calibration note

The X-axis to servo mapping assumes a joystick centered at ~1.65V (half of 3.3V) at rest:

```yaml
level: !lambda 'return (x - 1.65) / 1.65;'
```

If your joystick's resting voltage differs, read the raw `Joystick Axe X` sensor value at rest in Home Assistant and adjust `1.65` to match it.

## License

MIT — see [LICENSE](LICENSE).
