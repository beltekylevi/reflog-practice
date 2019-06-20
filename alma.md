# Reaction game
*Create a reaction tester with STM32 board*

## Objectives
- Practice STM32's GPIO
- Get familiar with the most basic timer, the SysTick timer
- Initialize a new peripheral, the true random number generator (RNG)
- Become acquainted with the LCD display (and optionally the touchscreen)
- Feel how hard it is to solve a problem without interrupts

## Materials & Resources

| Material | Duration |
|:---------|---------:|
|[STM32 random number generator](https://st-onlinetraining.s3.amazonaws.com/STM32F7-Security_Random_Number_Generator/index.html)| 3:23 |
|[Systick timer](http://www.micromouseonline.com/2016/02/02/systick-configuration-made-easy-on-the-stm32/)| - |
|:bookmark_tabs: [HAL RNG reference (52. section)](https://www.st.com/content/ccc/resource/technical/document/user_manual/45/27/9c/32/76/57/48/b9/DM00189702.pdf/files/DM00189702.pdf/jcr:content/translations/en.DM00189702.pdf#page=726)| - |

## Optional

| Material | Duration |
|:---------|---------:|
|:bookmark_tabs: [RNG block diagram and registers](https://www.st.com/content/ccc/resource/technical/document/reference_manual/c5/cf/ef/52/c0/f1/4b/fa/DM00124865.pdf/files/DM00124865.pdf/jcr:content/translations/en.DM00124865.pdf#page=543)| - |

## Material review

### RNG
There is a true random number generator in the MCU. Read the 52. section of HAL RNG reference for general understanding this unit.
You will see that it is a very simple peripheral.

### Cortex System Timer (SysTick)
Using timers is a huge topic that will come up next week but SysTick is very easy to use for taking simple measurements.
This timer is incrementing a 32bit register every ms so if you can read this register, you are done with the trustworthy measured time problem. 
In this case the reference manual is not a big help. You can use Google of course but the easiest way is the HAL: look inside the `HAL_Delay()` function! Here you can figure out why this function blocks the program running, how you can access to the needed data register and how you can write a delay that does not block the execution of the program.

### Using the LCD display with the BSP API
```c
#include "stm32f7xx.h"
#include "stm32746g_discovery.h"

/* necessary include files */
#include "stm32746g_discovery_lcd.h" // Feel free to look inside and explore!  

static void Error_Handler(void);
static void SystemClock_Config(void);

void LCD_Init()
{
    BSP_LCD_Init();
    BSP_LCD_LayerDefaultInit(1, LCD_FB_START_ADDRESS);
    BSP_LCD_SelectLayer(1);
    BSP_LCD_SetFont(&LCD_DEFAULT_FONT);
    BSP_LCD_SetBackColor(LCD_COLOR_WHITE);
    BSP_LCD_Clear(LCD_COLOR_WHITE);   
}

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    LCD_Init();
    
    BSP_LCD_SetTextColor(LCD_COLOR_RED);
    BSP_LCD_FillCircle(50, 50, 30);

    BSP_LCD_SetTextColor(LCD_COLOR_BLUE);
    BSP_LCD_DisplayStringAt(50, 50, "Hello world!", CENTER_MODE);

    while (1) {
    }
}

static void Error_Handler(void)
{
}

static void SystemClock_Config(void)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

    /**Configure the main internal regulator output voltage */
    __HAL_RCC_PWR_CLK_ENABLE();
    __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

    /**Initializes the CPU, AHB and APB busses clocks */
    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
    RCC_OscInitStruct.HSIState = RCC_HSI_ON;
    RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
    RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
    RCC_OscInitStruct.PLL.PLLM = 8;
    RCC_OscInitStruct.PLL.PLLN = 216;
    RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
    RCC_OscInitStruct.PLL.PLLQ = 2;

    if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
    {
        Error_Handler();
    }

    /**Activate the Over-Drive mode */
    if (HAL_PWREx_EnableOverDrive() != HAL_OK)
    {
        Error_Handler();
    }

    /**Initializes the CPU, AHB and APB busses clocks */
    RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK | RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV4;
    RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV2;

    if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_7) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### Using the touchscreen with the BSP API
```c
#include "stm32f7xx.h"
#include "stm32746g_discovery.h"

/* necessary include files */
#include "stm32746g_discovery_lcd.h"
#include "stm32746g_discovery_ts.h" // Feel free to look inside and explore!

static void Error_Handler(void);
static void SystemClock_Config(void);

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    BSP_LED_Init(LED_GREEN);

    // The LCD display has to be initalized first!
    BSP_TS_Init(BSP_LCD_GetXSize(), BSP_LCD_GetYSize());


    TS_StateTypeDef ts_state;

    while (1) {
        BSP_TS_GetState(&ts_state);
        if (ts_state.touchDetected) {
            BSP_LED_On(LED_GREEN);
        } else {
            BSP_LED_Off(LED_GREEN);
        }
    }
}

// ...
```

## Workshop
You will need to use the following:
- A random number between 1-10, generated by RNG
- The SysTick timer for reliable time measurement
- The LCD display
- A push button and a LED (or the touchscreen)

## The flow of the game

The game requires you to interact with the user. You have two choices for the implementation: you can either use a push button and a LED or the LCD display and the touchscreen.

- The board is waiting for the user to start the game. In this state flash the LED with a frequency of 1 Hz or display a message on the LCD display. 
- The user can start the game with an input (button push/screen touch). When this happened, signal the user that the game has started (turn the LED off/clear the screen).
- Do nothing for a random amount of time between 1 and 10 seconds. Use the random number generator to generate the time interval.
- After the time elapsed, signal the user to react (turn on the LED/display some message) and start measuring the reaction time.
- You need to measure the time that is elapsed between the signal and reaction (button push/screen touch). Use the SysTick timer for this.
- Print out the result for the user and start the game again.
- **Extra:**
    - Measure and average the time between the last 10 button pushes, and print out (continuously) the frequency you reached.
    - Using the LCD display, draw a circle on the screen that the user has to touch for the reaction to be accepted (something like [this](https://codepen.io/cliff538/full/Eslxr)). You can use random colors, random coordinates, random sizes and random shapes!
