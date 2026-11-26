# Automated Greenhouse

Temperature controller for a small greenhouse, running on an ESP32. It reads the temperature with a DHT22, shows it on a 2.8" TFT display with a three-button menu where you set the target temperature, and drives a stepper motor as the actuator, with an LED that lights up while the motor is on.

I built it in January 2025 as the electronics case of the CPE Jr. trainee program, written in C++ with PlatformIO and the Arduino framework.

## How it works

The display has three screens:

- Main menu, with "Definir temp. ideal" (set the target) and "Temp. em tempo real" (live temperature). Up/down move the cursor and select opens the screen.
- Target temperature: up/down change the target by 1 °C and select goes back. The default target is 25 °C.
- Live temperature: shows the current reading and select goes back. The number is only redrawn when the value changes, to avoid flicker.

Every loop the ESP32 reads the sensor twice, half a second apart, updates the display and steps the motor.

The motor control is meant to be proportional: the delay between steps is mapped linearly from the difference between the current and the target temperature (2 ms to 10 ms over a 20 °C range), and the motor only runs when the greenhouse is above the target. That function, `Motor::update_delay`, is written but still commented out, so right now the motor runs at a fixed step delay of 5 ms.

## Hardware

- ESP32 dev board (uPesy ESP32 WROOM)
- DHT22 temperature sensor
- ILI9341 2.8" TFT display (SPI)
- Stepper motor with a STEP/DIR driver
- LED
- 3 push buttons (to GND, using the internal pull-ups)

| Component | Pin | GPIO |
|---|---|---|
| DHT22 | data | 25 |
| Stepper driver | STEP | 21 |
| Stepper driver | DIR | 19 |
| LED | + | 33 |
| Display | CS | 5 |
| Display | DC | 32 |
| Display | SCK / MOSI | 18 / 23 (default SPI) |
| Button | select | 2 |
| Button | up | 15 |
| Button | down | 4 |

## Code structure

```
platformio.ini          board, framework and libraries
src/main.cpp            pin map, setup and main loop
src/Classes/Display.*   menu screens, button handling and drawing on the TFT
src/Classes/Motor.*     stepper control and the (disabled) proportional delay
```

The display and the motor are separate classes so the main loop only reads the sensor and passes values between them.

## Building

With [PlatformIO](https://platformio.org/) installed (CLI or the VS Code extension):

```bash
pio run --target upload
pio device monitor -b 115200
```

The libraries in `platformio.ini` (Adafruit ILI9341 and the DHT sensor libraries) are installed on the first build.

## What I'd do next

- Turn on the proportional control and check that the speed goes the right way (faster when it's hotter).
- Debounce the buttons; holding one down currently changes the value on every loop.
- Use the motor to drive something real, such as a fan or a vent, and add humidity, which the DHT22 already measures.
