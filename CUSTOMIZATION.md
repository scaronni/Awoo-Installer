# Customization

Leaf Installer ships with default graphics and sounds, but the user can override any of them at runtime by dropping replacement files into the app's data folder on the SD card.

## Where to put the files

All customization files go in:

```
sdmc:/switch/Leaf-Installer/
```

This is the same folder where `config.json` (Leaf Installer's settings) lives. The folder is created automatically the first time you launch the app, so you can drop replacements in afterwards.

## Files you can override

| Filename | Replaces | Where it appears | Render size | Format |
|---|---|---|---|---|
| `background.png` | `romfs:/images/background.jpg` | Full-screen background on every page | **1920 × 1080** | **PNG-24** (no transparency) or PNG-32 |
| `logo.png` | *(no override)* — bundled only | Top-bar "Leaf Installer" wordmark | **720 × 140** | PNG-32 (RGBA — needs transparency) |
| `leaf_main.png` | `romfs:/images/leaf_main.png` | Decorative figure on the right side of the **main menu** | **1296 × 732** | **PNG-32** (RGBA) |
| `leaf_inst.png` | `romfs:/images/leaf_inst.png` | Decorative figure on the right side of the **install / progress screen** | **1146 × 831** | **PNG-32** (RGBA) |
| `success.mp3` | `romfs:/audio/success.mp3` | Plays on **successful install / Amiibo generation completion** | n/a | **MP3** (also accepts **WAV** — SDL_mixer is initialized for both at startup) |
| `failure.mp3` | `romfs:/audio/failure.mp3` | Plays on **install failure** and **NCA signature-verification warning** | n/a | **MP3** (or WAV, same as above) |

Notes on each:

- **Render size matters.** The image is stretched to the listed dimensions at render time using bilinear filtering. Matching the listed size exactly is sharpest; oversized PNGs waste space without improving quality; undersized PNGs look soft.
- **PNG-32 vs PNG-24.** Use PNG-32 when the image has transparent regions (the leaf figures and the logo need this — the dark-red bar should show through the alpha). Use PNG-24 or even PNG-8 for fully opaque content like the background.
- **Background is JPG by default**, but the override file must be named `background.png`. JPG is bigger-for-equivalent-quality for natural images, but PNG is what Leaf looks for at the override path.
- **Sound format.** The `SDL2_mixer` runtime is initialized for both **MP3** (via `mpg123`) and **WAV** (built-in). The override filename's extension must match the file's actual format — `success.mp3` must be an MP3, `failure.mp3` must be an MP3. If you want to use WAV, you'd need to also update the path strings in the install drivers — currently the code looks for `.mp3` extensions exclusively. OGG/FLAC/Opus decoders are linked but not initialized; adding `MIX_INIT_OGG | MIX_INIT_OPUS` to the `Mix_Init` call in `source/util/util.cpp:initApp` would unlock them.

## How override resolution works

For each asset, Leaf Installer checks if the override file exists in `sdmc:/switch/Leaf-Installer/` first. If it does, that file is used. Otherwise the bundled romfs default loads. There is no need to delete the bundled file — your replacement simply takes priority.

## Interactions with Settings

The Settings menu has a **"Remove leaves"** toggle (the `noGraphics` flag in `config.json`) that turns off the decorative content even if you've replaced it:

- The decorative leaf figure (`leaf_main.png` / `leaf_inst.png`) is hidden on every page.
- Success and failure sounds (`success.mp3` / `failure.mp3`) are silenced.
- The "Leaf Installer" wordmark slides to the left to leave room for the version text.

## Building your own assets

Best results come from authoring at the exact target render size. Recommendations:

- **Photographs / painted backgrounds** → 1920×1080, PNG-24 quality. If file size matters, export JPG quality 85 and rename to `background.png`.
- **Character art with transparency** → 1296×732 (main) or 1146×831 (install), PNG-32 quality.
- **Logos / wordmarks** → 720×140, PNG-32. Anti-alias the edges and leave a couple of pixels of transparent padding to avoid hard edges against the top bar.
- **Sounds** → MP3, 22050 or 44100 Hz, mono or stereo. Keep them short (1–3 seconds). Loudness should be normalized to peak around -3 dBFS so they're audible without being jarring.

## Restoring defaults

To revert any single asset, just delete (or rename) the override file in `sdmc:/switch/Leaf-Installer/`. The bundled romfs asset will be used on the next launch.
