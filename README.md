# Assistente vocale ESPHome per Home Assistant

Un assistente vocale hands-free basato su ESP32-S3, con rilevamento locale della
parola di attivazione e pipeline Assist elaborata da Home Assistant.

## Hardware

### Componenti

- ESP32-S3 DevKitC-1 con modulo **N8R8** (8 MB flash e 8 MB PSRAM octal)
- Microfono I²S **ICS-43434**
- Amplificatore I²S **MAX98357A**
- Altoparlante passivo da 4 o 8 Ω, almeno 3 W
- Alimentatore USB stabile da 5 V, almeno 1 A
- Cavetti e, facoltativamente, una basetta millefori o PCB

### Collegamenti

| Segnale | ESP32-S3 | ICS-43434 | MAX98357A |
|---|---|---|---|
| Clock bit (BCLK/SCK) | GPIO5 | SCK | BCLK |
| Clock parola (LRCLK/WS) | GPIO6 | WS | LRC |
| Dati microfono | GPIO4 | SD | — |
| Dati altoparlante | GPIO16 | — | DIN |
| Alimentazione microfono | 3V3 | VDD | — |
| Alimentazione amplificatore | 5V | — | VIN |
| Massa comune | GND | GND | GND |
| Selezione canale microfono | GND | L/R | — |
| Uscita audio | — | — | SPK+ e SPK− all'altoparlante |

Condividere BCLK, LRCLK e GND tra i due moduli. Il pin L/R del microfono a GND
seleziona il canale sinistro, configurato nel firmware. Non collegare SPK− a GND:
entrambi i terminali dell'altoparlante vanno alle uscite dell'amplificatore.
Verificare il pinout della propria scheda prima di alimentarla; usare una scheda
ESP32-S3 con PSRAM octal e non collegare il modulo amplificatore direttamente al
GPIO.

## Configurazione

1. Copiare `voice-assistant.yaml` nella cartella ESPHome di Home Assistant.
2. Nel `secrets.yaml` usato da ESPHome, definire `wifi_ssid`, `wifi_password`,
   `fallback_password` e `api_encryption_key`. Usare una password robusta per il
   punto di accesso di emergenza e una chiave API ESPHome codificata in Base64 da
   32 byte; ad esempio, generarla con `openssl rand -base64 32`. La stessa chiave
   protegge anche gli aggiornamenti OTA. Non condividere né versionare il file
   dei segreti.
3. Validare e compilare con `esphome config voice-assistant.yaml` e
   `esphome compile voice-assistant.yaml`, poi installare il firmware via USB.
4. In Home Assistant, aggiungere il dispositivo ESPHome rilevato e configurare
   una pipeline Assist con lingua italiana, un motore STT, un agente di
   conversazione e un motore TTS.
5. Pronunciare **“Okay Nabu”** e attendere il segnale di ascolto prima di parlare.

Il microfono e il rilevamento della parola di attivazione restano attivi sul
dispositivo; il riconoscimento del parlato e la risposta vengono forniti dalla
pipeline di Home Assistant. Il modello `okay_nabu` si attiva localmente; la lingua
della pipeline Assist va impostata separatamente in Home Assistant.
