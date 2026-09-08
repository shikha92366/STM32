
# STM32CubeIDE Installation

## Overview

STM32CubeIDE is an integrated development environment provided by
STMicroelectronics for developing applications for STM32 microcontrollers.

It provides tools for:

- Writing and editing C/C++ code
- Building STM32 projects
- Configuring STM32 peripherals
- Programming and debugging STM32 microcontrollers

## Download

Download STM32CubeIDE from the official STMicroelectronics website:

https://www.st.com/en/development-tools/stm32cubeide.html

For Windows, select the latest available version.

## Installation Steps

1. Download STM32CubeIDE from the official STMicroelectronics website.
2. Run the downloaded installer.
3. Follow the installation wizard.
4. Accept the license agreement.
5. Choose the installation location.
6. Proceed to the **Choose Components** section.
7. Keep the required drivers selected.
8. Click **Install** and wait for the installation to complete.
9. Launch STM32CubeIDE.


## Download Page
<img width="1917" height="583" alt="stm32cubeide_software installation" src="https://github.com/user-attachments/assets/7c80f71c-46ca-4af2-8423-57d7a9a422a3" /> 

### Installation Components

During the installation, the following drivers were selected:

- **ST-LINK Drivers** – Used for programming and debugging STM32
  microcontrollers through the ST-LINK interface.
- **SEGGER J-Link Drivers** – Provides support for SEGGER J-Link
  debugging/programming hardware.

For the **NUCLEO-F401RE**, the **ST-LINK driver** is the relevant driver
because the board has an onboard ST-LINK programmer/debugger.


## Verification

After installation, launch STM32CubeIDE and verify that the IDE opens
successfully.

The STM32 development environment is now ready for the next step:
creating and programming an STM32 project.
