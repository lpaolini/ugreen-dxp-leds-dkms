# UGREEN DXP LED kernel module

This repository is based on a fork of
[miskcoo/ugreen_leds_controller](https://github.com/miskcoo/ugreen_leds_controller).

It has been intentionally reduced to only the `led-ugreen` DKMS kernel module
and reorganized as a standard Debian package source tree.

The original project includes the reverse-engineered LED protocol, CLI tools,
systemd helper scripts, packaging experiments, and platform-specific build
support. This fork keeps only the code needed to build and publish the Debian
DKMS package `ugreen-dxp-leds-dkms`.

The branch layout is designed for a GitHub Pages Debian repository:

- `debian/` contains the Debian packaging metadata.
- `ugreen-dxp-leds.c`, `ugreen-dxp-leds.h`, and `Makefile` are the module
  source installed under `/usr/src`.
- `debian/ugreen-dxp-leds-dkms.dkms` is the DKMS configuration template used by
  `dh-dkms`.
- `.github/workflows/debian-pages.yml` builds the `.deb`, generates the APT
  repository metadata, and publishes it through GitHub Pages.

## Install

Building from source is optional. The package is already published through the
GitHub Pages Debian repository, and the latest `.deb` is also available as a
direct download.

Install from the Debian repository:

```sh
sudo install -d -m 0755 /etc/apt/keyrings

curl -fsSL https://lpaolini.github.io/ugreen-dxp-leds-dkms/debian/public.key | sudo gpg --dearmor -o /etc/apt/keyrings/ugreen-dxp-leds-dkms.gpg

echo 'deb [signed-by=/etc/apt/keyrings/ugreen-dxp-leds-dkms.gpg] https://lpaolini.github.io/ugreen-dxp-leds-dkms/debian stable main' | sudo tee /etc/apt/sources.list.d/ugreen-dxp-leds-dkms.list

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
It also installs a systemd oneshot service that binds the `led-ugreen` driver
to the LED controller at I2C address `0x3a`.

If the module is loaded but `/sys/class/leds/ugreen:white:disk1` does not
exist, the I2C device has probably not been created yet. Run the helper
manually:

```sh
sudo /usr/libexec/ugreen-dxp-leds-dkms/probe-led-ugreen
ls /sys/class/leds
```

If the helper tries the wrong bus, list adapters and force the correct bus:

```sh
i2cdetect -l
echo I2C_BUS=1 | sudo tee /etc/default/ugreen-dxp-leds-dkms
sudo /usr/libexec/ugreen-dxp-leds-dkms/probe-led-ugreen
```

Replace `1` with the bus number that owns the LED controller.

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
