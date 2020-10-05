# OpenSync on OpenWrt with Individual PSK

This project allows you to deploy individual PSK authentication on OpenWrt based platforms configured through SDN. In this implementation of individual PSK based authentication, each AP has a default password which new devices are expected to use when they first connect. When a device connects, it's MAC address is searched in a PostgreSQL database, if the device is not registered in the database the default password of the AP is trying to connect to is searched, and that is returned as the expected password for the device to use. The devices' MAC address is then added to the database with the expected password for the AP the device is trying to connect to as the expected password for it. The same database is searched from all APs when a device connects, that means that once a device has connected once, it can always connect from any AP with only the expected password for the first AP it connected to.

This repository provides everything that is needed to build the custom version of OpenSync and run the controller to deploy the project.

## Building and Setting Up

### Building OpenSync 

These instruction cover how to build and set up this project using the example vendor target provided. Instructions on how to create other vendor target layers for other OpenWrt based APs are also provided below.

After cloning this repository, change directory into the cloned repository and fetch the submodules:

```
cd opensync-openwrt
git submodule update --init
```
An example target OpenWRT layer is provided in `opensync/vendor/ath79`, this can be modified to support other OpenWRT targets with some relatively small changes.

1. Copy `vendor/ath79` to `vendor/<new_target>`
2. Modify `build/vendor-arch.mk`, replacing "ATH79" and "ath79" with the new name
3. Rename and adjust `vendor/<new_target>/src/lib/target/inc/target_<new_target>.h`
4. Modify `vendor/<new_target>/src/lib/target/entity.c` so that the functions return the correct model name, id, etc.

The address of the OpenSync controller can be set either before building the package or by SSHing into AP once OpenSync is running.
To set the address before building the package modify the line begining with `CONTROLLER_ADDR` in `opensync/platform/openwrt/build/openwrt.mk` with the format `CONTROLLER_ADDR="tcp:<ip of controller>:<port>"`. Port 6440 is recomended.

To build the OpenSync Package change directory to `example` and then run `make TARGET=<oepnwrt target> SDK_URL=<OpenWrt SDK>`. The sdk should be for the version of OpenWrt you are targeting and must be version 19.07.1 or later.
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

`/usr/plume/tools/ovsh u AWLAN_Node redirector_addr:="tcp:<IP address of controller>:<port>"`

### Running the Controller

Before the controller is run, a PostgreSQL server with a database of the type definied in [opensync-controller/schema.sql](opensync-controller/schema.sql) must be running for it to connect to. The details of this database are configured in the controllers YAML based configuration file.

The controller is in the opensync-controller directory and is run with the command below. Port 6440 is recommended.

`./controller <config-file.yml> <port number>`

The controller configuration file is YAML based. An example configuration file is included in the opensync-controller directory.

The controller is written in python3 and requires the following python packages to be installed:
- pyyaml
- psycopg2
- pyrad
- six


Overview
--------

The main components of the `opensync-openwrt` project are contained in the submodules:

* [opensync/core](https://github.com/sdyear/opensync)
    - OpenSync core repository (see https://opensync.io/documentation for more details)
* [opensync/platform/openwrt](https://github.com/sdyear/opensync-platform-openwrt)
    - the OpenSync target layer for OpenWrt based platforms
* [opensync/vendor/armvirt](https://github.com/sdyear/opensync-vendor-ath79)
    - an example OpenSync vendor layer for the OpenWRT `ath79` target
* [opensync-controller](https://github.com/sdyear/opensync-controller)
    - OpenWrt makefile needed to build OpenSync
