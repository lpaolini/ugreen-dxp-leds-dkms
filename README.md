# UGREEN DXP LEDs DKMS package

This repository is a fork of
[miskcoo/ugreen_leds_controller](https://github.com/miskcoo/ugreen_leds_controller).
It has been intentionally reduced to only the `led-ugreen` DKMS kernel module
and reorganized as a Debian package source tree.

The original project includes the reverse-engineered LED protocol, CLI tools,
systemd helper scripts, packaging experiments, and platform-specific build
support. This fork keeps only the code needed to build and publish the Debian
DKMS package `ugreen-dxp-leds-dkms`.

The branch layout is designed for a GitHub Pages Debian repository:

- `debian/` contains the Debian packaging metadata.
- `ugreen-dxp-leds.c`, `ugreen-dxp-leds.h`, `Makefile`, and `dkms.conf` are the DKMS
  module source installed under `/usr/src`.
- `.github/workflows/debian-pages.yml` builds the `.deb`, generates the APT
  repository metadata, and publishes it through GitHub Pages.

## Install

Building from source is optional. The package is already published through the
GitHub Pages Debian repository, and the latest `.deb` is also available as a
direct download.

Install from the Debian repository:

```sh
echo 'deb [trusted=yes] https://lpaolini.github.io/ugreen-dxp-leds-dkms/debian stable main' | sudo tee /etc/apt/sources.list.d/ugreen-dxp-leds-dkms.list
sudo apt update
sudo apt install ugreen-dxp-leds-dkms
```

Or download and install the latest `.deb` directly:

```sh
curl -LO https://lpaolini.github.io/ugreen-dxp-leds-dkms/downloads/ugreen-dxp-leds-dkms_latest.deb
sudo apt install ./ugreen-dxp-leds-dkms_latest.deb
```

The package registers the module with DKMS and configures `i2c-dev`,
`led-ugreen`, `ledtrig-oneshot`, and `ledtrig-netdev` to load at boot.

## Build

Only build locally if you want to modify the package or inspect the generated
artifact yourself.

Install the Debian build dependencies:

```sh
sudo apt install build-essential debhelper dh-dkms dkms
```

Build the package from the repository root:

```sh
dpkg-buildpackage -b -us -uc -tc
```

The resulting `ugreen-dxp-leds-dkms_*.deb` package is written to the parent
directory by `dpkg-buildpackage`.
