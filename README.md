# USB_Serial_Bridge

### USB Serial Bridge for STM32F103C8Tx based ARM modules

Given below is the readme file from the original code which has been ported to work on STM32F103C8Tx modules

### README

  ******************** (C) COPYRIGHT 2016 STMicroelectronics *******************
  * @file    USB_Serial_Bridge/readme.txt 
  * @author  MCD Application Team
  * @version V1.4.0
  * @date    29-April-2016
  * @brief   Description of the USB Device CDC example.
  ******************************************************************************
  * @attention
  *
  * <h2><center>&copy; Copyright © 2016 STMicroelectronics International N.V. 
  * All rights reserved.</center></h2>
  *
  * Redistribution and use in source and binary forms, with or without 
  * modification, are permitted, provided that the following conditions are met:
  *
  * 1. Redistribution of source code must retain the above copyright notice, 
  *    this list of conditions and the following disclaimer.
  * 2. Redistributions in binary form must reproduce the above copyright notice,
  *    this list of conditions and the following disclaimer in the documentation
  *    and/or other materials provided with the distribution.
  * 3. Neither the name of STMicroelectronics nor the names of other 
  *    contributors to this software may be used to endorse or promote products 
  *    derived from this software without specific written permission.
  * 4. This software, including modifications and/or derivative works of this 
  *    software, must execute solely and exclusively on microcontroller or
  *    microprocessor devices manufactured by or for STMicroelectronics.
  * 5. Redistribution and use of this software other than as permitted under 
  *    this license is void and will automatically terminate your rights under 
  *    this license. 
  *
  * THIS SOFTWARE IS PROVIDED BY STMICROELECTRONICS AND CONTRIBUTORS "AS IS" 
  * AND ANY EXPRESS, IMPLIED OR STATUTORY WARRANTIES, INCLUDING, BUT NOT 
  * LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A 
  * PARTICULAR PURPOSE AND NON-INFRINGEMENT OF THIRD PARTY INTELLECTUAL PROPERTY
  * RIGHTS ARE DISCLAIMED TO THE FULLEST EXTENT PERMITTED BY LAW. IN NO EVENT 
  * SHALL STMICROELECTRONICS OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
  * INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
  * LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, 
  * OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF 
  * LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING 
  * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE,
  * EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  *
  ******************************************************************************


#### Example Description 

This application shows how to use the USB device application based on the Device Communication Class (CDC) 
following the PSTN subprotocol using the USB Device and UART peripherals.

This is a typical example on how to use the STM32F1xx USB Device peripheral where the STM32 MCU
behaves as a USB-to-RS232 bridge following the Virtual COM Port (VCP) implementation.
 - On one side, the STM32 exchanges data with a PC host through USB interface in Device mode.
 - On the other side, the STM32 exchanges data with other devices (same host, other host,
   other devices…) through the UART interface (RS232).

At the beginning of the main program the HAL_Init() function is called to reset all the peripherals,
initialize the Flash interface and the systick. The user is provided with the SystemClock_Config()
function to configure the system clock (SYSCLK) to run at 72 MHz.
The Full Speed (FS) USB module uses internally a 48-MHz clock, which is generated from an integrated PLL.

When the VCP application is started, the STM32 MCU is enumerated as serial communication port and is
configured in the same way (baudrate, data format, parity, stop bit) as it would configure a standard 
COM port. The 7-bit data length with no parity control is NOT supported.

During enumeration phase, three communication pipes "endpoints" are declared in the CDC class
implementation (PSTN sub-class):
 - 1 x Bulk IN endpoint for receiving data from STM32 device to PC host:
   When data are received over UART they are saved in the buffer "UserTxBuffer". Periodically, in a 
   timer callback the state of the buffer "UserTxBuffer" is checked. If there are available data, they
   are transmitted in response to IN token otherwise it is NAKed.
   The polling period depends on "CDC_POLLING_INTERVAL" value.
    
 - 1 x Bulk OUT endpoint for transmitting data from PC host to STM32 device:
   When data are received through this endpoint they are saved in the buffer "UserRxBuffer" then they
   are transmitted over UART using interrupt mode and in meanwhile the OUT endpoint is NAKed.
   Once the transmission is over, the OUT endpoint is prepared to receive next packet in
   HAL_UART_TxCpltCallback().
    
 - 1 x Interrupt IN endpoint for setting and getting serial-port parameters:
   When control setup is received, the corresponding request is executed in CDC_Itf_Control().
   In this application, two requests are implemented:
    - Set line: Set the bit rate, number of Stop bits, parity, and number of data bits 
    - Get line: Get the bit rate, number of Stop bits, parity, and number of data bits
   The other requests (send break, control line state) are not implemented.

