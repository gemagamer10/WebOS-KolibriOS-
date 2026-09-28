# KolibriOS WebOS

x86 emulator running inside a webOS TV app. Boots [KolibriOS](https://kolibrios.org/) — an open-source OS written entirely in assembly, with a full GUI, occupying ~1.4 MB and running on 16 MB of RAM.

Tested on webOS 7.6.0.

Download the `.ipk` from GitHub and install using `ares-install`.

## How to Use

1. Download the pre-built `.ipk` from this repository
2. Enable **Developer Mode** on your TV and connect via the webOS CLI
3. Install the app:

```bash
ares-install --device <device> com.kolibri.webos_1.0.0_all.ipk
```

4. Launch from the TV menu or via CLI:

```bash
ares-launch --device <device> com.kolibri.webos
```

The app opens KolibriOS full-screen on your TV.

## Features

- **Full KolibriOS** with GUI, mouse, keyboard, text editor, image viewer, games
- **No network dependency** — the emulator and OS are embedded inside the `.ipk`
- **Graphical interface** at 640×480 (any color depth)
- **Small package** — final `.ipk` is ~4.9 MB

## Technical Details

| Parameter | Value |
|---|---|
| Emulator | [v86](https://github.com/copy/v86) (x86 → WebAssembly) |
| Guest OS | KolibriOS (floppy image, 1.4 MB) |
| `memory_size` | 16 MB |
| `vga_memory_size` | 300 KB |
| `fda` | `kolibri.img` |
| `.ipk` size | ~4.9 MB |

## Project Structure

```
.
├── appinfo.json          # webOS app manifest
├── icon.png              # 80x80 icon
├── index.html            # instantiates the V86Starter
├── lib/
│   ├── v86-bundle.js     # v86 bundle (IIFE, no ES imports)
│   └── v86-wasm-b64.js   # v86.wasm as base64 (bypasses file:// CORS)
└── vm/
    └── kolibri.img       # KolibriOS image
```

## Build (from source)

```bash
# install v86 dependencies and generate the bundle
cd lib
npm init -y
npm install v86 esbuild

cat > bundle-entry.js << 'INNER'
import { V86Starter } from "./node_modules/v86/build/index.js";
import { seabios, vgabios } from "./node_modules/v86/build/binaries.js";
window.V86Starter = V86Starter;
window.v86Binaries = { seabios, vgabios };
INNER

npx esbuild bundle-entry.js --bundle --format=iife --outfile=v86-bundle.js

# generate the WASM as base64 (bypasses file:// restriction on webOS)
node -e "
const fs = require('fs');
const wasm = fs.readFileSync('node_modules/v86/build/v86.wasm');
fs.writeFileSync('v86-wasm-b64.js',
  'window.v86WasmBase64 = \"' + wasm.toString('base64') + '\";');
"

# package
cd ..
ares-package --no-minify .
```

## Known Limitations

- webOS enforces a per-process memory ceiling. On TVs with low free RAM, the system's OOM killer may terminate the app.
- **No persistence between boots**: changes made inside KolibriOS (resolution, theme, files) stay in RAM and are lost when the app is closed.
- **Video selection on boot**: the KolibriOS bootloader shows the video mode menu for a few seconds. To pick one, click the canvas and press an arrow key before the timeout — otherwise it auto-selects the default.

## Credits

- [v86](https://github.com/copy/v86) — Fabrice Bellard and contributors (BSD-2-Clause)
- [KolibriOS](https://kolibrios.org/) — KolibriOS community (GPLv2)
- This app — just gluing both inside the webOS sandbox

## License

This app's code (HTML, glue JS, README) is MIT.
v86 is BSD-2-Clause. KolibriOS is GPLv2.
