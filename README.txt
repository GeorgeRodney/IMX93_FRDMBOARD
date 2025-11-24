# Create a custom linex image using Yocto. 
# Flash the image to my IMX93

    Linux Kernel    :   An operating sytem that manages hardware resources. 

    YOCTO           :   High level project designed to simplify the creation
                    :   of custom embedded linux distros.

[x] :   Clone POKY

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

###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###
###
###     Well. This actually works to flash the NXP sourced imx-image-core to the imx93 frdm board. 
###
###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###---###
1. Create custom linux distro and generate image using Yocto

2. Files necessary to flash image to eMMC
        :       imx-image-core-imx93evk.rootfs.wic.zst          (Root file system)
        :       imx-boot-imx93evk-sd.bin-flash_singleboot       (Bootloader)

3. Decompress unzstd imx-image-core-imx93evk.rootfs-20251123182859.wic.zst
        :       unzstd imx-image-core-imx93evk.rootfs-20251123182859.wic is the output

4. Put the board in Serial Download Mode [1 0 0 0]

5. Check to be sure the device is available using: lsusb
        :       Example of the NXP being available      
        :       Bus 001 Device 011: ID 1fc9:0152 NXP Semiconductors USB download gadget

6. Flash with uuu:
        EmbeddedGuy ~/imx-bsp/build/tmp/deploy/images/imx93evk $ sudo uuu -b emmc_all imx-boot-imx93evk-sd.bin-flash_singleboot imx-image-core-imx93evk.rootfs-20251123182859.wic

7. Successful output:
        :       Success 1    Failure 0                                                         
                1:3      8/ 8 [Done                                  ] FB: done  