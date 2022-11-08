Steps to Flash SKR Pico RP2040 with Canboot.

- Update to the latest Canboot and Klipper codabase as of 11-8-2022
  - Canboot v0.0.1-20-g600967e or newer
  - Klipper v0.10.0-623-g5b1a6676 or newer
- SSH to  your Klipper host
- CD to Canboot

  - ```cd ~/CanBoot/```
- Run Make Clean

  -   ```make clean```
- Run Make Menuconfig
  
  -   ```make menuconfig```

- Settings
    - MCU Rasberru Pi RP2040
    - Build CanBoot deployment 16Kib Bootloader
    - Com interface CAN bus
    - CAN RX GPIO 1
    - CAN Tx GPIO 0
    - CAN bus speed 500000 <-- Chose your own adventure ( 250000,500000,1000000 )
 - Run make

  - ```make``` 

- Install boot jumper on the SKR Pico and press the reset button
- Mount the Pico file system
  
  - ```sudo mount /dev/sda1 /mnt```

- Copy Canboot.uf2 to the Pico

  - ```sudo cp ~/CanBoot/out/canboot.uf2 /mnt``` 

- This will copy the firmware image we compiled earlier onto the SKR-Pico's storage and it will be flashed automatically.

  - This take a minute or so.

Links:

[Rootiest SKR Pico Canbus guide](https://github.com/rootiest/zippy-klipper_config/blob/master/guides/Guide-pico_can.md)

[Arksine Canboot](https://github.com/Arksine/CanBoot)
