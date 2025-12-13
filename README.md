# Create a custom linex image using Yocto. 

    Linux Kernel    :   An operating sytem that manages hardware resources. 

    YOCTO           :   High level project designed to simplify the creation
                    :   of custom embedded linux distros.

# Clone POKY

    POKY    :   A distro of Yocto.
            :   A collection of layers and build scripts.
            :   POKY PARTS:
            :   BitBake - Build engine. Processes recipes to build:
                    :   Toolchains, OS, packages.
            :   Bootloader - Binary that runs first. Initializes the kernel.
            :   drivers - OS interfaces with hardware
                    :   In-tree divers - Native to the linux kernel source tree. EX:
                            :   I2C, SPI, GPIO, MIPI CSI/distro
                    :   Out-of-tree drivers - external modules. 
                            :   Vender supplied or custom.
            :   utilities - non-descript, maybe packages

    recipes :   The build rules
            :   *.bb file
            :   Tells BitBake what to fetch, configure, copmile, install, and package.

            :   Example.
                    :   meta-myThing/
                            recipes-kernel/                 [Kernel build]
                            recipes-bsp/                    [Bootlader]
                            recipes-core/
                            recipes-example/
                            classes/
                            conf/
                            recipes-apps/myApp/myApp_1.0.bb [Custom app]

# Well. This actually works to flash the NXP sourced imx-image-core to the imx93 frdm board. 

## 1. Create custom linux distro and generate image using Yocto

## 2. Files necessary to flash image to eMMC
        imx-image-core-imx93evk.rootfs.wic.zst          (Root file system)

## 3. Decompress unzstd imx-image-core-imx93evk.rootfs-20251123182859.wic.zst
        imx-image-core-imx93evk.rootfs-20251123182859.wic is the output

## 4. Put the board in Serial Download Mode [1 0 0 0]

## 5. Check to be sure the device is available using: lsusb

## 6. Flash with uuu
        sudo uuu -b emmc_all imx-image-core-imx93evk.rootfs-20251123182859.wic

## 7. Successful output:
        Success 1    Failure 0                                                         
        1:3      8/ 8 [Done                                  ] FB: done  

#     Creating cutsom layer with a cpp file in it. Using the Git Repo Route.
             Im using this method because I followed a guide online and it worked. 
             https://blog.mbedded.ninja/programming/embedded-linux/yocto-project/adding-a-custom-app-to-a-yocto-build/

## 1. Create the project files.
        HelloWorld.c
        HelloWorld.h
        LICENSE
        configure.ac
        Makefile.am

## 2. Create a git hub for this project. 

## 3. Make a layer to hold the application.
        mkdir meta-example
        mkdir meta-example/conf
        touch conf/layer.conf

## 4. Add this to conf/layer.conf:

        // We have a conf and classes directory, add to BBPATH
        BBPATH := "${BBPATH}:${LAYERDIR}"
        // We have a packages directory, add to BBFILES
        BBFILES := "${BBFILES} ${LAYERDIR}/recipes-*/*/*.bb \
                ${LAYERDIR}/recipes-*/*/*.bbappend"
        BBFILE_COLLECTIONS += "example"
        BBFILE_PATTERN_example := "^${LAYERDIR}/"
        BBFILE_PRIORITY_example := "5"

## 5. Create recipe.
        mkdir meta-example/recipes-example

## 6. Create application folder.
        mkdir recipes-example/helloworld

## 7. Create a bitbake .bb file in the application folder.
        touch helloworld/helloworld_1.0.bb

## 8. Fill the helloworld_1.0.bb with this:

        //
        // This file was derived from the 'Hello World!' example recipe in the
        // Yocto Project Development Manual.
        //

        DESCRIPTION = "Example Hello, World application for Yocto build."
        SECTION = "examples"
        DEPENDS = ""
        LICENSE = "MIT"
        LIC_FILES_CHKSUM = "file://LICENSE;md5=96af5705d6f64a88e035781ef00e98a8"

        FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}-${PV}:"

        SRCREV = "241a83c92f89a63a32042d421cdf91e93434e925"
        SRC_URI = "git://github.com/GeorgeRodney/meta-hellomike.git;protocol=https;branch=main"

        S = "${WORKDIR}/git"

        inherit autotools

        // The autotools configuration I am basing this on seems to have a problem with a race condition when parallel make is enabled
        PARALLEL_MAKE = ""


## 9. Look at these:
        SRCREV = "c96b1fdd0767a9a13b9fca9d91fd3975c44c9de4"
        To get this run "md5sum LICENSE" on the LICENSE file you created in your application

        SRC_URI = "git://github.com/GeorgeRodney/meta-hellomike.git;protocol=https;branch=main"
        This is the git URL that you created for the above project.


