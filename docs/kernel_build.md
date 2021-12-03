# Kernel Build Instructions

The following instructions are based upon the
[Compulab instructions](https://github.com/compulab-yokneam/meta-bsp-imx8mm/blob/iot-gate-imx8_r2.4/Documentation/linux_kernel_build.md) but got slightly adapted so that a Debian package gets built.


First we enter the cross development LXD container (for more details please check the README.md of this project):

``` bash
ssh IP_OF_CROSS_DEV_CONTAINER
```

For the kernel build we install some additional Debian packages within the development container:

``` bash
sudo apt install build-essential bc kmod cpio flex cpio libncurses5-dev bison libssl-dev wget lzop rsync
```

Then we pull the Compulab patches and apply them to the linux-imx kernel:

``` bash
export NXP_RELEASE=rel_imx_5.4.24_2.1.0
export CPL_BRANCH=iot-gate-imx8_r2.4
cd ~/edi-workspace
mkdir -p ${CPL_BRANCH} && cd ${CPL_BRANCH}
git clone -b ${CPL_BRANCH} https://github.com/compulab-yokneam/meta-bsp-imx8mm.git
export PATCHES=$(pwd)/meta-bsp-imx8mm/recipes-kernel/linux/compulab/imx8mm
git clone https://source.codeaurora.org/external/imx/linux-imx.git
cd linux-imx
git checkout -b linux-compulab ${NXP_RELEASE}
git am ${PATCHES}/*.patch
```

Finally we configure and build the kernel:

``` bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- iot-gate-imx8_defconfig
make -j $(nproc) KBUILD_IMAGE=arch/arm64/boot/Image ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- deb-pkg
```

The resulting Debian package got uploaded to [this packagecloud repository](https://packagecloud.io/get-edi/debian/).
