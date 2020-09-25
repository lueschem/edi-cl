# edi Project Configuration for Compulab Devices

This [edi](https://www.get-edi.io) project configuration currently supports the
[Compulab iot-gate-imx8](https://www.compulab.com/products/iot-gateways/iot-gate-imx8-industrial-arm-iot-gateway/).

## Introduction

The edi configuration contained in this repository can be used to
generate the following artifacts:

* A Debian buster arm64 (64bit) image suitable for the Compulab iot-gate-imx8.
* A matching Mender update artifact for the above configuration.
* An amd64/arm64 based LXD container with a pre-installed
cross development toolchain for C and C++.

## Important Note

Please note that image generation operations require superuser privileges
and therefore you can easily break your host operating system. Therefore
make sure that you have a backup copy of your data.

## Basic Usage

### Preparation

Prior to using this edi project configuration you have to install
[edi](https://www.get-edi.io) according to
[this instructions](https://docs.get-edi.io/en/latest/getting_started.html).
Please take a careful look at the "Setting up ssh Keys" section since you
will need a proper ssh key setup in order to access the container or
the target device using ssh.

The image post processing commands require some additional tools. On
Ubuntu 20.04 those tools can be installed as follows:

``` bash
sudo apt install e2fsprogs bmap-tools mtools parted zerofree python3-sphinx mender-artifact
```

### Creating a Target Image

A target image can be created using the following command:

``` bash
sudo edi -v image create iot-gate-imx8-buster-arm64.yml
```

The resulting image can be copied to a USB stick (here /dev/sda)
using the following command
(**Please note that everything on the USB stick will be erased!**):

``` bash
sudo bmaptool copy artifacts/iot-gate-imx8-buster-arm64.img /dev/sda
```

If the command fails, unmount the USB stick (`sudo umount /dev/sda?`) and repeat the above command.

**Warning: The image requires u-boot version 2.0 or above!** Please follow the
[Compulab instructions](https://mediawiki.compulab.com/w/index.php?title=IOT-GATE-iMX8_and_SBC-IOT-iMX8:_U-Boot:_Update)
in case you need to upgrade the bootloader of your device.

Once you have booted the device using the above USB stick (plugged into the USB port
next to the power button) you can access it using ssh (the access should be granted
thanks to to your ssh keys):

``` bash
ssh compulab@IP_ADDRESS
```

The password for the user _compulab_ is _compulab_ (just in case you want to
execute a command using `sudo` or login via a local terminal).

### Flashing the Image to the eMMC

The same image that has been used for the USB stick can also be flashed to the builtin eMMC as follows:

Copy the image to the device that has been booted from the USB stick:

``` bash
scp artifacts/iot-gate-imx8-buster-arm64.img compulab@IP_ADDRESS:
```

Access the device:

``` bash
ssh compulab@IP_ADDRESS
```

Flash the image to the eMMC (**Everything on mmcblk2 will be erased!**):

``` bash
sudo dd if=iot-gate-imx8-buster-arm64.img of=/dev/mmcblk2 bs=1M
```

Now you can remove the power supply and the USB stick from the device.
When powering up the device again, it should boot the new image from the eMMC storage device.

### Connecting to Mender

To enable over the air (OTA) updates, the generated images are configured
to connect to [https://hosted.mender.io/](https://hosted.mender.io/).
In order to connect to your Mender tenant you have to provide your tenant token prior to building the images.
The tenant token can be added to `configuration/mender/mender.yml`. If you do not want to
add the tenant token to the version control system you can also copy `configuration/mender/mender.yml` to
`configuration/mender/mender_custom.yml` and add the tenant token there.

### Creating a Cross Development LXD Container

A cross development container can be created using the
following command:

``` bash
sudo edi -v lxc configure iot-gate-imx8-buster-arm64-cross-dev iot-gate-imx8-buster-arm64-cross-dev.yml
```

The container can be accessed as follows (the password is _ChangeMe!_):

``` bash
lxc exec iot-gate-imx8-buster-arm64-cross-dev -- login ${USER}
```

Or with ssh (Hint: retrieve IP_OF_CONTAINER with `lxc list`):

``` bash
ssh IP_OF_CONTAINER
```

You can directly start to cross compile applications:


``` bash
aarch64-linux-gnu-g++ ...
```

For your convenience, the LXD container shares the folder _edi-workspace_
with the host operating system.


## Documenting an Artifact

During the image build the documentation gets rendered to artifacts/CONFIGNAME_documentation
as reStructuredText. The text files can be transformed into a nice pdf file with some
additional tools that need to be installed first:

``` bash
sudo apt install texlive-latex-recommended texlive-pictures texlive-latex-extra latexmk
```

Then the pdf can be generated using the following commands:

``` bash
cd artifacts/CONFIGNAME_documentation
make latexpdf
```

### More Information

For more information about the Compulab device please take a look at the
[official documentation](https://www.compulab.com/products/iot-gateways/iot-gate-imx8-industrial-arm-iot-gateway/).

For more information about this setup please read the [edi documentation](https://docs.get-edi.io) and
[this blog post](https://www.get-edi.io/A-new-Approach-to-Operating-System-Image-Generation/).

For details about the Mender based robust update integration please refer to this
[blog post](https://www.get-edi.io/Updating-a-Debian-Based-IoT-Fleet/).

If you are curious about the U-Boot bootloader setup please take a look at this
[blog post](https://www.get-edi.io/Booting-Debian-with-U-Boot/).
