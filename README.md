# Reflexometer KL05Z

A two-player reaction-time game for the NXP FRDM-KL05Z board (Kinetis KL05Z, Cortex-M0+). Players wait for the START signal and race to press their button first. The game is shown on a 16x2 character LCD and the start is signalled with a buzzer.

## How it works

A match is 3 rounds. In each round:

1. The LCD shows `Gotowi...` ("Get ready...") for 2 seconds.
2. After a random delay of 1-4 seconds, the display shows `START!` and the buzzer plays a short beep.
3. The first player to press their button wins the round, and their reaction time in milliseconds is shown.
4. Pressing your button **before** the START signal is a false start: the round is lost and the opponent gets the point.
5. If nobody reacts within 5 seconds, the round is void (`Brak reakcji`).
6. After every round the game waits for **S3** (next round) or **S4** (restart the whole game).

After 3 rounds the final score is shown, followed by the winner (or a draw). Pressing **S4** starts a new match.

## Controls

| Button | Pin | Function |
|--------|-----|----------|
| S1 | PTA9 | Player 1 reaction button |
| S2 | PTA10 | Player 2 reaction button |
| S3 | PTA11 | Continue to the next round |
| S4 | PTA12 | Restart the game |

Buttons are active-low with internal pull-ups enabled, and each read is debounced in software (20 ms).

## Hardware

- FRDM-KL05Z development board
- 16x2 character LCD with a PCF8574(A) I2C backpack (address `0x27` or `0x3F`)
- 4 push buttons (S1-S4)
- Buzzer / speaker

### Wiring

| Signal | Pin |
|--------|-----|
| LCD I2C SCL | PTB3 (I2C0) |
| LCD I2C SDA | PTB4 (I2C0) |
| Buzzer | PTB8 |
| Buttons S1-S4 | PTA9-PTA12 to GND |

## Software

- **Toolchain:** Keil uVision (Arm Compiler 6), project file `refleksomierz.uvprojx`, target `KL05Z`
- **Device pack:** NXP `MKL05Z32xxx4` (CMSIS Device Family Pack)
- **Debugger:** J-Link (settings in `JLinkSettings.ini`)

### Build and flash

1. Open `refleksomierz.uvprojx` in Keil uVision.
2. Install the `MKL05Z32xxx4` device pack if prompted.
3. Build the project (F7).
4. Connect the board and flash it (F8) or start a debug session (Ctrl+F5).

## Project structure

```
.
├── main.c           # game logic, SysTick timing, buzzer signal
├── buttons.c/.h     # button initialisation, debounced reads
├── lcd1602.c/.h     # HD44780 LCD driver over PCF8574 I2C expander
├── i2c.c/.h         # I2C0 master driver
├── frdm_bsp.h       # board definitions and helper macros
├── RTE/             # startup code, system init, CMSIS components
├── refleksomierz.uvprojx / .uvoptx   # Keil project files
└── JLinkSettings.ini                 # J-Link debugger settings
```

### Implementation notes

- **Timing:** SysTick generates a 1 ms tick (`timer_ms`), used for both delays and reaction time measurement.
- **Start beep:** a simple DDS-style square wave generated in software on PTB8 for 50 ms.
- **Random delay:** `rand()` picks the wait before START.

## Known limitations

- `rand()` is never seeded, so the sequence of random delays is identical after every reset.
- Button reads block for 20 ms while debouncing, which slightly affects the precision of the measured reaction time.
- The LCD messages are in Polish.
- Reaction time is measured in whole milliseconds (1 ms SysTick resolution).

## Repository notes

The archive currently contains build output (`Objects/`, `Listings/`) and a J-Link log (`JLinkLog.txt`). It's recommended to exclude these from version control with a `.gitignore`:

```
Objects/
Listings/
JLinkLog.txt
*.uvguix.*
```

## License

Add a license of your choice (e.g. MIT) in a `LICENSE` file.
