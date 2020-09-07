# OpenSync on OpenWrt

This project allows you to deploy individual PSK authentication on OpenWrt based platforms configured through SDN. This repositry porvides everything that is needed to build the custom version of OpenSync and run the controller to deploy the project.

## Building and Setting Up

### building OpenSync 

These instruction cover how to build and set up this project using the example vendor target provided. Instructions on how to create other vendor target layers for other OpenWrt based APs are also provided below.

After cloning this repository, change directory into the cloned repository and fetch the submodules:

```
cd opensync-openwrt
git submodule update --init
```
An example target OpenWRT layer is provided in `opensync/vendor/ath79`, this can be modified to suport other OpenWRT targets with some relativly small changes.

1. Copy `vendor/ath79` to `vendor/<new_target>`
2. Modify `build/vendor-arch.mk`, replacing "ATH79" and "ath79" with the new name
3. Rename and adjust `vendor/<new_target>/src/lib/target/inc/target_<new_target>.h`
4. Modify `vendor/<new_target>/src/lib/target/entity.c` so that the functions return the correct model name, id, etc.

To build the OpenSync Package change direcotry to `example` and then run make TARGET=<oepnwrt target> SDK_URL=<OpenWrt SDK>.

```
cd example
make TARGET=ATH79 SDK_URL=https://mirror.fsmg.org.nz/openwrt/snapshots/targets/ath79/nand/openwrt-sdk-ath79-nand_gcc-8.4.0_musl.Linux-x86_64.tar.xz
```
This will output an .ipk file in the `example/out/` directory.

Overview
--------

The `opensync-openwrt` project consists of the following key components:

* [opensync/core](https://github.com/plume-design/opensync)
    - OpenSync core repository, included as a submodule (see https://opensync.io/documentation for more details)
* [opensync/platform/openwrt](https://github.com/plume-design/opensync-platform-openwrt)
    - an example OpenSync target layer for OpenWrt based platforms (minimum implementation, stubs only)
* [opensync/vendor/armvirt](https://github.com/plume-design/opensync-vendor-armvirt)
    - an example OpenSync vendor layer for QEMU `armvirt` target (minimum implementation, stubs only)
* [feeds/network/services/opensync/Makefile](feeds/network/services/opensync/Makefile)
    - OpenWrt makefile needed to build OpenSync
* [feeds/lang/python/python3-kconfiglib/Makefile](feeds/lang/python/python3-kconfiglib/Makefile)
    - OpenWrt makefile needed to build the host-side kconfiglib dependency

External dependency:

* [OpenWrt SDK](https://openwrt.org/docs/guide-developer/using_the_sdk)
    - For more information, consult [OpenWrt Documentation](https://openwrt.org/docs/start)


Creating a Custom Target (Vendor Layer)
---------------------------------------

To create a new target, it is best to use the `vendor/armvirt` tree as a template.
Follow these steps for the basic bring-up of a new target:

1. Copy `vendor/armvirt` to `vendor/<new_target>`
2. Modify `build/vendor-arch.mk`, replacing "ARMVIRT" and "armvirt" with the new name
3. Rename and adjust `vendor/<new_target>/src/lib/target/inc/target_<new_target>.h`
4. Modify `vendor/<new_target>/src/lib/target/entity.c` so that the functions return the correct model name, id, etc.
5. Modify `vendor/<new_target>/ovsdb/static_configuration.json` according to actual Wi-Fi radios configuration
