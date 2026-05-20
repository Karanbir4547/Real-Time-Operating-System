# Experiment 1 — GPIO Digital Output: LED Blinking via Software Delay

## Objective
Set up a GPIO pin on the **STM32F446RE** microcontroller to function as a digital output and confirm LED blinking behavior by using software-driven delay routines.

---

## Components Required

| Component | Details |
|-----------|---------|
| Microcontroller Board | STM32 Nucleo-F446RE |
| Cable | USB Type-A to Mini-B |
| IDE | STM32CubeIDE |
| Configuration Tool | STM32CubeMX (integrated within CubeIDE) |

---

## Background Theory

The **STM32F446RE** microcontroller provides multiple general-purpose I/O (GPIO) ports — **GPIOA through GPIOH** — each of which can be independently assigned one of four operating modes:

| Mode | Description |
|------|-------------|
| **Input** | Captures an incoming digital signal |
| **Output** | Drives a HIGH or LOW logic level |
| **Alternate Function** | Delegated to on-chip peripherals (UART, SPI, I2C, etc.) |
| **Analog** | Connected to internal ADC/DAC circuitry |

### Why Pin PA5?
On the **Nucleo-F446RE** development board, the green user LED (**LD2**) is physically tied to **Arduino header pin D13**, which maps to microcontroller pin **PA5**. No external LED or wiring is necessary — configuring PA5 as an output pin directly controls LD2.

### Push-Pull Output Mode
PA5 is configured in **push-pull output** mode. In this configuration, the output driver can:
- Actively pull the pin **HIGH** (~3.3 V) — LED turns **ON**
- Actively pull the pin **LOW** (0 V) — LED turns **OFF**

This differs from open-drain mode, which can only pull the signal low and requires an external pull-up resistor to assert a high level.

### HAL Functions Used

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Flips the current logic state of the specified GPIO pin |
| `HAL_Delay(ms)` | Halts execution for the given number of milliseconds using the SysTick timer |

---

## STM32CubeMX Setup

### Step 1 — MCU Selection
- Launch STM32CubeMX and go to **ACCESS TO MCU SELECTOR**
- Search for and select: **STM32F446RETx**
- Click **Start Project**

### Step 2 — GPIO Configuration
- Navigate to **System Core -> GPIO**
- Right-click on pin **PA5** and choose **GPIO_Output**
- Apply the following GPIO settings:

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Output Push Pull |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| Maximum Output Speed | Low |
| User Label | LD2 (optional) |

### Step 3 — Clock Source (RCC)
- Go to **System Core -> RCC**
- Set High Speed Clock (HSE) to: **BYPASS Clock Source**
- This enables the board to accept the external clock signal provided by the ST-Link interface

### Step 4 — Clock Configuration

| Clock Domain | Prescaler | Frequency |
|--------------|-----------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL multiplier | 180 MHz |
| AHB (HCLK) | /1 | 180 MHz |
| APB1 | /4 | 45 MHz |
| APB2 | /2 | 90 MHz |

### Step 5 — Project Manager
- Enter a **Project Name** (e.g., `Experiment_1`)
- Set Toolchain/IDE to: **STM32CubeIDE**
- Click **Generate Code** then **Open Project**

### Step 6 — Post-Build Output Settings
- Right-click the project and go to **Properties -> C/C++ Build -> Settings -> MCU Post Build Outputs**
- Enable **Convert to Binary File (.bin)**
- Enable **Convert to Intel Hex File (.hex)**
- Click **Apply and Close**

---

## Source Code

After code generation, open `Core/Src/main.c` and insert these lines inside the `while(1)` loop between the designated user code markers:

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Invert LED LD2 state on PA5
    HAL_Delay(500);                           // Hold for 500 ms

    /* USER CODE END WHILE */
}
```

> Always write your code between `USER CODE BEGIN` and `USER CODE END` markers. Anything placed outside these blocks will be **overwritten** the next time CubeMX regenerates code.

---

## Build and Run

1. Plug the Nucleo board in via USB
2. Click **Build** (hammer icon) and confirm **0 errors, 0 warnings**
3. Click **Debug**, then click **Switch** when prompted to change perspectives
4. Click **Resume (Play)** to begin execution
5. The onboard LED LD2 should begin blinking at the configured rate

---

## Recorded Observations

The delay value was varied across trials to examine its effect on blink frequency:

| S. No. | GPIO Pin | Mode / Pull | Delay (ms) | Approx. Blink Period (s) | Comment |
|--------|----------|-------------|------------|--------------------------|---------|
| 1 | PA5 | Output, PP, No PU/PD | 100 | 0.1 s | Extremely fast — barely visible to the eye |
| 2 | PA5 | Output, PP, No PU/PD | 200 | 0.3 s | Moderate speed, clearly noticeable |
| 3 | PA5 | Output, PP, No PU/PD | 500 | 1 s | Comfortable pace, well visible |
| 4 | PA5 | Output, PP, No PU/PD | 1000 | 2 s | Slow — both ON and OFF phases are easy to distinguish |

> **Note:** The full blink period is roughly 2 times the delay value, since each ON and OFF phase individually consumes one complete delay interval.

---

## Result

GPIO pin **PA5** on the STM32F446RE Nucleo board was successfully set up as a **push-pull digital output** using STM32CubeIDE. The onboard user LED **LD2** blinked correctly at every tested rate. Adjusting the delay value produced a proportional and clearly visible change in blink frequency, confirming correct GPIO output behavior and system clock configuration.

---

## Key Takeaways

- GPIO pins are multi-mode — a single physical pin can serve as input, output, alternate function, or analog based solely on software settings.
- **Push-pull mode** actively controls both the high and low voltage levels on the pin, making it well suited for directly driving an LED without additional components.
- `HAL_Delay()` is a **blocking call** that uses the SysTick timer. While it counts down, the processor is completely idle and unable to handle any other work — a core limitation of bare-metal super loop programs.
- The **accuracy of `HAL_Delay()`** depends on a correctly configured SYSCLK. A misconfigured clock will result in inaccurate delays.
- The **super loop (`while(1)`)** is the most fundamental program architecture in embedded development — linear, single-task, and continuously running.
- Blink period equals 2 times the delay, because one full ON-OFF cycle consists of two delay intervals.

---

## Project Layout

```
01_GPIO_LED_Blink_SoftwareDelay/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c          <- User code lives here (inside while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_1.ioc        <- CubeMX pin and clock configuration file
└── README.md               <- This document
```
