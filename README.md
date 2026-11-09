# ESPHome Component ESP32 RMT PWM

PWM output using the ESP-IDF RMT driver for ESP32 devices.

This component provides a continuous PWM output using an RMT TX channel in infinite loop mode. It exposes a standard ESPHome `output` platform and is suitable for driving loads like fans or LED drivers that expect a hardware PWM signal.

> **Important:** This component has been **tested on ESP32-C3 only**. The implementation uses the generic ESP-IDF RMT driver API and is expected to work on other ESP32 variants with RMT support, but those targets are currently **untested**.

```yaml
# Sample configuration entry example
external_components:
  - source: github://mikelawrence/esphome-component-esp32-rmt-pwm
    components:
      - esp32_rmt_pwm

output:
  - platform: esp32_rmt_pwm
    id: fan_pwm_output
    pin: GPIO4
    frequency: 25kHz
    rmt_symbols: 96

fan:
  - platform: speed
    name: "RMT PWM Fan"
    output: fan_pwm_output
    speed_count: 3
```

## Output

The `esp32_rmt_pwm` output can be used by any component that accepts a float output.

```yaml
output:
  - platform: esp32_rmt_pwm
    id: fan_pwm_output
    pin: GPIO4
    frequency: 25kHz
```

### Configuration Variables

* **id** (*Required*, ID)
  The ID of the output component. This is used to reference the output from other ESPHome components (lights, fans, custom components, etc.).

* **pin** (*Required*, [Pin Schema](https://esphome.io/guides/configuration-types#config-pin-schema))
  The GPIO pin used for the PWM output signal.

* **frequency** (*Optional*, frequency, default: `25kHz`)
  The PWM frequency.
  Internally, the component:

  * Uses an RMT resolution of 10 MHz (100 ns per tick).
  * Computes the PWM period in ticks as:
    * `period_ticks = resolution_hz / frequency`
  * Validates that:
    * `period_ticks >= 2` (to avoid absurdly high frequencies)
    * `period_ticks <= 32767` (to fit the 15-bit RMT symbol duration field)

  If the requested frequency falls outside these bounds, the component logs an error and marks itself as failed during setup rather than generating an invalid PWM signal.

* **rmt_symbols** (*Optional*, integer, variant-specific default, minimum: `48`)

  Number of RMT symbols allocated for the TX channel (`mem_block_symbols`).

  This value controls how much RMT memory is reserved for this channel. The component itself uses a single symbol for the PWM period, but the allocation size still affects resource usage and plays more of a role if the implementation is later extended. You should override the default if you need to optimize RMT memory usage or are running multiple RMT-based components.

  Defaults to `96` on ESP32-C3, C5, C6 and H2, and `192` on ESP32, P4, S2 and S3.

* All other [float output options](https://esphome.io/components/output/#base-output-configuration), including `inverted`, `power_supply`, `min_power`, `max_power` and `zero_means_zero`.

## Development with VS Code Dev Containers

Open the component folder in VS Code and select **Reopen in Container** for development. Docker Desktop is required. The tasks below run from the Windows host workspace using **Terminal > Run Task**:

- **ML ESPHome: Build ESPHome test configs**: The default build task. Validates first, then compiles the test configurations with ESPHome stable, beta, and dev using the local component source.
- **ML ESPHome: Validate ESPHome component**: Applies supported automatic fixes and checks the component without building test configurations.
- **ML ESPHome: Clear cached downloads**: Clears shared ESPHome, PlatformIO, and pip caches and removes downloaded ESPHome Docker images. Close the Dev Container before running it.

Use **Terminal > Run Build Task** to run the default build task. Test configuration builds serve as integration tests; there is no separate unit-test runner.
