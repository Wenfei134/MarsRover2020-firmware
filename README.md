# Mars Rover 2020 Firmware Repository
![CI](https://github.com/uwrobotics/MarsRover2020-firmware/workflows/CI/badge.svg)

## Platform: [STM32F446xE](https://www.st.com/resource/en/datasheet/stm32f446re.pdf) / [NUCLEO-F446RE](https://os.mbed.com/platforms/ST-Nucleo-F446RE/)

This repository contains:
- Arm MBED OS 5 SDK source [[mbed-os](https://github.com/uwrobotics/MarsRover2020-firmware/tree/master/mbed-os)]
- Custom and external libaries [[lib](https://github.com/uwrobotics/MarsRover2020-firmware/tree/master/lib)]
- Applications for running on each control board [[apps](https://github.com/uwrobotics/MarsRover2020-firmware/tree/master/apps)]
- Test applications for testing code components [[apps/test_xxxx](https://github.com/uwrobotics/MarsRover2020-firmware/tree/master/apps)]
- Configuration files for each target hardware [[targets](https://github.com/uwrobotics/MarsRover2020-firmware/tree/master/targets)]
- Miscellaneous items [[misc](https://github.com/uwrobotics/MarsRover2020-firmware/tree/alex/add-readme/misc)]
- Github Actions configurations for automatic build testing [[.github](https://github.com/uwrobotics/MarsRover2020-firmware/tree/master/.github)]
- Makefile [[makefile](https://github.com/uwrobotics/MarsRover2020-firmware/tree/master/makefile)]

## Best Contribution Practices and Tips

- Create a branch in the format `yourName/#<issue-number>/featureName` for every feature you are working on
- Rebase onto master and test on hardware before merging into master
- Add a Github Actions build target for your application if it is not a test application
- Create a pull request to merge any branch into master and select everyone else working on the feature as reviewers
  - Name the pull request `Closes #<issue-number>: FeatureTitle`
- When merging a pull request that fixes an issue title the commit `Fixes #issueNumber: FeatureTitle`
- Clean binaries between making changes to the makefile
- There seems to be an annoying mix of CamelCase and snake_case in MBED but just try to be consistent with whatever code is nearby
- Squash when merging pull requests

## UWRT Firmware Development Instructions

1. Download the development toolchain (gcc and make) and serial interface software
   
   For Ubuntu (18.04 preferred)
    - `sudo apt update`
    - `sudo apt install make`
    - `sudo apt install screen can-utils` for serial and CAN interfacing
    - Install/update ARM GCC toolchain:

	      sudo apt autoremove gcc-arm-none-eabi
          wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2
          sudo tar -xvf gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2 -C /opt/
          echo "PATH=\$PATH:/opt/gcc-arm-none-eabi-9-2019-q4-major/bin" >> ~/.bashrc
          export PATH=$PATH:/opt/gcc-arm-none-eabi-9-2019-q4-major/bin 
       **Note:** If you are not using Ubuntu 18.04 and/or bash you may need to modify this script's paths/files. 
	
	For Windows
    - Install [Windows Subsystem for Linux (WSL)](https://linuxconfig.org/how-to-install-ubuntu-18-04-on-windows-10) with Ubuntu 18.04
    - Follow Ubuntu setup instructions (optionally instead of `screen` you can use [PuTTy](https://www.chiark.greenend.org.uk/~sgtatham/putty/), a GUI Windows app)
	
	For Mac
    - Open Command Line
    - Install Homebrew if not installed 
    	`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
    - Download auto-run script, which will auto install with latest version:
    	`brew tap ARMmbed/homebrew-formulae`
    - Install ARM GCC toolchain via HomeBrew:
    	`brew install arm-none-eabi-gcc`
    - Install [ZOC](https://www.emtec.com/zoc/index.html) for serial interfacing

2. Verify the the toolchains were installed correctly

    Open a new Command Prompt / Terminal window and run the following commands:

    `make --version                    # Should be v3.8.x or newer`  
    `arm-none-eabi-gcc --version       # Should be v9.2.x or newer`

3. Download source code

    `git clone https://github.com/uwrobotics/MarsRover2020-firmware.git`  
    `cd MarsRover2020-firmware`

4. Run make with the target application and board

    Ex. Compile the science application for the science board:  
    `make APP=science TARGET=science`

    Ex. Compile the CAN test application for the safety board:  
    `make APP=test-can TARGET=safety`
    
    After compiling an application you should see a message similar to the following:  
    `===== bin file ready to flash: ../build/test_serial/test_serial_nucleo.bin =====`
    
    **Note:** Our makefile automatically detects the number of available execution threads and uses them all to significantly speed up compile time.

5. Deploy onto board (see below for how to connect to a rover control board)

    Find the application .bin file, located in the build/app directory.

	For Ubuntu
		
    - Install libusb `sudo apt install libusb-1.0-0-dev`
    - Drag and Drop .bin file into NODE_F446RC device folder
	
	For Windows
    
    - Drag and Drop .bin file into NODE_F446RC device folder OR if this does not work or debugging is required:
    - Download [st-link utility](http://www.st.com/content/st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-programmers/stsw-link004.html). Scroll down to Get Software
    - Connect USB to nucleo board and open st-link utility
    - Load code by going to Target->Program and browse for .bin file
	
	For Mac
    - Drag and Drop .bin file into NODE_F446RC disk

    After deploying, the Nucleo will begin to flash red and green. Once the LED stays green, power-cycle the board by unplugging and replugging the 5V connector on the Nucleo.

- To clean the project workspace of app and library build files, run `make clean`
- To clean compiled MBED SDK files, run `make clean-mbed`

    
## Using the Nucleo Dev Board to Program the Rover Boards

In order to use the Nucleo development board as a programmer, the two jumpers (black caps) labelled NUCLEO - ST-LINK will need to be removed. This will sever the ST-LINK debugger portion of the Nucleo from the MCU side, allowing it to be used as a general debugger.

The ST-LINK debugger can then be connected via header CN4 (pins 1-5 with 1 nearest to the SWD label) to a rover board debug header (pins should be labelled) to program it according to the following table:

```
+-----------------------+-----------------------------------+
| Nucleo CN4 Pin Number | Rover Board Debug Header Pin Name |
+-----------------------+-----------------------------------+
| 1 (VREF)              | VCC                               |
| 2 (SWCLK)             | CLK                               |
| 3 (GND)               | GND                               |
| 4 (SWDIO)             | IO                                |
| 5 (NRST)              | RST                               |
| 6 (SWO)               | Not Connected                     |
+-----------------------+-----------------------------------+
```
![](https://github.com/uwrobotics/MarsRover2020-firmware/blob/master/misc/NucleoSWDLabels.png)

After deploying the binary to the board, the Nucleo's `LD1` LED will flash red and green. Programming is complete when the LED stays green, so don't powercycle the board before this.

## Serial Communication

The boards can be communicated with through the serial interface exposed through the debug pins. You can use the USB-serial interface built into the Nucleo dev boards to communicate with the control boards by connecting the TX pin to the board's RX pin and the RX pin to the board's TX pin (transmit to recieve and vice versa). 

On Ubuntu
- Run `screen /dev/serial/by-id/usb-STM* 115200` from the terminal. You may need to prepend this with `sudo`.

On Windows
- Device manager, go to Ports (COM & LPT) and find the name of the Nucleo port (ie COM4)
- Open PuTTy, select the Serial radio button, enter the COM port name and the baud rate (default 115200) and click open

## CAN Communication

The boards can also be communicated with over the CAN bus interfaces. You can use a CANable serial USB-CAN dongle to communicate with them from your development computer. Connect the CAN_H, CAN_L, and GND pins of the CANable to the corresponding pins on the board, and the dongle to your computer.

On Ubuntu
- Run `sudo slcand -o -c -s6 /dev/serial/by-id/*CAN*-if00 can0` to set up the CAN interface
    - The flag `-s6` sets the bus speed to 500 kbps
    - The flag `-s8` sets the bus speed to 1 Mbps
- Run `sudo ip link set can0 up` to enable the interface
- Run `cansend can0 999#DEADBEEF` to send a frame to ID 0x999 with payload 0xDEADBEEF
- Run `candump can0` to show all traffic received by can0

See the [CANable Getting Started guide](https://canable.io/getting-started.html) for more information including Windows support.
