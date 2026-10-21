# ESPHome Component ESP32 RMT PWM

PWM output using the ESP-IDF RMT driver for ESP32 devices.

This component provides a continuous PWM output using an RMT TX channel in infinite loop mode. It exposes a standard ESPHome `output` platform and is suitable for driving loads like fans or LED drivers that expect a hardware PWM signal.

> **Important:** This component has been **tested on ESP32-C3 only**. The implementation uses the generic ESP-IDF RMT driver API and is expected to work on other ESP32 variants with RMT support, but those targets are currently **untested**.

```yaml
# Sample configuration entry example
external_components:
  - source: github://mikelawrence/esphome-components-esp32-rmt-pwm
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

## Configuration Variables

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

## Development with VS Code Dev Containers

Open the component checkout in VS Code and reopen it in the Dev Container. The setup uses Docker Compose with a persistent cache volume shared by all builds.

On Windows, run the VS Code task **Build ESPHome test configs** to compile every `.yaml` and `.yml` file under `test/` with the ESPHome `stable`, `beta`, and `dev` images. The script detects the GitHub owner/repository from the checkout's `origin` remote and replaces matching ESPHome GitHub component sources with the local `components/` directory in temporary copies of the test files. The original test YAML files are not modified.

Run the separate VS Code task **Validate ESPHome component** for strict YAML linting, C/C++ formatting, Ruff format/lint, flake8, pylint, pyupgrade, and ESPHome-style text convention checks. Tool versions are pinned to the versions used by ESPHome CI, and the existing `.yamllint`, `.clang-format`, `.flake8`, and `pyproject.toml` configurations are used. clang-tidy is not included because it requires ESPHome's PlatformIO compilation metadata, which is not present in a standalone component checkout.

Use **Fix ESPHome component formatting/lint** to apply clang-format, Ruff formatting and safe Ruff lint fixes, pyupgrade, and portable text fixes, then rerun validation. `yamllint`, flake8, pylint, and semantic or unsupported lint findings are not automatically rewritten and will be reported for manual attention.

The build script can also be run directly with `-ConfigDirectory`, `-ComponentsDirectory`, and `-Versions` to select ESPHome tags, or with `-Image` to use one custom image. If the checkout does not have a GitHub `origin`, pass `-ComponentSource owner/repository` explicitly.
