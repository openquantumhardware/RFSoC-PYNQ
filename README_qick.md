# QICK-specific notes
  
## Our changes
* backport Xilinx#17 so the MAC address is read correctly from EEPROM on ZCU216 and ZCU111 (this did not work in ZCU216 v2.7 and ZCU111 v3.0.1)
* don't build the base overlay, which we don't use
* remove a bunch of Ubuntu packages we don't use, and shrink the SD card image
* add ZCU216 as a copy of ZCU208 (there is actually no difference between the two boards on the PS side, so ZCU208 images can be used with ZCU216, but it's nice to have an image with the boardname set correctly)

## Running the build
This has been tested for 4x2, 111, 216, and 208, running on Ubuntu 18.04.
You should read the Xilinx READMEs (both the one in this directory and in `pynq/sdbuild`), but in short:
* install Vivado and Petalinux 2022.1
* download the BSPs for the boards you want to build for
* run in an account that has passwordless sudo

Here's an example script, which expects BSPs (from Xilinx) and a prebuilt image (from Xilinx, or [ours](https://s3df.slac.stanford.edu/people/meeg/qick/sd_images/rootfs/jammy.aarch64.3.0.1.tar.gz)) in `~/pynq_deps`:
```
git clone --recursive https://github.com/openquantumhardware/RFSoC-PYNQ.git

ln -s ~/pynq_deps/bsp/xilinx-zcu216-v2022.1-04191534.bsp RFSoC-PYNQ/boards/ZCU216/ZCU216.bsp
ln -s ~/pynq_deps/bsp/xilinx-zcu111-v2022.1-04191534.bsp RFSoC-PYNQ/boards/ZCU111/ZCU111.bsp
ln -s ~/pynq_deps/bsp/RFSoC4x2_2022_1.bsp RFSoC-PYNQ/boards/RFSoC4x2/RFSoC4x2.bsp
ln -s ~/pynq_deps/bsp/xilinx-zcu208-v2022.1-04191534.bsp RFSoC-PYNQ/boards/ZCU208/ZCU208.bsp

# to use prebuilt board-agnostic image (if you don't do this, by default the script will download ours)
ln -s ~/pynq_deps/jammy.aarch64.3.0.1.tar.gz RFSoC-PYNQ/pynq/sdbuild/prebuilt/pynq_rootfs.aarch64.tar.gz

cd RFSoC-PYNQ
source ~/Soft/Xilinx/Vitis/2022.1/settings64.sh ; source ~/Soft/Xilinx/PetaLinux/2022.1/settings.sh

# to generate prebuilt board-agnostic image (if you didn't symlink it, and don't want the script to download it)
#make BOARD=ZCU216 REBUILD_PYNQ_ROOTFS=1
# mv pynq/sdbuild/output/jammy.aarch64.3.0.1.tar.gz ~/pynq_deps/

for board in ZCU216 ZCU111 RFSoC4x2 ZCU208
do
  make BOARD=$board
  echo "success???"|mailx -s "$board build done" -- suemura@fnal.gov
done
```

The gzipped images will end up in `RFSoC-PYNQ/pynq/sdbuild/build/`.
