# Mender Client Backport for Debian Buster

Remark: The Debian bullseye setup is using Mender from the Mender Debian repository. Therefore there is no
backport for Debian bullseye.

The following steps are required to backport a recent version of mender-client to Debian buster
(we do everything on an iot-gate-imx8 to avoid cross compilation).

Remark: The backport can also be done on a Raspberry Pi. If you would like to do it
for armhf then just do it by installing an armhf image instead.

Warning 1: Once you have an image with Mender 2.x running you cannot downgrade to an image
that has Mender 1.x pre installed.

Warning 2: **Don't do this instructions on a Debian installation you want to keep.** We will end up with
a mixed Debian buster/bullseye installation that you better replace with a clean installation afterwards.

## Preparation

Install a recent Debian buster arm64 image on the iot-gate-imx8.

## Backport Steps

Install various build tools:

```
sudo apt update
sudo apt install build-essential devscripts equivs liblzma-dev liblzma5 golang-go debhelper dh-golang golang-src golang-any vim
```

Add `bullseye.list` to `/etc/apt/sources.list.d`:

```
deb http://deb.debian.org/debian bullseye main contrib non-free
deb-src http://deb.debian.org/debian bullseye main contrib non-free
```

Do an apt update and fetch the mender-client source:

```
sudo apt update
sudo apt install dwz/bullseye
mkdir edi-workspace && cd edi-workspace
apt source mender-client && cd mender-client-2.4.1
```

Install the required build dependencies:

```
sudo mk-build-deps --install --remove
```

Assign a backport (bpo) version:

```
dch --bpo
```

Rebuild mender-client:

```
debuild -us -uc
```

## Package Upload

The resulting Debian package can then be uploaded to a Debian package repository.

## Cleanup

According to "Warning 2" it is now recommended to replace the current Debian installation with a clean one.
At least `/etc/apt/sources.list.d/bullseye.list` should be removed again. 


