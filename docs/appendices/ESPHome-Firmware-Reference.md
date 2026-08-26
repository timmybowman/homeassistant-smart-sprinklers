# Appendix A — ESPHome Firmware Reference

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** ESP32 Sprinkler Controller  
> **Firmware:** ESPHome  
> **Status:** Current configuration reference

## Overview

The sprinkler controller is based on an ESP32 running ESPHome and connected to Home Assistant through the native ESPHome API. The controller operates a nine-channel active-low relay board, with each relay exposed to Home Assistant as an individual sprinkler switch.

The current garden installation uses the first seven channels for the sprinkler system. Channels 8 and 9 are configured in the firmware and remain available for future expansion.

A hardware-level safety feature is included in every relay switch: once activated, a channel automatically turns itself off after **31 minutes**. This provides a safeguard against a valve remaining energised indefinitely if an automation or communication problem occurs.

## Controller Configuration

| Setting | Value |
|---|---|
| ESPHome device name | `esphome-web-fd4824` |
| Friendly name | `Sprinklers` |
| ESP32 variant | `esp32` |
| Framework | ESP-IDF |
| Minimum ESPHome version | `2026.4.0` |
| MAC suffix | Disabled |
| Home Assistant connection | Native ESPHome API |
| Updates | Over-the-air (OTA) |
| Wi-Fi power saving | `LIGHT` |

## Status LED

The ESP32's onboard blue LED is connected to **GPIO2** and is used as a simple activity indicator.

Whenever a sprinkler relay is switched on or off, the `flash_led_twice` script flashes the LED twice. Each flash consists of 100 milliseconds on followed by 100 milliseconds off.

### LED Activity

| Event | Behaviour |
|---|---|
| Sprinkler switched on | LED flashes twice |
| Sprinkler switched off | LED flashes twice |

## Relay Configuration

The relay board is **active-low**, meaning the GPIO output is inverted in ESPHome. Each relay is also configured with `restore_mode: ALWAYS_OFF`, ensuring that sprinkler channels default to the off position following a restart or reboot.

### GPIO and Relay Assignment

| ESP32 GPIO | ESPHome ID | Home Assistant Name | Current Use |
|---|---|---|---|
| GPIO13 | `zone_1` | Garden Sprinkler 1 | Garden Zone 1 |
| GPIO14 | `zone_2` | Garden Sprinkler 2 | Garden Zone 2 |
| GPIO16 | `zone_3` | Garden Sprinkler 3 | Garden Zone 3 |
| GPIO17 | `zone_4` | Garden Sprinkler 4 | Garden Zone 4 |
| GPIO18 | `zone_5` | Garden Sprinkler 5 | Garden Zone 5 |
| GPIO19 | `zone_6` | Garden Sprinkler 6 | Garden Zone 6 |
| GPIO21 | `zone_7` | Garden Sprinkler 7 | Garden Zone 7 |
| GPIO22 | `zone_8` | Garden Sprinkler 8 | Available for expansion |
| GPIO23 | `zone_9` | Garden Sprinkler 9 | Available for expansion |
| GPIO2 | `onboard_led` | Onboard LED | Status indicator |

## Safety Timeout

Every relay has an independent **31-minute automatic shutdown**.

The timeout starts when the relay is turned on:

1. The status LED flashes twice.
2. The relay remains active.
3. After 31 minutes, ESPHome automatically turns that relay off.
4. The status LED flashes twice again as part of the normal switch-off behaviour.

> **Important:** Home Assistant's normal cycle-and-soak automation should always use runtimes below this limit. The firmware timeout is a safety backstop rather than the intended method of ending a watering cycle.

## Complete ESPHome Firmware

The following is the complete sprinkler controller configuration and can be used as a backup when rebuilding the ESP32 controller.

```yaml
esphome:
  name: esphome-web-fd4824
  friendly_name: Sprinklers
  min_version: 2026.4.0
  name_add_mac_suffix: false

esp32:
  variant: esp32
  framework:
    type: esp-idf

logger:

api:

ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: LIGHT

output:
  - platform: gpio
    pin: GPIO2
    id: onboard_led

script:
  - id: flash_led_twice
    mode: restart
    then:
      - output.turn_on: onboard_led
      - delay: 100ms
      - output.turn_off: onboard_led
      - delay: 100ms
      - output.turn_on: onboard_led
      - delay: 100ms
      - output.turn_off: onboard_led

# Control for 9 channels on the active-low relay board
switch:
  - platform: gpio
    name: "Garden Sprinkler 1"
    id: zone_1
    pin:
      number: GPIO13
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_1
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 2"
    id: zone_2
    pin:
      number: GPIO14
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_2
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 3"
    id: zone_3
    pin:
      number: GPIO16
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_3
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 4"
    id: zone_4
    pin:
      number: GPIO17
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_4
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 5"
    id: zone_5
    pin:
      number: GPIO18
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_5
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 6"
    id: zone_6
    pin:
      number: GPIO19
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_6
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 7"
    id: zone_7
    pin:
      number: GPIO21
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_7
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 8"
    id: zone_8
    pin:
      number: GPIO22
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_8
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 9"
    id: zone_9
    pin:
      number: GPIO23
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_9
    on_turn_off:
      - script.execute: flash_led_twice
