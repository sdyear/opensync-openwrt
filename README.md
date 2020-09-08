# OpenSync on OpenWrt

This project allows you to deploy individual PSK authentication on OpenWrt based platforms configured through SDN. This repository provides everything that is needed to build the custom version of OpenSync and run the controller to deploy the project.

## Building and Setting Up

### building OpenSync 

These instruction cover how to build and set up this project using the example vendor target provided. Instructions on how to create other vendor target layers for other OpenWrt based APs are also provided below.

After cloning this repository, change directory into the cloned repository and fetch the submodules:

```
cd opensync-OpenWrt
git submodule update --init
```
An example target OpenWRT layer is provided in `opensync/vendor/ath79`, this can be modified to support other OpenWRT targets with some relatively small changes.

1. Copy `vendor/ath79` to `vendor/<new_target>`
2. Modify `build/vendor-arch.mk`, replacing "ATH79" and "ath79" with the new name
3. Rename and adjust `vendor/<new_target>/src/lib/target/inc/target_<new_target>.h`
4. Modify `vendor/<new_target>/src/lib/target/entity.c` so that the functions return the correct model name, id, etc.

The address of the OpenSync controller can be set either before building the package or by SSHing into AP once OpenSync is running.
To set the address before building the package modify the line begining with `CONTROLLER_ADDR` in `opensync/platform/openwrt/build/openwrt.mk` with the format `CONTROLLER_ADDR="tcp:<ip of controller>:<port>"`. Port 6440 is recomended.

To build the OpenSync Package change directory to `example` and then run `make TARGET=<oepnwrt target> SDK_URL=<OpenWrt SDK>`.
To build OpenSync using the vendor target given in this repository run the commands below.

```
cd example
make TARGET=ATH79 SDK_URL=https://mirror.fsmg.org.nz/openwrt/snapshots/targets/ath79/nand/openwrt-sdk-ath79-nand_gcc-8.4.0_musl.Linux-x86_64.tar.xz
```
This will output an .ipk file in the `example/out/` directory.

### Installing OpenSync

Installing the custom OpenSync package on OpenWRT requires these packages to also be installed 
- libev
- jansson
- protobuf
- libprotobuf-c
- libmosquitto
- libopenssl
- openvswitch
- libpcap
- hostapd (a patched version of hostapd that works with Open vSwitch is required and available [here](http://packages.wand.net.nz/openwrt/hostapd/).

To change the address at which OpenSync expects the OpenSync controller to be at while OpenSync is running, use the command below. Port 6440 is recommended.

`/usr/plume/tools/ovsh u AWLAN_Node redirector_addr:=:"tcp:<IP address of controller>:<port>"`

### Running the Controller

The controller is in the opensync-controller directory and is run with the command ./controller <config-file.yml> <port number>. Port 6440 is recommended.

The controller is written in python3 and requires the following python packages to be installed:
- pyyaml
- psycopg2
- pyrad
- six

The controller configuration file is YAML based. An example configuration file is included in the opensync-controller directory.


Overview
--------

The `opensync-OpenWrt` project consists of the following key components:

* [opensync/core](https://github.com/sdyear/opensync)
    - OpenSync core repository, included as a submodule (see https://opensync.io/documentation for more details)
* [opensync/platform/openwrt](https://github.com/sdyear/opensync-platform-openwrt)
    - an example OpenSync target layer for OpenWrt based platforms
* [opensync/vendor/armvirt](https://github.com/sdyear/opensync-vendor-ath79)
    - an example OpenSync vendor layer for the OpenWRT `ath79` target
* [opensync-controller](https://github.com/sdyear/opensync-controller)
    - OpenWrt makefile needed to build OpenSync