*  Receiving data over UART is handled by interrupt while transmitting is handled by DMA allowing
      hence the application to receive data at the same time it is transmitting another data (full- 
      duplex feature).

The support of the VCP interface is managed through the ST Virtual COM Port driver available for 
download from www.st.com.

*  The user has to check the list of the COM ports in Device Manager to find out the number of the
      COM ports that have been assigned (by OS) to the VCP interface.

This application uses UART as a communication interface. The UART instance and associated resources
(GPIO, NVIC) can be tailored in "usbd_cdc_interface.h" header file according to your hardware 
configuration. Moreover, this application can be customized to communicate with interfaces other than UART.
For that purpose a template CDC interface is provided in: 
Middlewares/ST/STM32_USB_Device_Library/Class/CDC/Src directory.

To run this application, the user can use one of the following configurations:

 - Configuration 1: 
   Connect USB cable to host and UART (RS232) to a different host (PC or other device) or to same host.
   In this case, you can open two hyperterminals to send/receive data to/from host to/from device.
   Hyperterminal is configured as follows:
   - Data Length = 8 Bits
   - One Stop Bit
   - None parity
   - BaudRate = 115200 baud
   - Flow control: None 
    
 - Configuration 2: 
   Connect USB cable to Host and connect UART TX pin to UART RX pin on the STM3210C-EVAL board
   (Loopback mode). In this case, you can open one terminal (relative to USB com port or UART com port)
   and all data sent from this terminal will be received by the same terminal in loopback mode.
   This mode is useful for test and performance measurements.

*  Care must be taken when using HAL_Delay(), this function provides accurate delay (in milliseconds)
      based on variable incremented in SysTick ISR. This implies that if HAL_Delay() is called from
      a peripheral ISR process, then the SysTick interrupt must have higher priority (numerically lower)
      than the peripheral interrupt. Otherwise the caller ISR process will be blocked.
      To change the SysTick interrupt priority you have to use HAL_NVIC_SetPriority() function.
      
*  The application need to ensure that the SysTick time base is always set to 1 millisecond
      to have correct HAL operation.

*  To reduce the example footprint, the toolchain dynamic allocation is replaced by a static allocation
      by returning the address of a pre-defined static buffer with the CDC class structure size. 
	  
#### Directory contents 

  - USB_Serial_Bridge/Src/main.c                  Main program
  - USB_Serial_Bridge/Src/system_stm32f1xx.c      STM32F1xx system clock configuration file
  - USB_Serial_Bridge/Src/stm32f1xx_it.c          Interrupt handlers
  - USB_Serial_Bridge/Src/stm32f1xx_hal_msp.c     HAL MSP module
  - USB_Serial_Bridge/Src/usbd_cdc_if.c    USBD CDC interface
  - USB_Serial_Bridge/Src/usbd_conf.c             General low level driver configuration
  - USB_Serial_Bridge/Src/usbd_desc.c             USB device CDC descriptor
  - USB_Serial_Bridge/Inc/main.h                  Main program header file
  - USB_Serial_Bridge/Inc/stm32f1xx_it.h          Interrupt handlers header file
  - USB_Serial_Bridge/Inc/stm32f1xx_hal_conf.h    HAL configuration file
  - USB_Serial_Bridge/Inc/usbd_conf.h             USB device driver Configuration file
  - USB_Serial_Bridge/Inc/usbd_desc.h             USB device descriptor header file
  - USB_Serial_Bridge/Inc/usbd_cdc_interface.h    USBD CDC interface header file  


#### Hardware and Software environment

  - This example runs on STM32F1xx devices.
    
  - This example has been tested with a STM3210C-EVAL board embedding
    a STM32F107x device and can be easily tailored to any other supported device
    and development board.

  - STM3210C-EVAL Set-up
    - Connect the STM3210C-EVAL board to the PC through 'USB Type micro A-Male to A-Male' cable.
    - Connect the STM3210C-EVAL board to the PC (or to another evaluation board) through RS232 (USART)
      serial cable CN6 connector.  
    - For loopback mode test: remove RS232 cable on CN6 and connect directly USART TX and RX pins:
      GPIOD GPIO_PIN_5 and GPIO_PIN_6.

#### How to use it ?

In order to make the program work, you must do the following :
 - Open your preferred toolchain
 - Rebuild all files and load your image into target memory
 - Run the example
 - Install the USB virtual COM port driver
 - Find out the number of the COM port assigned to the STM32 CDC device
 - Open a serial terminal application and start the communication

 * <h3><center>&copy; COPYRIGHT STMicroelectronics</center></h3>
