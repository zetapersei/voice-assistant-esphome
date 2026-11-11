# Assistente vocale ESPHome per Home Assistant

Assistente vocale hands-free basato su ESP32-S3. Il dispositivo rileva in locale
la parola di attivazione **“Okay Nabu”**; Home Assistant gestisce la pipeline
Assist per il riconoscimento del parlato, la conversazione e la sintesi vocale.

## Hardware

### Componenti e requisiti

- Scheda **ESP32-S3 DevKitC-1** con modulo **N8R8**: 8 MB di flash e 8 MB di
  PSRAM octal. La configurazione usa ESP-IDF e richiede PSRAM octal.
- Modulo microfono I²S **ICS-43434**. È alimentato a 3,3 V e fornisce l’audio
  digitale al pin dati dell’ESP32.
- Modulo amplificatore I²S **MAX98357A**, alimentato a 5 V. Riceve l’audio
  digitale dall’ESP32 e pilota direttamente l’altoparlante.
- Altoparlante passivo da **4 o 8 Ω**, con potenza nominale di almeno **3 W**.
- Alimentatore USB stabile da **5 V, almeno 1 A**; cavi e, facoltativamente,
  basetta millefori o PCB.

I moduli ICS-43434 e MAX98357A devono essere compatibili con i livelli logici
3,3 V dell’ESP32. Le sigle dei pin possono variare: verificare la serigrafia e
il datasheet dei moduli effettivamente acquistati prima di collegarli.

### Collegamenti

| Segnale | ESP32-S3 DevKitC-1 | ICS-43434 | MAX98357A |
|---|---|---|---|
| Clock bit (BCLK) | GPIO5 | SCK | BCLK |
| Clock parola (LRCLK/WS) | GPIO6 | WS | LRC |
| Dati del microfono | GPIO4 | SD | — |
| Dati verso l’altoparlante | GPIO16 | — | DIN |
| Alimentazione microfono | 3V3 | VDD | — |
| Alimentazione amplificatore | 5V | — | VIN |
| Massa comune | GND | GND | GND |
| Selezione canale microfono | GND | L/R | — |
| Uscita amplificatore | — | — | SPK+ e SPK− all’altoparlante |

BCLK, LRCLK e GND sono condivisi dai due moduli I²S. Collegare L/R del
microfono a GND seleziona il canale sinistro, corrispondente a `channel: left`
nella configurazione. Il microfono va alimentato a 3,3 V e l’amplificatore a
5 V; non alimentare il microfono a 5 V. Collegare i due terminali
dell’altoparlante esclusivamente a SPK+ e SPK−: l’uscita è a ponte e nessuno
dei due terminali va collegato a GND o a un GPIO.

Usare una massa comune tra scheda, microfono e amplificatore. Verificare il
pinout della propria scheda prima di alimentarla e non collegare l’uscita
dell’amplificatore direttamente all’ESP32. Per evitare riavvii o disturbi audio,
usare un’alimentazione stabile e cablaggi di massa e 5 V adeguati alla corrente
richiesta dall’amplificatore.

## Configurazione

1. Copiare `voice-assistant.yaml` nella cartella ESPHome di Home Assistant.
2. Nel `secrets.yaml` usato da ESPHome, definire `wifi_ssid`, `wifi_password`,
   `fallback_password` e `api_encryption_key`. Scegliere password robuste per il
   Wi‑Fi e il punto di accesso di emergenza. `api_encryption_key` deve essere
   una chiave API ESPHome codificata in Base64 da 32 byte; si può generare con
   `openssl rand -base64 32`. La chiave API protegge la connessione nativa con
   Home Assistant e gli aggiornamenti OTA secondo il flusso consigliato da
   ESPHome con crittografia. Non condividere né versionare il file dei segreti.
3. Validare e compilare con `esphome config voice-assistant.yaml` e
   `esphome compile voice-assistant.yaml`, poi installare il firmware via USB.
4. In Home Assistant, aggiungere il dispositivo ESPHome e configurare una
   pipeline Assist in italiano con motore STT, agente di conversazione e motore
   TTS.
5. Pronunciare **“Okay Nabu”** e attendere il segnale di ascolto prima di parlare.

La modalità Wi-Fi `power_save_mode: none` evita il risparmio energetico della
radio, riducendo la latenza durante l’uso audio a scapito di un maggiore consumo.
Il microfono e il rilevamento della parola di attivazione funzionano sul
dispositivo; il riconoscimento del parlato e la risposta sono forniti dalla
pipeline di Home Assistant. La lingua della pipeline Assist va impostata
separatamente in Home Assistant.
