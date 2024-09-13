# edi Project Configuration for Compulab Devices

This [edi](https://www.get-edi.io) project configuration currently supports the
[Compulab iot-gate-imx8](https://www.compulab.com/products/iot-gateways/iot-gate-imx8-industrial-arm-iot-gateway/).

<img alt="Compulab iot-gate-imx8" src=https://www.get-edi.io/assets/images/blog/IOT-GATE-iMX8.png width="75%"/>

## Introduction

The edi configuration contained in this repository can be used to
generate the following artifacts:

* A **minimal** Debian trixie arm64 (64bit) image suitable for the Compulab iot-gate-imx8.
* A matching Mender update artifact for the above configuration.
* A Podman/Docker image with a pre-installed cross development toolchain (arm64) for C and C++.

## Basic Usage

### Preparation

Prior to using this edi project configuration you have to install
[edi](https://www.get-edi.io) according to
[this instructions](https://docs.get-edi.io/en/latest/getting_started.html).
Please take a careful look at the "Setting up ssh Keys" section since you
will need a proper ssh key setup in order to access the
the target device using ssh.

The artifact generation requires some additional tools. On
Ubuntu 24.04 and newer those tools can be installed as follows:

``` bash
sudo apt install buildah containers-storage crun curl distrobox dosfstools e2fsprogs fakeroot genimage git mender-artifact mmdebstrap mtools parted python3-sphinx python3-testinfra podman rsync zerofree
```

### Cloning this Repository

The edi-cl project configuration contained in this git repository can be cloned as follows:

``` bash
mkdir -p ~/edi-workspace/ && cd ~/edi-workspace/
git clone --recursive https://github.com/lueschem/edi-cl.git
```

The following steps assume that you are located within the project configuration directory:

``` bash
cd ~/edi-workspace/edi-cl/
```

If the repository has already been cloned earlier, do not forget to update the submodules:

``` bash
git submodule update --init
```

### Optional: Connecting to Mender

To enable over the air (OTA) updates, the generated images are configured
to connect to [https://hosted.mender.io/](https://hosted.mender.io/).
In order to connect to your Mender tenant you have to provide your tenant token prior to building the images.
The tenant token can be added to `configuration/mender/mender.yml`. If you do not want to
add the tenant token to the version control system you can also copy `configuration/mender/mender.yml` to
`configuration/mender/mender_custom.yml` and add the tenant token there.

### Creating a Target Image

A target image can be created using the following command:

``` bash
edi -v project make iot-gate-imx8.yml
```

The resulting image can be copied to a USB stick (here /dev/sda)
using the following command
(**Please note that everything on the USB stick will be erased!**):

``` bash
sudo umount /dev/sda?
sudo dd if=artifacts/iot-gate-imx8.img of=/dev/sda bs=4M conv=fsync status=progress
```

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
scp artifacts/iot-gate-imx8.img compulab@IP_ADDRESS:
```

Access the device:

``` bash
ssh compulab@IP_ADDRESS
```

Flash the image to the eMMC (**Everything on mmcblk2 will be erased!**):

``` bash
sudo dd if=iot-gate-imx8.img of=/dev/mmcblk2 bs=4M conv=fsync status=progress
```

Now you can remove the power supply and the USB stick from the device.
When powering up the device again, it should boot the new image from the eMMC storage device.

### Creating a Cross Development Container

A Podman image of a cross development container can be created using the
following command:

``` bash
edi -v project make iot-gate-imx8-cross-dev.yml
```

distrobox can be used to transform the image into a convenient development container:

``` bash
source artifacts/pi-cross-dev_manifest
distrobox create --image ${podman_image} --name SOME_CONTAINER_NAME --init --unshare-all --additional-packages "systemd libpam-systemd"
```

For all the development work the container can be entered as follows:

``` bash
distrobox enter SOME_CONTAINER_NAME
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
sudo apt install texlive-latex-recommended texlive-pictures texlive-latex-extra texlive-xetex latexmk
```

Then the pdf can be generated using the following commands:

``` bash
cd artifacts/CONFIGNAME_documentation
make PDFLATEX=xelatex latexpdf
make PDFLATEX=xelatex latexpdf
```

### More Information

For more information about the Compulab device please take a look at the
[official documentation](https://www.compulab.com/products/iot-gateways/iot-gate-imx8-industrial-arm-iot-gateway/).

For more information about this setup please read the [edi documentation](https://docs.get-edi.io) and
[this blog post](https://www.get-edi.io/Rootless-Creation-of-Debian-OS-and-OCI-Images/).

For details about the Mender based robust update integration please refer to this
[blog post](https://www.get-edi.io/Updating-a-Debian-Based-IoT-Fleet/).

If you are curious about the U-Boot bootloader setup please take a look at this
[blog post](https://www.get-edi.io/Booting-Debian-with-U-Boot/).

For the kernel build instructions please check the [docs folder of this project](docs/kernel_build.md).

The WiFi setup is [documented here](docs/wifi_setup.md).
