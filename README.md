# SNES Mapper

A browser-based tool for mapping a USB SNES controller to your JavaScript game. Press buttons, click to assign, export the mapping as drop-in code.

https://seewhatimade.github.io/SNES-Mapper

## Why?

Browsers expose gamepads through the [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API), but button indices are wildly inconsistent across browsers, operating systems, and USB adapters. The same SNES controller can report its A button as `buttons[0]` on Chrome/Mac and `buttons[1]` on Firefox/Windows. Hardcoding indices is a recipe for "works on my machine."

Rather than guess, plug your controller in, map it visually in about thirty seconds, and paste the generated code into your game.

## Features

- Visual SNES controller diagram with click-to-assign workflow
- Live input monitor — see exactly which button indices fire as you press them
- Auto-advances to the next unmapped slot after each assignment
- One-click copy of the generated code
- Zero dependencies, single self-contained HTML file
- Generated helpers include both `isPressed()` (held state) and `justPressed()` (one-shot edge detection)

## Usage

### Quick start

Open `gamepad-mapper.html` in any modern browser. That's it — no build step, no install.

If you want to serve it locally instead of opening the file directly:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/gamepad-mapper.html
```

### Mapping workflow

1. Plug in your USB SNES controller.
2. Press any button on it — browsers refuse to expose gamepad data until they receive a user gesture from the device.
3. Click any button on the diagram (or in the side list) to select it. The slot pulses red.
4. Press the matching physical button. It's assigned, and the selector auto-advances to the next unmapped slot.
5. Repeat until all twelve inputs are mapped.
6. Click **Generate Code** and copy the result into your project.

## Generated code

The exported module is plain JavaScript with no dependencies. It looks like this (your indices will differ):

```javascript
const CONTROLLER_MAP = {
  A: 1,
  B: 0,
  X: 3,
  Y: 2,
  L: 4,
  R: 5,
  DPAD_UP: 12,
  DPAD_DOWN: 13,
  DPAD_LEFT: 14,
  DPAD_RIGHT: 15,
  SELECT: 8,
  START: 9
};

function getGamepad() { /* ... */ }
function isPressed(name) { /* ... */ }
function justPressed(name) { /* ... */ }
```

Drop it into your game loop:

```javascript
function update() {
  if (justPressed('A'))        jump();
  if (isPressed('B'))          attack();
  if (isPressed('DPAD_LEFT'))  player.x -= speed;
  if (isPressed('DPAD_RIGHT')) player.x += speed;
  if (justPressed('START'))    togglePause();

  requestAnimationFrame(update);
}
requestAnimationFrame(update);
```

`isPressed()` returns `true` every frame the button is held. `justPressed()` returns `true` only on the frame the press began — use it for jumps, menu confirms, or anything else you don't want auto-repeating.

## Browser support

Works anywhere the [Gamepad API](https://caniuse.com/gamepad) is supported:

- Chrome / Edge — full support
- Firefox — full support
- Safari — supported, but stricter about user-gesture requirements

## Known quirks with USB SNES adapters

A few things to watch for, because cheap USB-to-SNES adapters do not behave consistently:

### D-pad reported as axes instead of buttons

Some adapters expose the d-pad on the analog axes rather than as discrete buttons:

- `axes[0]`: `-1` = left, `+1` = right
- `axes[1]`: `-1` = up, `+1` = down

If you press d-pad directions and nothing appears under "Buttons pressed" in the live monitor, this is what's happening. The current mapper handles buttons only — until axis support is added, you'll need to read those axes manually in your game code.

### Printed letter does not match button index

Browsers usually label buttons by *position* (Xbox layout) rather than the letter printed on the controller. The button labeled "A" on your SNES controller may come through as `buttons[1]` because that's the position Xbox calls "A".

This does not matter for the mapper. When the "A" slot is selected, press whichever button your controller has labeled A — the correct index gets stored under the name `A`, and your game code reads `isPressed('A')` regardless of underlying index.

## Project layout

```
.
├── index.html    # the entire tool, single self-contained file
└── README.md
```

## License

MIT.
