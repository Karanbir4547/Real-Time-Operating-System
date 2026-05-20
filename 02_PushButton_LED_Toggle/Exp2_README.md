# Experiment 2 — GPIO Digital Input: Push Button Controlled LED Toggle

## Objective
Interface a push button as a digital input device and demonstrate LED state control by toggling it on each confirmed button press.

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

This experiment extends the GPIO output knowledge from Experiment 1 by introducing **GPIO input**, where the onboard user button controls the onboard LED.

### GPIO Configured as Input
When a GPIO pin is placed in **digital input** mode, the microcontroller passively reads the voltage present on that pin and interprets it as logic `1` (HIGH) or `0` (LOW). Unlike output mode, the MCU does not drive the pin — it simply observes it.

### Board Pin Assignments

| Function | Arduino Label | MCU Pin | Default Logic Level |
|----------|---------------|---------|---------------------|
| User LED (LD2) | D13 | **PA5** | Output, Push-Pull |
| User Button (B1) | — | **PC13** | Input, Active LOW |

### Understanding Active-Low Button Logic
The onboard user button **B1** on the Nucleo-F446RE connects to **PC13**, which has a hardware pull-up resistor already present on the board. This means:
- Button **not pressed** — PC13 reads **HIGH (1)**
- Button **pressed** — PC13 reads **LOW (0)**

This behavior is referred to as **active-low** — the signal transitions low when the intended event (a button press) takes place.

### What is Contact Bounce?
Mechanical push buttons do not produce a single clean transition. When pressed, the metal contacts physically bounce several times before settling, generating a burst of rapid HIGH/LOW transitions over a short period. This is known as **contact bounce**, and it can cause the MCU to register one physical press as several separate events.

A simple **software debounce** is used here — after detecting a press, a short `HAL_Delay(200)` pause allows the signal to stabilize before the next read.

### HAL Functions Used

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_ReadPin(GPIOx, GPIO_Pin)` | Returns the current logic level of an input pin (`0` or `1`) |
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Inverts the current state of a GPIO output pin |
| `HAL_Delay(ms)` | Blocking millisecond pause — used here for software debounce |

---

## STM32CubeMX Setup

### Step 1 — MCU Selection
- Open STM32CubeMX and go to **ACCESS TO MCU SELECTOR**
- Search for and select: **STM32F446RETx**
- Click **Start Project**

### Step 2 — GPIO Configuration

**PA5 — LED Output:**

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Output Push Pull |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| Maximum Output Speed | Low |
| User Label | LED (optional) |

**PC13 — Push Button Input:**

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Input mode |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| User Label | BTN (optional) |

> The Nucleo board already has a hardware pull-up on PC13, so no internal pull-up needs to be configured in CubeMX.

### Step 3 — Clock Source (RCC)
- Go to **System Core -> RCC**
- High Speed Clock (HSE): **BYPASS Clock Source**

### Step 4 — Clock Configuration

| Clock Domain | Prescaler | Frequency |
|--------------|-----------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL output | 180 MHz |
| AHB (HCLK) | /1 | 180 MHz |
| APB1 | /4 | 45 MHz |
| APB2 | /2 | 90 MHz |

### Step 5 — Project Manager
- Enter a **Project Name** (e.g., `Experiment_2`)
- Set Toolchain/IDE to: **STM32CubeIDE**
- Click **Generate Code** then **Open Project**

### Step 6 — Post-Build Output Settings
- Right-click the project and go to **Properties -> C/C++ Build -> Settings -> MCU Post Build Outputs**
- Enable **Convert to Binary File (.bin)**
- Enable **Convert to Intel Hex File (.hex)**
- Click **Apply and Close**

---

## Source Code

Open `Core/Src/main.c` and insert the following code inside the `while(1)` loop between the user code markers:

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    // Read PC13 — Active LOW: a reading of 0 means the button is pressed
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == 0)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Invert LED state on PA5
        HAL_Delay(200);                           // Debounce delay
    }
    /* USER CODE END WHILE */
}
```

### Program Flow

```
Loop begins
    |
    v
Sample PC13
    |
    +-- HIGH (button not pressed) --> do nothing --> loop again
    |
    +-- LOW (button held down)
            |
            v
        Toggle PA5 (LED state flips)
            |
            v
        Wait 200 ms  <- lets contact bounce settle
            |
            v
        Loop again
```

> Keep all user code strictly between the `USER CODE BEGIN` and `USER CODE END` markers to avoid it being erased during CubeMX regeneration.

---

## Build and Run

1. Connect the Nucleo board via USB
2. Click **Build** (hammer icon) and confirm **0 errors**
3. Click **Debug**, then click **Switch** in the perspective dialog
4. Click **Resume (Play)** to start execution
5. Press the blue **B1 user button** on the board — the green LED **LD2** should toggle on every press

---

## Recorded Observations

### Button Press vs. LED State

| Trial No. | Button Action | PC13 State | LED State Before | LED State After | Remark |
|-----------|---------------|------------|------------------|-----------------|--------|
| 1 | No press (initial) | HIGH | OFF | OFF | No change |
| 2 | 1st Press | LOW | OFF | ON | LED turns ON |
| 3 | Release | HIGH | ON | ON | LED holds ON |
| 4 | 2nd Press | LOW | ON | OFF | LED turns OFF |
| 5 | 3rd Press | LOW | OFF | ON | LED turns ON again |
| 6 | Rapid Press | Bouncing | Varies | Toggles | LED may flicker at high speed |

**Key observations:**
- The LED state is **retained** — it holds the new value even after the button is released.
- Each confirmed press produces exactly one toggle.
- Very rapid pressing can produce flickering since the 200 ms window may not always be sufficient.

---

## Result

For every valid button press, the LED on PA5 toggled once and retained its state after release. This confirms the correct configuration of **PC13 as a digital input** and **PA5 as a digital output**, along with a functional software-level debounce implemented through `HAL_Delay()`.

---

## Key Takeaways

- A microcontroller has no built-in notion of a button press event — it only reads binary logic levels at a given moment. The programmer decides how to interpret signal transitions.
- **Active-low logic** is widely used in embedded hardware. Always review the board schematic to understand the idle-state polarity of an input signal.
- **Contact bounce** is a physical phenomenon. A single press can produce dozens of false transitions within microseconds. A delay-based debounce is the simplest remedy, though state-machine or timer-based approaches are more reliable.
- The LED in this experiment is **stateful** — unlike the periodic blink in Experiment 1, it stores and holds its last state. This forms the basis for latches and toggles in real embedded control systems.
- Combining GPIO input and output in a single application creates a meaningful stimulus-response loop — the foundational principle behind all embedded control systems.

---

## Project Layout

```
02_PushButton_LED_Toggle/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c          <- User code lives here (inside while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_2.ioc        <- CubeMX pin and clock configuration file
└── README.md               <- This document
```
