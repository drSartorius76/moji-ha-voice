# Moji HA Voice

ESPHome voice satellite configuration for the **Movecall Moji ESP32-S3** with Home Assistant Assist.

The project was tested on the target device with:

- ESP32-S3, 16 MB flash, 8 MB PSRAM;
- ES8311 audio codec;
- `okay_nabu` micro wake word;
- Home Assistant Voice Assistant;
- Whisper STT, YandexGPT conversation and Piper TTS on the Home Assistant host.

## Hardware and pinout

| Function | GPIO |
|---|---:|
| ES8311 I2C SDA | 5 |
| ES8311 I2C SCL | 4 |
| I2S MCLK | 6 |
| I2S BCLK | 14 |
| I2S LRCLK/WS | 12 |
| Microphone DIN | 13 |
| Speaker DOUT | 11 |
| Amplifier enable | 9 |
| User button | 0 |
| Status WS2812 LED | 21 |
| ES8311 address | `0x18` |

The microphone path is an external analog ES8311 path. Keep `use_microphone: false`; enabling PDM mode produces unusable audio on this device.

## Important: power

Moji must be connected to a stable USB power source for continuous operation. The tested board does not expose a usable battery-level or charging-status input to this ESPHome configuration. An uptime reset after disappearance indicates a restart or power loss; it does not by itself prove a firmware hang.

## Local setup

Install ESPHome or use the official ESPHome Docker image. Copy the example secrets file:

```sh
cp secrets.yaml.example secrets.yaml
# edit secrets.yaml locally; never commit it
```

Compile and validate:

```sh
esphome config moji.yaml
esphome compile moji.yaml
```

For a low-memory Rock Pi build, run Ninja with one job. One tested Docker pattern is:

```sh
docker run --rm --name moji-build --entrypoint /bin/sh \
  -v esphome-cache:/root/.cache/esphome \
  -v "$PWD":/config esphome/esphome:latest \
  -c 'cp /usr/local/bin/ninja /tmp/ninja-real; \
      printf "%s\n" "#!/bin/sh" "exec /tmp/ninja-real -j1 \"\$@\"" > /tmp/ninja-wrap; \
      chmod +x /tmp/ninja-wrap; \
      PATH=/tmp/ninja-wrap:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin \
      esphome compile /config/moji.yaml'
```

## Upload

After a successful compile and while the device is already online:

```sh
esphome upload moji.yaml --device DEVICE_IP
```

Do not put real IP addresses, Wi-Fi credentials, API keys or OTA passwords into the repository.

## Home Assistant voice pipeline

The device uses the Home Assistant Assist pipeline. The tested server-side arrangement was:

- Whisper/faster-whisper for speech-to-text;
- YandexGPT through the Home Assistant conversation integration;
- Piper for text-to-speech;
- Home Assistant Container on a Linux host.

Configure these services in Home Assistant and select the resulting pipeline for the Moji satellite. The exact hostnames and credentials are deployment-specific and intentionally omitted here.

## Diagnostics

The configuration exposes:

- Wi-Fi signal;
- uptime;
- restart button;
- status LED;
- amplifier enable internally;
- user button for push-to-talk.

Wake word flow:

1. `okay_nabu` is continuously detected locally.
2. The wake-word detector stops.
3. Voice Assistant records and sends the request to Home Assistant.
4. On completion or error, wake-word detection starts again.

## Verification record

The tested final firmware compiled successfully and uploaded over OTA. After upload the device answered on the ESPHome API, reconnected to Wi-Fi, and completed a full Assist cycle (`listening` → `processing` → `responding` → `idle`).

## Safety and publication scope

This public repository intentionally excludes:

- Wi-Fi, Home Assistant and YandexGPT credentials;
- factory flash backups;
- private IP and MAC addresses;
- local logs and build output;
- unrelated files from the working directory.

## License

MIT. See [LICENSE](LICENSE).
