# SuperTCLDevDemo — Project Guide for Claude

Arduino sketch for the Cool Neon Total Control Lighting Developer Shield.
Single `.ino` file project using CoolNeon_DevShield, FastLED, and EEPROM
libraries.

## Build & Verify

```bash
arduino-cli compile --fqbn arduino:avr:uno SuperTCLDevDemo.ino
```

CI runs this on every PR via GitHub Actions.

## Arduino C++ Rules

### Functions MUST be declared before they are called

The Arduino build preprocessor generates function prototypes automatically for
the IDE, but `arduino-cli` and standard GCC compilation enforce strict
declaration-before-use ordering. Always place function **definitions** before
their first call site, or add explicit **forward declarations** near the top of
the file (after globals, before `setup()`).

```cpp
// Forward declarations — required before setup() if the function
// definition appears later in the file.
bool stableRead(int pin, int targetState, unsigned long debounceDelay = 20);
void responsiveDelay(unsigned long ms);
```

This applies to any new function added to the sketch. Place the definition
before its callers, or add a forward declaration.

### Other conventions

- **Constants at compile time**: use `#define` or `const PROGMEM` for lookup
  tables. Plain `const float`/`const int` scalars live in SRAM on AVR — prefer
  `#define` for scalars.
- **No dynamic allocation**: avoid `malloc`, `new`, `String`, or any
  heap-allocating type. All buffers are statically sized.
- **Delays**: use `responsiveDelay()` (not bare `delay()`) so `CheckSwitches()`
  is polled during waits. The one exception is the 20 ms debounce hold inside
  `stableRead()`.
- **Digital reads**: use `stableRead()` for debounced switch/button checks at
  state-transition points. Raw `digitalRead()` is fine in tight loops (e.g.
  `update_strand()`) where the 20 ms debounce penalty would be noticeable.
