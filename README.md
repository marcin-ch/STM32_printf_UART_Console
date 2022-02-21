# Overview
This is a double-check of ST article named [How to redirect the printf function to a UART for debug messages](https://community.st.com/s/article/how-to-redirect-the-printf-function-to-a-uart-for-debug-messages). Article is based on Nucleo-G070RB, I am using [STM32WB5MM-DK](https://www.st.com/en/evaluation-tools/stm32wb5mm-dk.html).

Following this article, you get as follows:
1. `printf` output is redirected to UART of STM32 target MCU which is connected to UART of ST-Link (programmer/debugger embedded on discovery kit)
    * connection between target MCU and ST-Link has been show in [Workflow](#workflow) (point 8.)
2. `printf` output is transmitted by ST-Link to the PC and displayed in terminal emulator application such as [Tera Term](https://ttssh2.osdn.jp/index.html.en)

**21.02.2022 update:** oh huh! There is an example with the same functionality available straight away. It is even extended with taking input from keyboard and displaying it in console. To check it out:
1. In STM32CubeIDE hit **File** -> **Import** -> **General** -> **Import STM32Cube Example**
2. In **Example Selector** tab specify the board you working on, list of examples dedicated to your board will be showed up
3. Look for *UART_Console*, hit **Next**, **Next** again and **Finish** finally
4. In **Project Explorer** go to */Application/User/main.c* and look for `printf` functions

What I have noticed about examples imported in such way, when you are editing *.ioc* file and re-generating the code, project simply breaks down throwing a lot of errors an warnings. I have not found any quick fix for this. Therefore, caution is strongly advised!

# Hardware
* [STM32WB5MM-DK](https://www.st.com/en/evaluation-tools/stm32wb5mm-dk.html)

# Software
* [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html) 1.8.0
* [Tera Term](https://ttssh2.osdn.jp/index.html.en) terminal emulator application

# Workflow
1. Connect your evaluation board to PC
2. Open STM32CubeIDE and provide workspace name
3. Hit **File** -> **New** -> **STM32 Project**
4. Switch to **Board Selector** tab and provide the name of your board (in my case [STM32WB5MM-DK](https://www.st.com/en/evaluation-tools/stm32wb5mm-dk.html))<br>
![0_STM32CubeIDE_board_select_resized.PNG](/doc/github_readme_images/0_STM32CubeIDE_board_select_resized.PNG)
5. Select your board from the list and hit **Next**
6. Define **Project Name**, for example *UART_Console*, then hit **Next** and **Finish** (all settings default)<br>
![1_STM32CubeIDE_project_name.PNG](/doc/github_readme_images/1_STM32CubeIDE_project_name.PNG)
![2_STM32CubeIDE_project_name_cont.PNG](/doc/github_readme_images/2_STM32CubeIDE_project_name_cont.PNG)
7. Question will pop up whether to initialize all peripherals with their default mode, hit **No** as only required peripherals will be set in next steps<br>
![3_STM32CubeIDE_periph_init.PNG](/doc/github_readme_images/3_STM32CubeIDE_periph_init.PNG)
    * as a side note: in reference to [STM32CubeIDE user guide](https://www.st.com/resource/en/user_manual/um2609-stm32cubeide-user-guide-stmicroelectronics.pdf) (page 41.) there is a suggestion to hit **Yes** (please see below) however in this project answering **No** is totally fine
    > Press **Yes** since it is a good practice to get software needed to initialize the peripherals
8. *UART_Console.ioc* file should be opened automatically now. Go to **Pinout & Configuration** -> **System view** tab and you should see as follows<br>
![4_STM32CubeIDE_system_view.PNG](/doc/github_readme_images/4_STM32CubeIDE_system_view.PNG)
9. Type **USART1** in search box on the left hand side, set the mode to **Asynchronous** and double-check whether selected pins are **PB6** and **PB7**<br>
![5_STM32CubeIDE_usart1.PNG](/doc/github_readme_images/5_STM32CubeIDE_usart1.PNG)
    * why USART1? This is because pins **PB6** and **PB7** of target MCU are connected to ST-Link MCU and they can be configured as **USART1_TX** and **USART1_RX** respectively, ST-Link transmits messages to PC (as stated at the begining of this doc in [Overview](#overview))
      * please see below drawing which has been prepared based on STM32WB5MM-DK [schematic](https://www.st.com/resource/en/schematic_pack/mb1292-wb5mm-b01_schematic.pdf)<br>
![6_STM32WB5MMDK_schematic.PNG](/doc/github_readme_images/6_STM32WB5MMDK_schematic.PNG)
    * all further parameters default
      * please double-check **Parameters Settings** -> **Basic Parameters** as they will be needed afterwards<br>
![7_STM32CubeIDE_usart1_params.PNG](/doc/github_readme_images/7_STM32CubeIDE_usart1_params.PNG)
10. Save *UART_Console.ioc* (question about generating the code will pop up, choose **Yes**) or hit **Project** -> **Generate Code**
11. Open *Core/Src/main.c* and update the code as follows
    * between `/* USER CODE BEGIN PFP */` and `/* USER CODE END PFP */`
    ```c
    /* USER CODE BEGIN PFP */
    #define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
    /* USER CODE END PFP */
    ```
    * in ``while`` loop in `main` function
    ```c
    while (1)
    {
        printf("printf message TEST\n\r");
        HAL_Delay(2000);
        /* USER CODE END WHILE */

        /* USER CODE BEGIN 3 */
    }
    ```
    * between `/* USER CODE BEGIN 4 */` and `/* USER CODE END 4 */`
    ```c
    /* USER CODE BEGIN 4 */
    /**
      * @brief  Retargets the C library printf function to the USART.
      * @param  None
      * @retval None
      */
    PUTCHAR_PROTOTYPE
    {
    /* Place your implementation of fputc here */
    /* e.g. write a character to the USART1 and Loop until the end of transmission */
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);

    return ch;
    }
    /* USER CODE END 4 */
    ```
12. Select the project in **Project Explorer**, hit **Project** -> **Build All** and then **Run** -> **Debug As** -> **STM32 Cortex-M C/C++ Application** (wait a while, once flashing is done hit `Ctrl`+`F2` to terminate the debug session)<br>
![7a_STM32CubeIDE_build.PNG](/doc/github_readme_images/7a_STM32CubeIDE_build.PNG)
![7b_STM32CubeIDE_debug.PNG](/doc/github_readme_images/7b_STM32CubeIDE_debug.PNG)
13. Open Tera Term and follow the steps
    * choose serial port connection to the ST-Link Virtual COM port hitting **File** -> **New Connection**<br>
![8_TeraTerm_serial_config.PNG](/doc/github_readme_images/8_TeraTerm_serial_config.PNG)
    * check connection settings **Setup** -> **Serial port**, they should match the parameters set in point 9.<br>
![9_TeraTerm_connection_config.PNG](/doc/github_readme_images/9_TeraTerm_connection_config.PNG)
14. Observe `printf` output appearing every 2000ms<br>
![10_TeraTerm_output.PNG](/doc/github_readme_images/10_TeraTerm_output.PNG)

# Summary
ST article provides good content and I can still use my favourite debugging method 😍

---
## How to use this repo (source code)
1. Clone the repo or download as *.zip* to your local disc drive
    * when clonning please use below command:
    ```console
    git clone https://github.com/marcin-ch/STM32_printf_UART_Console.git
    ```
2. Open *.project* file (it should be opened in STM32CubeIDE)
3. Hit **Run** -> **Debug As** -> **STM32 Cortex-M C/C++ Application** (wait a while, once flashing is done hit `Ctrl`+`F2` to terminate the debug session)
4. The application should be up and running

## How to use this repo (binary file)
If you just want to check how the project looks like running on the board, you can flash binary file available in */doc/UART_Console.bin*. As STM32F469 Discovery kit enumerates as MSD (Mass Storage Device) when connected to PC through USB cable, you can simply drag-n-drop binary file to your board. Wait few moments when flashing is in progress, reset the board and you should see application working.

---