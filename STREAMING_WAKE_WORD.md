# Streaming wake-word firmware

This firmware keeps the Voice PE microphone connected to Home Assistant while the device is unmuted. Home Assistant runs the wake-word stage, then continues through speech-to-text, intent handling, and text-to-speech on the selected Assist pipeline.

## What changed

- `voice_assistant.use_wake_word` is enabled and the client starts with `voice_assistant.start_continuous`.
- The normal on-device `okay nabu`, `hey jarvis`, and `hey mycroft` models and their sensitivity selector are removed.
- Wake feedback, LED state changes, and media ducking begin in `on_listening`, after Home Assistant reports a detected wake word. They do not begin when the continuous stream starts.
- Muting stops the stream; unmuting and completed interactions resume it.
- The center button distinguishes an active interaction from an idle continuous stream.
- The local internal `stop` model remains available only while a timer or a long response is playing.
- Stock Home Assistant Voice PE HTTP update manifests are removed from the factory image. An official update would otherwise replace this custom firmware with the stock on-device wake-word configuration. ESPHome OTA remains enabled.
- The ESPHome toolchain and the external Voice PE component source are pinned for repeatable builds.

The older fork's vendored ESPHome audio-reader timeout patch is intentionally not included. ESPHome's 2026.6 audio and media-player stack has since been replaced, and the current hardware speaker already uses `timeout: never`.

## Build

From this repository directory, run:

```bash
docker run --rm \
  -v "$PWD:/config" \
  -w /config \
  ghcr.io/esphome/esphome:2026.6.0 \
  compile home-assistant-voice.factory.yaml
```

The two useful outputs are:

- Initial USB install: `.esphome/build/home-assistant-voice/.pioenvs/home-assistant-voice/firmware.factory.bin`
- Later ESPHome OTA install: `.esphome/build/home-assistant-voice/.pioenvs/home-assistant-voice/firmware.ota.bin`

The normal Voice PE has 16 MB flash and should use `home-assistant-voice.factory.yaml`. `home-assistant-voice.8mb.yaml` is retained only for the less common 8 MB hardware variant.

## Configure Home Assistant

Do this before testing the wake phrase:

1. Configure an Assist pipeline, using Home Assistant Cloud or local speech-to-text and text-to-speech providers.
2. Install and start the **openWakeWord** app, or connect another Wyoming wake-word provider.
3. Go to **Settings > Voice assistants** and create or edit an assistant.
4. From the assistant's three-dot menu, select **Add streaming wake word**. Select the wake-word engine and phrase; `ok nabu` is a useful first test.
5. Assign that assistant to the Voice PE when Home Assistant prompts during setup, or from the device's voice-assistant configuration.

Home Assistant's current walkthrough is [Enabling a wake word](https://www.home-assistant.io/voice_control/install_wake_word_add_on/). A fully local STT/TTS setup is covered in [Set up a fully local voice assistant](https://www.home-assistant.io/voice_control/voice_remote_local_assistant/).

## Flash and provision

1. Connect the Voice PE to a computer over USB-C.
2. Open [ESPHome Web](https://web.esphome.io/) in a WebSerial-capable browser.
3. Select **Connect**, choose the Voice PE serial port, then select **Install**.
4. Choose `firmware.factory.bin` from the build output above.
5. Provision Wi-Fi when prompted. The factory build includes Improv over serial and Bluetooth.
6. Add the discovered ESPHome device in Home Assistant and assign the streaming-wake-word assistant.

Flashing custom firmware is reversible: the official Voice PE installer can restore stock firmware. Treat the generated binary as experimental until it has passed the runtime checks below on your hardware.

## Runtime checks

After flashing, verify each of these:

- With the mute switch off, the device connects to Home Assistant and remains in its idle LED state while streaming.
- The selected server-side phrase triggers the wake sound and listening LED state.
- A full command completes, audio is no longer ducked afterward, and the next wake phrase works without rebooting.
- The hardware and Home Assistant mute switches stop detection; unmuting resumes it.
- Music continues at normal volume after a separate announcement finishes.
- A timer can be cancelled with the local stop phrase while ringing.
- A long TTS response can be interrupted with the local stop phrase, after which the server-side wake phrase works again.
- Disconnecting and reconnecting Home Assistant restores continuous listening.

## Troubleshooting

- **`wake-provider-missing` or `wake-engine-missing`**: the assigned assistant does not have a working streaming wake-word engine. Recheck openWakeWord/Wyoming and the assistant's streaming wake-word selection.
- **Wake phrase never triggers**: confirm the device is assigned to the assistant you edited and that it is not muted. Start with the built-in `ok nabu` server model before trying a custom model.
- **Wake works only once**: inspect ESPHome logs for a disconnect or error after `on_end`; continuous mode should restart whenever the device is unmuted.
- **Audio stays quiet after an interaction**: inspect whether `voice_assistant_phase` returned to idle and whether an announcement or timer is still active.
- **Need stock behavior again**: reinstall the official Voice PE firmware and re-adopt the device.

## Privacy and network behavior

While unmuted and connected, microphone audio is continuously sent over the local network to the assigned Home Assistant instance so the server can detect the wake phrase. The hardware mute switch stops the voice-assistant stream. This is materially different from stock firmware, where normal wake phrases are detected on the device and audio is sent only after detection.
