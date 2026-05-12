# Leaf Installer
An NSP, NSZ, XCI, and XCZ Installer for Nintendo Switch.

Derived from [Awoo Installer](https://github.com/Huntereb/Awoo-Installer).

![Awoo Installer Main Menu](/images/main.jpg)

## Features
- Uses [XorTroll's Plutonium](https://github.com/XorTroll/Plutonium) for a pretty graphical interface
- Installs NSP/NSZ/XCI/XCZ files:
  - Over LAN or USB from tools such as [NS-USBloader](https://github.com/developersu/ns-usbloader)
  - From the SD card
  - Over the internet by URL or Google Drive
- Verifies NCAs by header signature before they're installed
- Auto updates from the Github releases page.
- Installs and manages the latest Ultrahand and some overlays:
  - [Ultrahand overlay](https://github.com/ppkantorski/Ultrahand-Overlay)
  - [sys-patch](https://github.com/impeeza/sys-patch)
  - [emuiibo](https://github.com/xortroll/emuiibo)
- Generates Amiibo data for [emuiibo](https://github.com/xortroll/emuiibo) directly on the console:
  - Downloads the Amiibo database from a configurable JSON endpoint (defaults to [amiiboapi.org](https://amiiboapi.org/api/amiibo/))
  - Generates `amiibo.json` + `amiibo.flag` and a 150px-tall PNG icon for every entry under `sdmc:/emuiibo/amiibo/`
  - Skips Amiibos that already exist on disk, so it's safe to re-run after the database is refreshed
  - Inspired by [Slluxx/AmiiboGenerator](https://github.com/Slluxx/AmiiboGenerator)
- All update endpotins are configurable under `Settings`.
- Show storage information in the main window. It is also updated:
  - After every NSP install for network and SD installs.
  - At the end of the NSP install batch for USB installs (NS-USBLoader protocol limitation).

## Building with Podman
The GitHub Actions workflow builds Leaf Installer inside the official devkitPro container image (`devkitpro/devkita64:latest`). You can reproduce the exact same build locally with Podman — no need to install devkitPro, devkitA64, or any Switch toolchain on your host.

### Prerequisites
- [Podman](https://podman.io/) installed on your system
- Clone fresh with submodules:
  ```sh
  git clone --recursive https://github.com/dragonflylee/Awoo-Installer.git
  cd Awoo-Installer
  ```
  
  Or, if you already cloned without `--recursive`, populate the submodules from inside the repository:
  ```sh
  git submodule update --init --recursive
  ```

### Build
From the root of the repository, run:
```sh
podman run --rm -it \
    -v "$PWD":/Leaf-Installer:Z \
    -w /Leaf-Installer \
    docker.io/devkitpro/devkita64:latest \
    bash -c "git config --system --add safe.directory '*' && make -j\"\$(nproc)\" all"
```

What this does:
- `--rm` removes the container when the build finishes.
- `-v "$PWD":/Leaf-Installer:Z` mounts the current directory into the container. The mount path matters — the Makefile names the artifact after the working directory (`$(notdir $(CURDIR))`), so mounting at `/Leaf-Installer` produces `Leaf-Installer.nro` just like CI. The `:Z` suffix relabels the volume for SELinux — drop it on systems without SELinux.
- `-w /Leaf-Installer` sets the working directory inside the container.
- `git config --system --add safe.directory '*'` mirrors the workflow step that lets git operate on the bind-mounted tree (it is owned by a different UID inside the container).
- `make -j"$(nproc)" all` runs the same build command as CI.

When the build completes, `Leaf-Installer.nro` will be in the repository root, ready to copy to your Switch's SD card under `/switch/Leaf-Installer/`.

### Clean build
To start from scratch:
```sh
podman run --rm -it -v "$PWD":/Leaf-Installer:Z -w /Leaf-Installer docker.io/devkitpro/devkita64:latest make clean
```

### Note on Docker
The same commands work with Docker — just replace `podman` with `docker` and omit the `:Z` SELinux relabel suffix if your system does not use SELinux.

## Thanks to
- [Plutonium](https://github.com/XorTroll/Plutonium) graphics library
- Original [Awoo Installer](https://github.com/Huntereb/Awoo-Installer)
- Fork of [Awoo Installer](https://github.com/harmonsir/Awoo-Installer) with HTTP and USB hard drive support
- [Claude](https://claude.ai/) for the incredible speed on iterating over features
