# ESPHome voice assistant for Home Assistant

A hands-free voice assistant based on the ESP32-S3. The device detects the
**“Okay Nabu”** wake word locally; Home Assistant runs the Assist pipeline for
speech recognition, conversation, and speech synthesis.

## Hardware

### Components and requirements

- **ESP32-S3 DevKitC-1** board with an **N8R8** module: 8 MB flash and 8 MB
  octal PSRAM. This configuration uses ESP-IDF and requires octal PSRAM.
- **ICS-43434** I²S microphone module. It runs on 3.3 V and sends digital audio
  to the ESP32 data pin.
- **MAX98357A** I²S amplifier module, powered from 5 V. It receives digital
  audio from the ESP32 and directly drives the speaker.
- **4 or 8 Ω** passive speaker rated for at least **3 W**.
- Stable **5 V, at least 1 A** USB supply, hookup wires, and optionally a
  perfboard or PCB.

The ICS-43434 and MAX98357A modules must support the ESP32's 3.3 V logic
levels. Pin labels may vary; check the markings and datasheets for the actual
modules before wiring them.

The KiCad design files are in [`voiceass-kicad/`](voiceass-kicad/):
[`voiceass-kicad.kicad_sch`](voiceass-kicad/voiceass-kicad.kicad_sch) is the
schematic and [`voiceass-kicad.kicad_pcb`](voiceass-kicad/voiceass-kicad.kicad_pcb)
is the PCB layout.

### Wiring

| Signal | ESP32-S3 DevKitC-1 | ICS-43434 | MAX98357A |
|---|---|---|---|
| Bit clock (BCLK) | GPIO5 | SCK | BCLK |
| Word clock (LRCLK/WS) | GPIO6 | WS | LRC |
| Microphone data | GPIO4 | SD | — |
| Speaker data | GPIO16 | — | DIN |
| Microphone power | 3V3 | VDD | — |
| Amplifier power | 5V | — | VIN |
| Common ground | GND | GND | GND |
| Microphone channel select | GND | L/R | — |
| Amplifier enable/channel select | 3V3 | — | SD_MODE |
| Amplifier gain setting | 5V | — | GAIN_SLOT |
| Amplifier output | — | — | SPK+ and SPK− to speaker |

BCLK, LRCLK, and GND are shared by both I²S modules. Connecting the
microphone's L/R pin to GND selects the left channel, matching `channel: left`
in the configuration. Power the microphone from 3.3 V and the amplifier from
5 V; do not power the microphone from 5 V. Connect the speaker's two
terminals only to SPK+ and SPK−: this is a bridge-tied output, so neither
speaker terminal should be connected to GND or an ESP32 GPIO.

Connect `SD_MODE` to 3V3 to enable the amplifier and select the left audio
channel; pulling it low shuts the amplifier down. Connect `GAIN_SLOT` to 5V
for 6 dB gain. In the current KiCad files, `GAIN_SLOT` is connected to +5V,
but `SD_MODE` is left unconnected, so connect it to 3V3 when assembling the
board.

Use a common ground for the board, microphone, and amplifier. Check the
development board pinout before applying power, and never connect the
amplifier output directly to the ESP32. Use a stable supply and wiring sized
for the amplifier's 5 V current draw to help prevent resets and audio noise.

## Configuration

1. Copy `voice-assistant.yaml` to the ESPHome configuration directory used by
   Home Assistant.
2. In ESPHome's `secrets.yaml`, define `wifi_ssid`, `wifi_password`,
   `fallback_password`, and `api_encryption_key`. Choose strong passwords for
   Wi-Fi and the fallback access point. `api_encryption_key` must be a
   Base64-encoded, 32-byte ESPHome API key; you can generate one with
   `openssl rand -base64 32`. The API key protects both the native Home
   Assistant connection and OTA updates via ESPHome's recommended encrypted OTA
   flow. Do not share or commit the secrets file.
3. Validate and compile with `esphome config voice-assistant.yaml` and
   `esphome compile voice-assistant.yaml`, then install the firmware over USB.
4. Add the discovered ESPHome device in Home Assistant. Configure an Assist
   pipeline in Italian with an STT engine, conversation agent, and TTS engine.
5. Say **“Okay Nabu”** and wait for the listening cue before speaking.

Wi-Fi `power_save_mode: none` disables radio power saving, reducing latency
during audio use at the cost of higher power consumption. The microphone and
wake-word detection run on the device; speech recognition and responses are
handled by Home Assistant's pipeline. Set the Assist pipeline language
separately in Home Assistant.