## 10. Add the layer to the bitbake-layers.
        bitbake-layers add-layer meta-example

        You should see something like this in the bblayers.conf file

        BBLAYERS ?= " \
        /home/username/temp/poky/meta \
        /home/username/temp/poky/meta-poky \
        /home/username/temp/poky/meta-yocto-bsp \
        /home/username/temp/poky/meta-example \
        "

## 11. Run it baby.
        bitbake imx-image-core

        Or whatever the BSP package is. 


## 12. Place the IMX-93 board in Serial Wire Mode. 1000

## 13. Unzip the built image file.
        unzstd imx-image-core-imx93evk.rootfs-20251129154309.wic.zst

        You can see that this image is for a imx6qp sabre board.
        MAKE SURE YOU ASSIGN 'MACHINE' correctly in the conf/local.conf file.
        I should have done MACHINE = "imx93evk"

## 14. Flash the board.
        sudo uuu -b emmc_all imx-image-core-imx93evk.rootfs-20251129221548.wic

## 15. helloworld binary in /usr/bin
        Example execution <./helloworld>
        root@imx93evk:/usr/bin#         
        ---- Sent utf8 encoded message: "./helloworld\r" ---- 
        ./helloworld
        Hello, World...

        VOILA!
        

# Problems I ran into. 
        MACHINE = "imx93evk" 
        Changed to "sabre" (some other nxp board). Not sure why. Some scripting presumably. 

        IMAGE_INSTALL:append = "helloworld"
        Need this line in the build/conf/local.conf 
        This tells yocto to add the package that was created by the custom layer's recipe meta data. 
        helloworld_1.0.bb <----- This one
       
        SW1 dip switches on the imx93 board can be cattywompus. Be sure to power off the device and hard click these chicklets. 

        The naming of the custom application needed to match the binary. Im not 100 % sure this is necessary but it seamed to resolve
        A bitbake error I was getting.
        helloworld <-----binary
        helloworld_1.0.bb not HelloWorld_xxxxx. I think caps may matter here. 


# EXAMPLE CODE:

## HelloWorld.c:

        \#include <stdio.h>
        \#include <stdlib.h>
        \#include <stdint.h>

        \#include "HelloWorld.h"

        int main(int argc, char *argv[]) {
        printf("Hello, World...\n");
        return 0;
        }


## HelloWorld.h:

        \#ifndef HELLO_WORLD_H
        \#define HELLO_WORLD_H

        /* Some cross-platform definitions generated by autotools */
        \#if HAVE_CONFIG_H
        \#  include <config.h>
        \#endif /* HAVE_CONFIG_H */

        \#endif /* HELLO_WORLD_H */


# LICENSE:

        The MIT License (MIT)

        Copyright (c) 2014 Dynamic Devices

        Permission is hereby granted, free of charge, to any person obtaining a copy
        of this software and associated documentation files (the "Software"), to deal
        in the Software without restriction, including without limitation the rights
        to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
        copies of the Software, and to permit persons to whom the Software is
        furnished to do so, subject to the following conditions:

        The above copyright notice and this permission notice shall be included in all
        copies or substantial portions of the Software.

        THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
        IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
        FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
        AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
        LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
        OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
        SOFTWARE.


## configure.ac:

        \#                                               -*- Autoconf -*-
        \# Process this file with autoconf to produce a configure script.

        \#AC_PREREQ(2.60)
        AC_INIT(YoctoHelloWorldApp, 1.0, gbmhunter@gmail.com)
        AC_CONFIG_SRCDIR([HelloWorld.c])
        AC_CONFIG_HEADER([config.h])
        AM_INIT_AUTOMAKE(YoctoHelloWorldApp, main)

        \# Checks for programs.
        AC_PROG_CC
        AC_PROG_INSTALL
        AC_PROG_MAKE_SET

        \# Checks for libraries.
        AM_PROG_LIBTOOL

        \# Set shared libraries
        AC_DISABLE_STATIC
        AC_ENABLE_SHARED

        \# Checks for header files.

        \# Checks for typedefs, structures, and compiler characteristics.

        \# Checks for library functions.

        \#AC_CONFIG_MACRO_DIR([m4])
        AC_CONFIG_FILES([Makefile])
        AC_OUTPUT


## Makefile.am:

        AUTOMAKE_OPTIONS = foreign

        CFLAGS = -Wall -pedantic
        include_HEADERS = HelloWorld.h

        bin_PROGRAMS = helloworld
        helloworld_SOURCES = HelloWorld.c

# Having a hard time getting SSH hook up. It seams that the ssh binary is on my target but the "service file" doesnt appear to be here. 

## 1. Turns out the DISTRO I was using didnt have systemd type things available to me. I was attempting to use the most simple/quick to build image as possible.
        In the local.conf 'DISTRO = "fsl-imx-fb"' is now 'DISTRO = "fsl-imx-xwayland"'
        -fb is too pared down. 

# Using Video 4 Linux.