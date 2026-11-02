# STM32 Embedded Temperature Control

A two-stage embedded project for the STM32F103RET6: an LCD settings interface followed by temperature acquisition and proportional PWM output using an MPU6050. Both development stages are collected in one repository while retaining their separate firmware snapshots.

## Repository contents

```text
Github-STM32/
|-- Report/
|   `-- STM32_Embedded_Temperature_Control_Report.pdf
|-- Code/
|   |-- 01_Menu_and_Settings/
|   |   |-- Core/Src/menu_and_settings.c
|   |   |-- Core/Inc/
|   |   |-- Core/Startup/
|   |   |-- Drivers/
|   |   `-- ... original CubeIDE configuration and linker script
|   |-- 02_Temperature_Control/
|   |   |-- Core/Src/temperature_control.c
|   |   `-- ... original supporting files and drivers
|   |-- Source_Manifest.csv
|   `-- README.md
|-- Results/
|   `-- .gitkeep
|-- Readme.tex
|-- README.md
`-- .gitignore
```

## Implemented design

- Stage 1: three-option LCD menu, button-driven setpoint and nonnegative offset editing, five-second inactivity return, flash storage of the setpoint, and timer-driven save/error LEDs.
- Stage 2: retained menu and setpoint storage, a UART text-reception screen, MPU6050 temperature conversion, and TIM3 channel 1 PWM on PA6.
- PWM logic: zero output for absolute error below 1 degree C; otherwise `min(999, floor(50 * absolute_error))`. With an auto-reload value of 999, the maximum compare value represents 99.9% duty. The editable offset is not used by this control law.

## Hardware and connections

| Function | Connection / setting |
|---|---|
| Target MCU | STM32F103RET6, LQFP64; 8 MHz HSI in application code |
| LCD control | RS: PC0; R/W: PC1; E: PC2 |
| LCD data | D4-D7: PC3-PC6 |
| Buttons | PB11: cycle/increase; PB12: enter/decrease/back, depending on screen |
| Save switches | PB13 and PB14, either edge triggers a save attempt |
| Indicators | PC8: save; PC9: offset underflow; PB15: nonzero control effort in stage 2 |
| MPU6050 I2C1 | PB6: SCL; PB7: SDA; 100 kHz; 7-bit address 0x68 |
| USART1 | PA9: TX; PA10: RX; 9600 baud, 8 data bits, no parity, 1 stop bit |
| Control PWM | PA6: TIM3_CH1; nominal 1 kHz |

The stage-2 file contains conflicting copies of GPIO initialization. The later copy configures PB13/PB14 and PC9; the earlier copy does not. Stage-2 inputs use no internal pull resistors, so the board must establish stable input levels. The pin map describes the intended application wiring, not a verified schematic.

## Source preservation and build status

All 178 retained source, configuration, and license files are byte-for-byte copies of the supplied files. Only the two application filenames and their enclosing stage directories have been renamed. `main.h`, `lcd.c`, `lcd.h`, startup files, driver names, and IDE metadata keep their original names to preserve dependencies. The SHA-256 manifest records every copied file. Generated `Debug` outputs have been omitted; their existing presence does not establish that the current source builds.

**The supplied stage-2 source is not build-ready.** It references `MenuState` and `MENU_*` constants without defining them, and defines `MX_I2C1_Init`, `MX_TIM1_Init`, `MX_TIM3_Init`, `MX_USART1_UART_Init`, and `MX_GPIO_Init` twice. These issues were preserved under the instruction not to change the code. No fresh firmware compilation or hardware test was performed during packaging.

Both original IDE project identities are `phase_one`. Inspect/import each snapshot in a separate STM32CubeIDE workspace. The supplied configuration records STM32CubeMX 6.15.0 and STM32Cube FW_F1 V1.8.6; generated build files identify GNU Tools for STM32 13.3.rel1. After import, confirm source discovery includes the renamed application file. The `.ioc` files are retained design snapshots and differ from handwritten application code; regenerating code can overwrite implementation details. The original initialization disables JTAG and SWD. Stage 2 must first have its documented source defects resolved in a separate working copy before a successful new build can be expected.

## Differences from the requested behavior

The specified UART protocol uses start byte `0xEE`, a two-byte `uint16_t` value, and end byte `0xAA`, with a ten-second start timeout and a `Wrong Packet` response. The supplied implementation instead collects up to 16 text characters, completes on CR/LF or buffer length, and displays the result. It does not update the setpoint or offset from UART.

Only the setpoint is persisted; offset starts at 2 after reset. The second stage replaces the first stage's timer-driven 500 ms save blink with blocking 200 ms intervals. Its Tune Temp screen returns home after five seconds without button activity. Button exit and timeout include PWM shutdown, but the save-switch path bypasses that cleanup and can leave the last PWM output active. Sensor I2C return values and flash-write return values are not validated. The report explains these limitations, the three-digit setpoint display buffer risk, and the lack of measured performance data. No PID controller is implemented.

## Report and demonstration

Read the [English technical report](Report/STM32_Embedded_Temperature_Control_Report.pdf). It includes the combined objectives, architecture, firmware behavior, equations, calculated examples, limitations, and a hardware verification plan.

`Results` is reserved for the real-hardware demonstration video. No video was present in the supplied project folder, so none is included. Add the recording here, for example as `STM32_Hardware_Demonstration.mp4`, and add its link to this section. That filename is a suggested future name, not an existing result. No measured settling time, temperature accuracy, or oscilloscope trace is claimed.


## Third-party material

The retained STM32 HAL and CMSIS files include their original notices and license files inside `Drivers`. No new repository-wide license has been assigned to the application code.
