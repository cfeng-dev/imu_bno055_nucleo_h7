# BNO055 STM32 library

This project demonstrates how to use the **STM32 NUCLEO-H7A3ZI-Q** to read data from a **BNO055 IMU sensor** via I2C and transmit the data to a PC over UART. The code is based on the [bno055_stm32 repository](https://github.com/ivyknob/bno055_stm32).

This repository now serves as documentation of how I configured the IMU sensor with the microcontroller. I hope it can help others who are starting with IMU sensors and STM32 development.

## Hardware and Software Configuration

### Step 1: Open the `.ioc` File in STM32CubeIDE

-   Launch **STM32CubeIDE** and open the `.ioc` file for the project.

### Step 2: Enable and Configure I2C1

1. Navigate to **Pinout & Configuration**.
2. Under the peripherals list (A → Z), search for **I2C1**.
3. Enable **I2C1** and configure it in **GPIO Settings** as shown in the image below.

![GPIO Settings for I2C1](/img/gpio_settings_I2C1.png)

#### Note:

The default pins for **I2C1** were **PB6 (I2C1_SCL)** and **PB7 (I2C1_SDA)**, but these pins did not work in my setup.  
So I changed the pins to:

-   **PB8 (I2C1_SCL)**
-   **PB9 (I2C1_SDA)**

### Step 3: Adjust Code Generation Settings

1. Go to **Project Manager** → **Code Generator**.
2. Uncheck the box for **Generate peripheral initialization as a pair of `.c/.h` files per peripheral**.

### Step 4: Save the Configuration

-   Press `Ctrl + S` to save the `.ioc` file, which will regenerate the code with the updated configuration.

### Step 5: Add Flags for Float Support in printf and scanf

To enable floating-point support in `printf` and `scanf`, follow these steps:

1. Go to **Project** → **Properties** in STM32CubeIDE.
2. Navigate to **C/C++ Build** → **Settings**.
3. Under the **Tool Settings** tab, find **Miscellaneous** (under **MCU GCC Linker**).
4. In the **Other Flags** field, add the following flags:
    - `-u _printf_float`
    - `-u _scanf_float`
5. Click **Apply and Close** to save the changes.

This step ensures that floating-point values can be properly handled in formatted input/output functions like `printf` and `scanf`.

### Step 6: Add the Required Files to Your Project

Add these files to your project:

-   Place `bno055.c` in the **Core/Src** folder.
-   Place `bno055.h` and `bno055_stm32.h` in the **Core/Inc** folder.

### Step 7: Import the Library in `main.c`

1. Open `main.c` in STM32CubeIDE.
2. Add the following includes inside the `/* USER CODE BEGIN Includes */` section:

```c
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include "string.h"
#include "bno055_stm32.h"
/* USER CODE END Includes */
```

### Step 8: Add IMU Setup Code

-   In the `/_ USER CODE BEGIN 2 _/` section of `main.c`, add the following setup code to initialize the BNO055 IMU sensor:

```c
/* USER CODE BEGIN 2 */
bno055_assignI2C(&hi2c1);
bno055_setup();
bno055_setOperationModeNDOF();
/* USER CODE END 2 */
```

### Step 9: Write a Function to Fetch and Transmit IMU Data

1.  Inside the `/_ USER CODE BEGIN 0 _/` section of `main.c`, add the following function:

```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
void send_imu_data() {
    char buffer[100];

    // Get Orientation and Linear Acceleration data
    bno055_vector_t orient = bno055_getVectorEuler();
    bno055_vector_t linear = bno055_getVectorLinearAccel();

    // IMU data
    snprintf(buffer, sizeof(buffer),
            "[IMU Data] Orient: x= %.2f, y= %.2f, z= %.2f ; Linear: x= %.2f, y= %.2f, z= %.2f\r\n",
            orient.x, orient.y, orient.z, linear.x, linear.y, linear.z);

    HAL_UART_Transmit(&huart3, (uint8_t*) buffer, strlen(buffer), HAL_MAX_DELAY);
    HAL_Delay(100); // 10 Hz
}
/* USER CODE END 0 */
```

2. Call the function inside the main loop to continuously fetch and transmit data:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    /* USER CODE END WHILE */
    send_imu_data();
    /* USER CODE BEGIN 3 */
}
/* USER CODE END 3 */
```

### Step 10: Circuit Connections

To connect the **BNO055 IMU sensor** to the **STM32 NUCLEO-H7A3ZI-Q**, use the following wiring:

| **BNO055 Pin** | **NUCLEO Pin** |
| -------------- | -------------- |
| Vin            | 3V3            |
| GND            | GND            |
| SDA            | SDA (PB9)      |
| SCL            | SCL (PB8)      |

Ensure the connections are secure to avoid communication issues. The image below illustrates the wiring:

![Circuit Diagram for BNO055 and STM32 NUCLEO-H7A3ZI-Q](/img/imu_circuit_diagram.png)

For more detailed information about the external header connections of the **NUCLEO-H7A3ZI-Q**, refer to the image below:

![NUCLEO-H7A3ZI-Q External Header Connections](/img/extension_connectors_stm32_h7.png)

## Tools

-   **IDE**: STM32CubeIDE (1.16.1)
-   **Microcontroller**: [STM32 NUCLEO-H7A3ZI-Q](https://www.st.com/en/evaluation-tools/nucleo-h7a3zi-q.html)
-   **IMU Sensor**: [Adafruit 9-DOF Absolute Orientation IMU Fusion Breakout - BNO055](https://www.adafruit.com/product/2472)

## Third-Party Code

This project uses code from the [bno055_stm32 repository](https://github.com/ivyknob/bno055_stm32), licensed under the MIT License. The original copyright belongs to Ivy Knob and contributors.
