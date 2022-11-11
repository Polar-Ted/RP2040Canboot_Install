# Steps to Flash SKR Pico  ( RP2040 ) with Canboot
#### [Canbus over CAN](https://github.com/Polar-Ted/RP2040Canboot_Install/blob/main/README.MD#canboot-for-skr-pico-as-a-can-device)

#### [Canbus over USB to can bridge](https://github.com/Polar-Ted/RP2040Canboot_Install/blob/main/README.MD#canboot-for-skr-pico-in-usb-to-can-bridge-mode)

#### [Canbus over USB](https://github.com/Polar-Ted/RP2040Canboot_Install/blob/main/README.MD#canboot-for-skr-pico-in-usb-mode)

#### [Canbus over Serial UART](https://github.com/Polar-Ted/RP2040Canboot_Install/blob/main/README.MD#canboot-for-skr-pico-in-uart-mode)

### Note: I have found that Klipper firmware for USB is not cabable of calling the CanBoot bootloader. To re-flash you will need to quickly press the rest the Pico twice to have it enter the CanBoot bootloader.   

  This guide assumes you alreay have a SKR Pico working on CAN bus. See the [Rootiest SKR Pico Canbus guide](https://github.com/rootiest/zippy-klipper_config/blob/master/guides/Guide-pico_can.md) for more information. 
  
## Install CanBoot if you haven't already
- Run the following from your klipper host to copy the canboot github files
```
cd ~/
git clone https://github.com/Arksine/CanBoot
```

- Add Canboot to the moonraker update manager by adding the following lines to your moonraker.cof file.
```
[update_manager CanBoot]
type: git_repo
path: ~/CanBoot
origin: https://github.com/Arksine/CanBoot.git
is_system_service: False
```

## Canboot for SKR Pico as a CAN device

- Update to the latest Canboot and Klipper codabase as of 11-8-2022
  - Canboot v0.0.1-20-g600967e or newer
  - Klipper v0.10.0-623-g5b1a6676 or newer
- SSH to  your Klipper host
- CD to Canboot

  ```
  cd ~/CanBoot/
  ```
- Run Make Clean
  ```
  make clean
  ```

- Run Make Menuconfig
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - Build CanBoot deployment 16Kib Bootloader
    - Com interface CAN bus
    - CAN RX GPIO 1
    - CAN Tx GPIO 0
    - CAN bus speed 500000 <-- Chose your own adventure ( 250000,500000,1000000 )

![Canboot on CAN](/img/skr_pico_canboot_can.png)    

- Run make
  ```
  make
  ``` 
- Connect the Pico by USB to the Klipper Host
- Install boot jumper on the SKR Pico and press the reset button
- Mount the Pico file system
  ```
  sudo mount /dev/sda1 /mnt
  ```

- Copy Canboot.uf2 to the Pico
  ```
  sudo cp ~/CanBoot/out/canboot.uf2 /mnt
  ``` 

- This will copy the firmware image we compiled earlier onto the SKR-Pico's storage and it will be flashed automatically.

  - This take a minute or so.

- When the /mnt unmounts remove the boot jumper. 

- Remove the USB cable, Reboot and continue to the next phase.. 
 
### Flashing Klpper via canboot over CAN

- SSH into the Klipper host and CD to klipper
  ```
  cd ~/klipper/
  ```

- Run Make Clean
  ```
  make clean
  ```
  
- Run Make Menuconfig
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - bootloader offset 16Kib Bootloader
    - Com interface CAN bus
    - CAN RX GPIO 1
    - CAN Tx GPIO 0
    - CAN bus speed 500000 <-- Chose your own adventure ( 250000,500000,1000000 )

![KLipper on CAN with canbooot offset](/img/skr_pico_klipper_can.png)

- Run make
  ```
  make
  ``` 
- Connect the SKR Pico to the CAN bus. 

- Get your UUID
  ```
  python3 ~/CanBoot/scripts/flash_can.py -q
  ```

 - Flash the klipper.bin file to the SKR Pico
  ```
  python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u {your uuid}
  ```
  
## Canboot for SKR Pico in USB to can bridge mode 
  
- Update to the latest Canboot and Klipper codabase as of 11-8-2022
  - Canboot v0.0.1-20-g600967e or newer
  - Klipper v0.10.0-623-g5b1a6676 or newer
- SSH to  your Klipper host
- CD to Canboot
  ```
  cd ~/CanBoot/
  ```
  
- Run Make Clean
  ```
  make clean
  ```

- Run Make Menuconfig
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - Build CanBoot deployment 16Kib Bootloader
    - Com interface USB

![Canboot in USB mode](/img/skr_pico_canboot_usb.png)

- Run make
  ```
  make
  ``` 
- Connect the Pico by USB to the Klipper Host
- Install boot jumper on the SKR Pico and press the reset button
- Mount the Pico file system
  ```
  sudo mount /dev/sda1 /mnt
  ```

- Copy Canboot.uf2 to the Pico
  ```
  sudo cp ~/CanBoot/out/canboot.uf2 /mnt
  ``` 

- This will copy the firmware image we compiled earlier onto the SKR-Pico's storage and it will be flashed automatically.

  - This take a minute or so.

- When the /mnt unmounts remove the boot jumper. 

- Reboot and continue to the next phase.. 
 
### Flashing Klpper via USB the first time

- SSH into the Klipper host and CD to klipper
  ```
  cd ~/klipper/
  ```

- Run Make Clean
  ```
  make clean
  ```

- Run Make Menuconfig
  
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - bootloader offset 16Kib Bootloader
    - Com interface USB to CAN bus
    - CAN RX GPIO 1
    - CAN Tx GPIO 0
    - CAN bus speed 500000 <-- Chose your own adventure ( 250000,500000,1000000 )

![Klipper USB to CAN](/img/skr_pico_klipper_usbtocan.png)    
  
 - Run make
  ```
  make
  ``` 

- Get your serial ID
  ```
  ls /dev/serial/by-id
  ```

 - Flash the klipper.bin file to the SKR Pico
  ```
  python3 ~/CanBoot/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d /dev/serial/by-id/{your serial id}
  ```

- Reboot and your Pico should come bacup up in USB to canbridge mode.

### Flashing after Canboot is installed ( this is only for KLipper in USB to can birdge mode )

- Run the canboot flash by UUID to enable CanBoot on USB firmware. 
  ```
  python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u { your uuid}
  ```

- Canboot throw an error but will now appear as a USB device, Get the Canboot serial ID
  ```
  ls /dev/serial/by-id
  '''

 - Build klipper as before ( see above ) 
 - Flash your new klipper bin via Canbus by serial ID
  ```
  python3 ~/CanBoot/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d /dev/serial/by-id/{your serial id}
  ```
  
  - Reboot 
  
## Canboot for SKR Pico in USB mode 
### Note: I have found that Klipper firmware for USB is not cabable of calling the CanBoot bootloader. To re-flash you will need to quickly press the rest the Pico twice to have it enter the CanBoot bootloader.    
- Update to the latest Canboot and Klipper codabase as of 11-8-2022
  - Canboot v0.0.1-20-g600967e or newer
  - Klipper v0.10.0-623-g5b1a6676 or newer
- SSH to  your Klipper host
- CD to Canboot
  ```
  cd ~/CanBoot/
  ```
  
- Run Make Clean
  ```
  make clean
  ```

- Run Make Menuconfig
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - Build CanBoot deployment 16Kib Bootloader
    - Com interface USB

![Canboot in USB mode](/img/skr_pico_canboot_usb.png)

- Run make
  ```
  make
  ``` 
- Connect the Pico by USB to the Klipper Host
- Install boot jumper on the SKR Pico and press the reset button
- Mount the Pico file system
  ```
  sudo mount /dev/sda1 /mnt
  ```

- Copy Canboot.uf2 to the Pico
  ```
  sudo cp ~/CanBoot/out/canboot.uf2 /mnt
  ``` 

- This will copy the firmware image we compiled earlier onto the SKR-Pico's storage and it will be flashed automatically.

  - This take a minute or so.

- When the /mnt unmounts remove the boot jumper. 

- Reboot and continue to the next phase.. 
 
### Flashing Klpper via USB the first time

- SSH into the Klipper host and CD to klipper
  ```
  cd ~/klipper/
  ```

- Run Make Clean
  ```
  make clean
  ```

- Run Make Menuconfig
  
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - bootloader offset 16Kib Bootloader
    - Com interface USB


![Klipper USB to CAN](/img/SKR_Pico_Klipper_canbus_usb.png)    
  
 - Run make
  ```
  make
  ``` 

- Get your serial ID
  ```
  ls /dev/serial/by-id
  ```

 - Flash the klipper.bin file to the SKR Pico
  ```
  python3 ~/CanBoot/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d /dev/serial/by-id/{your serial id}
  ```

- Reboot and your Pico should come back up in USB to canbridge mode.

### Flashing again after Canboot and klipper have been installed

- Press the reset button twice to have the Pico enter the Canboot Firmware. 

- Run ls /dev/serial/by-id and verify the Canboot bootloader is running. 
  ```
  ls /dev/serial/by-id
  ```
  
- Run the canboot flash with your serial ID to burn the firmware. 
  ```
  python3 ~/CanBoot/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d /dev/serial/by-id/{your serial id}
  ```
 
  - Reboot 

## Canboot for SKR Pico in UART mode 
### Note: I have found that Klipper firmware for USB is not cabable of calling the CanBoot bootloader. To re-flash you will need to quickly press the rest the Pico twice to have it enter the CanBoot bootloader.      
- Update to the latest Canboot and Klipper codabase as of 11-8-2022
  - Canboot v0.0.1-20-g600967e or newer
  - Klipper v0.10.0-623-g5b1a6676 or newer
- SSH to  your Klipper host
- CD to Canboot
  ```
  cd ~/CanBoot/
  ```
  
- Run Make Clean
  ```
  make clean
  ```

- Run Make Menuconfig
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - Build CanBoot deployment 16Kib Bootloader
    - Com interface Serial on UART

![Canboot in UART mode](/img/skr_pico_canboot_uart.png)

- Run make
  ```
  make
  ``` 
- Connect the Pico by USB to the Klipper Host
- Install boot jumper on the SKR Pico and press the reset button
- Mount the Pico file system
  ```
  sudo mount /dev/sda1 /mnt
  ```

- Copy Canboot.uf2 to the Pico
  ```
  sudo cp ~/CanBoot/out/canboot.uf2 /mnt
  ``` 

- This will copy the firmware image we compiled earlier onto the SKR-Pico's storage and it will be flashed automatically.

  - This take a minute or so.

- When the /mnt unmounts remove the boot jumper. 

- Reboot and continue to the next phase.. 
 
### Flashing Klpper via UART the first time

- SSH into the Klipper host and CD to klipper
  ```
  cd ~/klipper/
  ```

- Run Make Clean
  ```
  make clean
  ```

- Run Make Menuconfig
  
  ```
  make menuconfig
  ```

- Settings
    - MCU Raspberry Pi RP2040
    - bootloader offset 16Kib Bootloader
    - Com interface Serial on UART


![Klipper UART](/img/skr_pico_klipper_canbus_uart.png)    
  
 - Run make
  ```
  make
  ``` 

 - Connect the SKR Pico UART pins to the klipper host.
 - Flash the klipper.bin file to the SKR Pico
  ```
  python3 ~/CanBoot/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d  /dev/ttyAMA0
  ```

- Reboot and your Pico should come back up in Serial UART mode.

### Flashing again after Canboot and klipper has been installed

- Press the reset button twice to have the Pico enter the Canboot Firmware. 
- Run the canboot flash. 
  ```
  python3 ~/CanBoot/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d /dev/ttyAMA0
  ```
 
  - Reboot 


# Links:

[Rootiest SKR Pico Canbus guide](https://github.com/rootiest/zippy-klipper_config/blob/master/guides/Guide-pico_can.md)

[Arksine Canboot](https://github.com/Arksine/CanBoot)
