# Da Vinci 1.0 Pro + Raspberry Pi + Klipper

Versão em português: [README.pt.md](README.pt.md)

#### I used:
- Raspberry Pi 3 Model B+ 2017 (any model can work);

- SD card (16GB) to boot MainsailOS (Klipper) on the Raspberry Pi;

- Laptop (Ubuntu 22.04) to configure the Raspberry Pi (or you can use an HDMI monitor);
  
- Da Vinci 1.0 Pro 3D printer (other versions may vary);
  
- Arduino USB cable.

## 1. Install Raspberry Pi Imager and flash MainsailOS 3.0.0 to your SD card

Raspberry Pi Imager: https://www.raspberrypi.com/software/

Linux:
```bash
sudo apt update
sudo apt install rpi-imager
rpi-imager
```
#### After installation, select: 

  CHOOSE OS --> Other specific-purpose OS --> 3D printing --> MainsailOS 3.0.0 - Raspberry Pi (32-bit)

#### Select your SD card:

CHOOSE STORAGE

#### Before flashing, open the settings:
![raspberry pi imager](raspberry_pi_imager_config.png)

##### NOTE: 

Some Raspberry Pi models may not connect to 5GHz networks; prefer 2.4GHz;

In the settings, I used the default username and hostname because custom values were not being saved correctly. (password: raspberry);

#### With the SD card flashed, insert it into the Raspberry Pi and power it on.

## 2. Configure the Raspberry Pi

For this step, I used SSH to access the Raspberry Pi remotely, but you can also use a monitor and keyboard.

#### First connect to the same network as your Raspberry Pi (it must have internet access for this step)

#### Access your Raspberry Pi

Linux/Mac/Windows:
```bash
ssh <username>@<hostname>.local
```
*default user (user: pi  password: raspberry)*

or

Find your Raspberry Pi IP:
```bash
sudo apt install nmap -y
nmap -sn 192.168.1.0/24
```
And connect using the IP:
```bash
ssh <username>@<RASPBERRY-PI-IP>
```

#### Update the OS

    sudo apt update
    sudo apt upgrade 
    sudo reboot

#### You can also access

Settings:

    sudo raspi-config

Manage Wi-Fi connections:

    sudo nmtui

## 3. Build the printer board firmware (Klipper)

#### Build firmware for your printer
```bash
cd ~/klipper
make menuconfig
```
Configure the menu with the following parameters for the **Da Vinci 1.0 Pro**:

- **Micro-controller Architecture:** SAM3/SAM4 (Atmel SAM)

- **Processor model:** SAM4E8E

- **Communication interface:** USB

Press **Q**, then **Y** to save. Then compile the firmware:

```bash
make
```
*The generated file will be saved at ~/klipper/out/klipper.bin*

#### Download the firmware to your PC

Copy to ~/printer_data/config/
```bash
sudo cp ~/klipper/out/klipper.bin ~/printer_data/config/firmware.bin
```
*Note that the file name klipper.bin was changed to firmware.bin, which is how the printer recognizes the firmware file*

Access:
```bash
http://<hostname>.local/
```
Go to Machine and download the firmware.bin file to your PC.

## 4. Put the motherboard in Bootloader Mode (ERASE)

*Do this at your own risk*

1. Turn off the printer and unplug it.
2. Look for two pins, or solder pads, marked SW2 (older versions: J37 or JP1/ERASE)
3. Short these two pins using a jumper
4. Turn on the printer for 5 seconds, then turn it off
5. Remove the short from the two pins
6. Connect the Arduino USB cable to the board and your PC, then turn the printer on again

#### Check if it worked

Is the USB cable connected?

    ls /dev/ttyACM*

Test with *bossac*

    sudo bossac -p /dev/ttyACM0 -i

It should show information about your board MCU.

If the LCD screen displays static blocks, it probably worked too.

## 5. Flash Klipper to the printer board
```bash
sudo bossac -p /dev/ttyACM0 -e -w -v -b -R ~/YOUR-PATH-TO/firmware.bin
```
The output should not contain errors.

## 6. Create the printer.cfg configuration file

#### For this step, connect your printer to your Raspberry Pi

#### Get the MCU ID on the Raspberry Pi

    ls /dev/serial/by-id/*

#### Access MainsailOS on your PC

http://<hostname>.local/

Go to Machine, create a new file, and name it **printer.cfg**

Open printer.cfg, copy and paste everything from [DaVinci_1_0_Pro_printer.cfg](DaVinci_1_0_Pro_printer.cfg)

*This file is configured for Da Vinci 1.0 Pro printers*


At the beginning of the file, change the serial value highlighted in the image below to the serial ID obtained above.
![printer.cfg](printer.cfg.png)

With this, it should already be possible to communicate with the MCU.

## 6. Configure printer.cfg

Finally, two more steps were needed to actually get the printer working.

#### First error:
![error_heater_bed.png](error_heater_bed.png)

Go to heater_bed and uncomment the lines below:

    [heater_bed]
    heater_pin: PD12
    sensor_type: DaVinciBed
    sensor_pin: PA20
    min_temp: -20
    max_temp: 130
    control: pid
    pid_kp: 76.125
    pid_ki: 1.591
    pid_kd: 910.651

#### Second error:
![error_stepper_z.png](error_stepper_z.png)

Go to stepper_z and uncomment the lines below:

    [stepper_z]
    step_pin: PC20
    dir_pin: !PD7
    enable_pin: PD6
    microsteps: 32
    full_steps_per_rotation: 200
    rotation_distance: 1.25
    endstop_pin: ^PD9
    position_endstop: -6.102
    position_min: -6.125
    position_max: 201.25
    homing_speed: 4
    homing_retract_dist: 0
    homing_retract_speed: 3
    second_homing_speed: 2

With this, it should be possible to test the printer, controlling bed temperature and extruder motors.

## Credits:

Jo Info Tech: [How to Install Klipper Firmware on the Printer](https://www.youtube.com/watch?v=Tgbp7A-5afQ)

DaVinci-10: [DaVinci1.0_Klipper](https://github.com/DaVinci-10/DaVinci1.0_Klipper/)
