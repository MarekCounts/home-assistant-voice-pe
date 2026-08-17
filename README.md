# Home Assistant Voice: Preview Edition — streaming wake word

This fork builds Home Assistant Voice: Preview Edition firmware that streams microphone audio to Home Assistant for wake-word detection instead of running the normal wake phrases with microWakeWord on the ESP32-S3.

The small internal **stop** model remains local so a ringing timer or a long spoken response can still be interrupted. It is not offered as a Voice Assistant wake phrase. `okay nabu`, `hey jarvis`, custom openWakeWord models, and other normal wake phrases are selected and run in Home Assistant.

See [STREAMING_WAKE_WORD.md](STREAMING_WAKE_WORD.md) for the architecture, reproducible build command, flashing steps, Home Assistant configuration, and troubleshooting.

This work ports the approach demonstrated by [JLo's experimental configuration](https://gist.github.com/jlpouffier/41351187e2f6f94e797382a658702433) and [Rob Meades's firmware fork](https://github.com/RobMeades/home-assistant-voice-pe) onto ESPHome 2026.7.4 and the current Voice PE audio stack.
