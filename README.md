# 💧 STM32U575 Ultrasonic Water-Level Monitoring and Alert System

## 1. Project Title

**Ultrasonic Water-Level Monitoring and Alert System using STM32U575ZIT6Q and HC-SR04**

---

## 2. Project Overview

This project is a microcontroller-based **ultrasonic water-level monitoring and alert system** developed using the **STM32U575ZIT6Q NUCLEO development board** and an **HC-SR04 ultrasonic sensor**.

The HC-SR04 measures the distance between the sensor and the water surface. The STM32U575 measures the duration of the sensor's Echo pulse using **TIM1 Input Capture** and converts that measured time into distance.

The system continuously checks the measured distance against the warning thresholds configured in the firmware. When the measured distance falls outside the configured normal range, the system activates a **status LED** and a **5 V active buzzer module** as a warning.

A resistor voltage divider is used between the HC-SR04 Echo output and the STM32 input because the sensor Echo signal is 5 V while the STM32 GPIO input is intended for a lower logic level.

The firmware is written in **C** using the **STM32 HAL library** and is intended to be developed and built using **STM32CubeIDE**.

---

## 3. Main Objectives

The main objectives of this project are:

1. To interface an HC-SR04 ultrasonic sensor with an STM32U575 microcontroller.
2. To generate the required trigger pulse for the ultrasonic sensor.
3. To measure the Echo pulse duration using a hardware timer.
4. To use Timer Input Capture and interrupts for accurate Echo measurement.
5. To calculate the distance from the sensor to the detected surface.
6. To monitor the measured distance continuously.
7. To provide a visual warning using an LED.
8. To provide an audible warning using a buzzer.
9. To demonstrate practical use of GPIO, timers, interrupts and sensor interfacing on an STM32 microcontroller.

---

# 4. Hardware Components

| Component | Quantity | Purpose |
|---|---:|---|
| STM32U575ZIT6Q NUCLEO Board | 1 | Main microcontroller and development platform |
| HC-SR04 Ultrasonic Sensor | 1 | Distance measurement |
| 5 V Active Buzzer Module | 1 | Audible warning |
| 1 kΩ Resistor | 1 | Voltage-divider resistor |
| 2 kΩ Resistor | 1 | Voltage-divider resistor |
| Connecting Wires | As required | Electrical connections |
| 5 V Supply | As required | Sensor/buzzer supply |
| On-board LED (LD2) | 1 | Visual status indication |

---

# 5. Microcontroller

## STM32U575ZIT6Q

The project uses the **NUCLEO-U575ZI-Q** development board containing the **STM32U575ZIT6Q** microcontroller.

The firmware uses:

- GPIO
- TIM1
- Timer Input Capture
- Interrupts
- STM32 HAL
- SysTick/HAL timing functions

The source program initializes the STM32 HAL, configures the system clock, initializes GPIO and TIM1, starts the timer and enables TIM1 Channel 1 Input Capture interrupts. 

---

# 6. Circuit Description

The major blocks of the system are:

```text
                  +----------------------+
                  |   STM32U575ZIT6Q     |
                  |      NUCLEO          |
                  |                      |
                  | PE11 --------------->|---- TRIG
                  |                      |
                  | PE9 <----------------|---- ECHO
                  |       3.3 V          |
                  |                      |
                  | PA5 --------------->|---- Buzzer
                  |                      |
                  | GND -----------------|---- GND
                  +----------------------+
                           |
                           |
                    On-board LED
```

The complete system contains:

```text
             +----------------+
             |    HC-SR04     |
             | Ultrasonic     |
             |    Sensor      |
             +----------------+
               |     |    |
             VCC    TRIG  ECHO
               |     |    |
              5V    PE11   |
                          |
                     Voltage
                      Divider
                    1 kΩ + 2 kΩ
                          |
                          ↓
                         PE9
                          |
                   +-------------+
                   | STM32U575   |
                   +-------------+
                     |         |
                    PA5       LED
                     |
                   Buzzer
```

---

# 7. Pin Connections

## 7.1 HC-SR04 to STM32U575

| HC-SR04 Pin | STM32 / Supply | Description |
|---|---|---|
| VCC | 5 V | Sensor power |
| TRIG | PE11 | Trigger output from STM32 |
| ECHO | PE9 through voltage divider | Echo input to STM32 |
| GND | GND | Common ground |

---

## 7.2 Buzzer to STM32U575

| Buzzer Pin | Connection |
|---|---|
| + | PA5 |
| GND | GND |

The firmware configures PA5 as a push-pull output and controls the buzzer by setting this GPIO HIGH or LOW.

---

## 7.3 Status LED

The project uses a configured status LED output. The firmware turns the status LED ON when the warning condition is detected and turns it OFF during normal operation.

The source comments identify the onboard LED as **LD2 (blue, PB7)**.

---

# 8. HC-SR04 Voltage Divider

## Why is a voltage divider required?

The HC-SR04 Echo signal is a 5 V signal, while the STM32 input is connected as a lower-voltage digital input.

Therefore, the Echo signal is passed through a resistor voltage divider before reaching **PE9**.

The circuit is:

```text
HC-SR04 ECHO
     |
    1 kΩ
     |
     +-----------> STM32 PE9
     |
    2 kΩ
     |
    GND
```

The voltage-divider arrangement is intended to reduce the sensor's Echo voltage to approximately the STM32's 3.3 V logic level.

This protects the microcontroller input from the sensor's higher-voltage Echo signal.

---

# 9. Working Principle

The system works according to the following sequence:

### Step 1 — Generate Trigger Pulse

The STM32 sets the HC-SR04 TRIG pin LOW initially.

It then records the current TIM1 counter value, sets TRIG HIGH, waits for the programmed short trigger interval, and sets TRIG LOW.

The firmware function responsible for this operation is:

```c
HCSR04_Trigger();
```

The source code comments specify a 10-microsecond trigger pulse.

---

### Step 2 — Ultrasonic Transmission

After receiving the trigger pulse, the HC-SR04 sends an ultrasonic burst toward the surface being measured.

In a water-tank application, the ultrasonic wave travels toward the water surface.

---

### Step 3 — Echo Reception

The ultrasonic wave is reflected back toward the HC-SR04.

The sensor keeps its ECHO output active for a duration corresponding to the ultrasonic wave's round-trip travel time.

---

### Step 4 — Rising-Edge Detection

TIM1 Channel 1 is configured for Input Capture.

When the Echo signal changes from LOW to HIGH, the interrupt callback captures the starting timer value:

```c
echo_start = HAL_TIM_ReadCapturedValue(
    htim,
    TIM_CHANNEL_1
);
```

The firmware then changes the capture polarity from rising edge to falling edge.

---

### Step 5 — Falling-Edge Detection

When the Echo signal changes from HIGH to LOW, the second timer value is captured:

```c
echo_end = HAL_TIM_ReadCapturedValue(
    htim,
    TIM_CHANNEL_1
);
```

The difference between the two captured values represents the Echo pulse duration.

---

### Step 6 — Calculate Echo Time

If the timer has not overflowed:

```c
echo_time = echo_end - echo_start;
```

If the timer counter has wrapped around:

```c
echo_time =
    (65535U - echo_start) + echo_end + 1U;
```

This allows the firmware to handle a timer rollover.

---

### Step 7 — Calculate Distance

After a complete Echo measurement is available, the firmware calculates distance using:

```c
distance_cm = (echo_time * 0.0343f) / 2.0f;
```

The factor `0.0343` represents the approximate speed of sound in air in cm/µs.

The division by 2 accounts for the ultrasonic wave traveling:

```text
Sensor → Surface → Sensor
```

Therefore:

```text
Distance = (Echo Time × Speed of Sound) / 2
```

---

### Step 8 — Check Warning Condition

The calculated distance is compared with the thresholds defined in the firmware:

```c
#define HIGH_LEVEL_THRESHOLD  5.0f
#define LOW_LEVEL_THRESHOLD   10000.0f
```

The warning condition is:

```c
if ((distance_cm < HIGH_LEVEL_THRESHOLD) ||
    (distance_cm > LOW_LEVEL_THRESHOLD))
```

Therefore, the current firmware treats a measurement below the configured high-level threshold or above the configured low-level threshold as a warning condition.

These threshold values are firmware parameters and can be modified according to the actual tank, sensor position and desired application range.

---

### Step 9 — Activate Warning

When the warning condition is detected:

- Status LED is turned ON.
- Buzzer is activated.
- The buzzer activation time is recorded.

The buzzer duration is configured as:

```c
#define BUZZER_ON_MS 5000
```

which represents 5000 milliseconds, or 5 seconds.

---

### Step 10 — Normal Condition

When the measured distance is inside the configured normal range:

- Status LED is turned OFF.
- Buzzer is turned OFF.
- `buzzer_active` is cleared.

---

### Step 11 — Repeat Measurement

After processing the current measurement, the program waits:

```c
HAL_Delay(100);
```

and then starts another measurement.

Thus, the system continuously monitors the sensor.

---

# 10. System Block Diagram

```text
                  +----------------------+
                  |      5 V Supply      |
                  +----------+-----------+
                             |
                 +-----------+-----------+
                 |                       |
                 ↓                       ↓
          +-------------+         +-------------+
          |   HC-SR04   |         |   Buzzer    |
          |  Ultrasonic |         | 5 V Active  |
          |   Sensor    |         |   Module    |
          +------+------+         +------+------+
                 |                       ↑
             ECHO|                       |
                 ↓                       |
          +-------------+                |
          |   Voltage   |                |
          |   Divider   |                |
          | 1 kΩ + 2 kΩ |                |
          +------+------+                |
                 |                       |
                 ↓                       |
              PE9|                       |PA5
        +----------------------------------------+
        |                                        |
        |            STM32U575ZIT6Q              |
        |                                        |
        | PE11 ------------------> TRIG          |
        | PE9  <------------------ ECHO          |
        | PA5  ------------------> BUZZER        |
        |                                        |
        | Status LED ----------------> Warning   |
        +----------------------------------------+
```

---

# 11. Software Architecture

The software is organized around three major operations:

```text
+--------------------------+
|      System Startup      |
+------------+-------------+
             |
             ↓
+--------------------------+
| GPIO and TIM1 Init       |
+------------+-------------+
             |
             ↓
+--------------------------+
| Start Timer + Interrupt  |
+------------+-------------+
             |
             ↓
+--------------------------+
| Main Measurement Loop    |
+------------+-------------+
             |
       +-----+-----+
       |           |
       ↓           ↓
 Trigger       Capture Echo
 Sensor        using TIM1
       |           |
       +-----+-----+
             |
             ↓
     Calculate Distance
             |
             ↓
       Check Threshold
        /           \
       /             \
  Warning           Normal
     |                 |
     ↓                 ↓
 LED ON            LED OFF
 Buzzer ON         Buzzer OFF
     |
     ↓
  Repeat
```

---

# 12. Important Firmware Variables

The following variables are used by the firmware:

```c
volatile uint32_t echo_start = 0;
volatile uint32_t echo_end = 0;
volatile uint32_t echo_time = 0;
```

### `echo_start`

Stores the timer value captured at the rising edge of the Echo signal.

### `echo_end`

Stores the timer value captured at the falling edge of the Echo signal.

### `echo_time`

Stores the calculated duration of the Echo pulse.

---

The firmware also uses:

```c
volatile uint8_t echo_state = 0;
volatile uint8_t measurement_ready = 0;
```

### `echo_state`

Keeps track of whether the firmware is waiting for the rising or falling Echo edge.

### `measurement_ready`

Indicates that a complete Echo measurement has been captured and is ready for distance calculation.

---

The distance variable is:

```c
float distance_cm = 0.0f;
```

It stores the calculated distance in centimeters.

---

Buzzer control uses:

```c
volatile uint8_t buzzer_active = 0;
volatile uint32_t buzzer_start_tick = 0;
```

These variables keep track of whether the buzzer is active and when it was activated.

---

# 13. Important Configuration Constants

The firmware defines:

```c
#define BUZZER_ON_MS 5000
#define HIGH_LEVEL_THRESHOLD  5.0f
#define LOW_LEVEL_THRESHOLD   10000.0f
```

| Constant | Value | Purpose |
|---|---:|---|
| `BUZZER_ON_MS` | 5000 ms | Buzzer timing |
| `HIGH_LEVEL_THRESHOLD` | 5.0 cm | Upper/warning distance threshold |
| `LOW_LEVEL_THRESHOLD` | 10000.0 cm | Lower/warning distance threshold |

The threshold values should be calibrated for the physical tank and intended water-level limits before using the system as a practical water-level controller.

---

# 14. TIM1 Configuration

TIM1 is configured as the timer used for Echo pulse measurement.

Important settings in the firmware include:

```c
htim1.Instance = TIM1;
htim1.Init.Prescaler = 3;
htim1.Init.CounterMode = TIM_COUNTERMODE_UP;
htim1.Init.Period = 65535;
htim1.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
```

TIM1 Channel 1 is configured for Input Capture:

```c
sConfigIC.ICPolarity = TIM_INPUTCHANNELPOLARITY_RISING;
sConfigIC.ICSelection = TIM_ICSELECTION_DIRECTTI;
sConfigIC.ICPrescaler = TIM_ICPSC_DIV1;
sConfigIC.ICFilter = 0;
```

The capture polarity is changed dynamically:

```text
Initial:
Rising Edge

After rising edge:
Falling Edge

After falling edge:
Rising Edge
```

This allows the firmware to measure the complete HIGH duration of the Echo signal.

---

# 15. GPIO Configuration

The firmware configures the following functions:

| GPIO | Function | Direction |
|---|---|---|
| PE11 | HC-SR04 TRIG | Output |
| PE9 | HC-SR04 ECHO | Timer Input Capture |
| PA5 | Buzzer | Output |
| Status LED pin | Warning indication | Output |

The TRIG, buzzer and status LED outputs are initialized LOW.

---

# 16. Main Program Flow

The main program starts with:

```c
HAL_Init();
SystemClock_Config();
MX_GPIO_Init();
MX_TIM1_Init();
```

The timer and Input Capture interrupt are then started:

```c
HAL_TIM_Base_Start(&htim1);
HAL_TIM_IC_Start_IT(&htim1, TIM_CHANNEL_1);
```

The main loop then repeatedly:

```text
1. Reset measurement_ready
2. Trigger HC-SR04
3. Wait for Echo measurement
4. Stop waiting after the timeout condition
5. Calculate distance
6. Check warning threshold
7. Control LED
8. Control buzzer
9. Turn buzzer off after its configured duration
10. Wait 100 ms
11. Repeat
```

The firmware includes a 100 ms timeout while waiting for the Echo measurement, preventing the main loop from waiting indefinitely if a measurement is not completed.

---

# 17. HC-SR04 Trigger Function

The trigger function is:

```c
static void HCSR04_Trigger(void)
```

Its operation is:

```text
TRIG LOW
   ↓
Record timer value
   ↓
TRIG HIGH
   ↓
Wait for trigger interval
   ↓
TRIG LOW
```

The source code uses the TIM1 counter for this short timing interval.

---

# 18. Input Capture Interrupt

The main measurement callback is:

```c
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
```

The callback first verifies:

```c
if (htim->Instance == TIM1 &&
    htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1)
```

Then it performs two stages.

### Rising edge

```text
Capture start time
      ↓
Set echo_state = 1
      ↓
Change capture to falling edge
```

### Falling edge

```text
Capture end time
      ↓
Calculate Echo duration
      ↓
measurement_ready = 1
      ↓
Reset echo_state
      ↓
Change capture back to rising edge
```

This interrupt-driven design allows the timer hardware to capture the Echo transitions without relying on continuous software polling of the Echo signal.

---

# 19. Timer Overflow Handling

Because TIM1 uses a 16-bit counter with a maximum configured period of:

```text
65535
```

the timer may wrap around between the rising and falling Echo edges.

The firmware checks:

```c
if (echo_end >= echo_start)
```

If true:

```c
echo_time = echo_end - echo_start;
```

Otherwise, it calculates the elapsed time across the counter rollover:

```c
echo_time =
    (65535U - echo_start) + echo_end + 1U;
```

This makes the Echo measurement robust against a timer counter wrap.

---

# 20. Buzzer Logic

The buzzer is controlled using two states:

```text
buzzer_active = 0
buzzer_active = 1
```

When a warning condition is detected:

```text
If buzzer is not already active
        ↓
Set buzzer_active = 1
        ↓
Store current system tick
        ↓
Turn buzzer ON
```

The firmware later checks:

```c
if(buzzer_active &&
   ((HAL_GetTick()-buzzer_start_tick)>=BUZZER_ON_MS))
```

When the configured time has elapsed:

```text
Turn buzzer OFF
        ↓
buzzer_active = 0
```

---

# 21. Warning and Normal Conditions

## Warning condition

The current firmware considers the system to be in a warning state when:

```text
distance < 5 cm
```

OR

```text
distance > 10000 cm
```

During this state:

```text
Status LED → ON
Buzzer     → ON
```

The buzzer is then automatically switched off after the configured 5-second period unless the normal-condition logic resets it earlier.

---

## Normal condition

When:

```text
5 cm ≤ distance ≤ 10000 cm
```

the firmware performs:

```text
Status LED → OFF
Buzzer     → OFF
```

**Important:** These are the exact thresholds currently present in the supplied firmware. They should be changed to values appropriate for the physical tank and the desired definition of "high" and "low" water levels.

---

# 22. Error Handling

The firmware includes an `Error_Handler()` function.

If a critical HAL initialization or configuration operation fails, the program enters:

```c
__disable_irq();

while (1)
{
}
```

This stops normal operation and keeps the microcontroller in an error state.

---

# 23. System Clock

The project configures the STM32 system clock using the MSI oscillator.

The supplied firmware sets:

```c
RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_MSI;
RCC_OscInitStruct.MSIState = RCC_MSI_ON;
RCC_OscInitStruct.MSIClockRange = RCC_MSIRANGE_4;
RCC_OscInitStruct.PLL.PLLState = RCC_PLL_NONE;
```

The system clock is configured to use MSI directly.

---

# 24. Technologies Used

### Hardware

- STM32U575ZIT6Q
- NUCLEO-U575ZI-Q
- HC-SR04
- Active Buzzer Module
- Resistor Voltage Divider
- On-board LED

### Software

- C
- STM32 HAL
- STM32CubeIDE
- GPIO
- TIM1
- Timer Input Capture
- Interrupts
- HAL SysTick timing

---

# 25. Project Files

A typical STM32CubeIDE project structure is:

```text
STM32-Ultrasonic-Water-Level/
│
├── Core/
│   ├── Inc/
│   │   └── main.h
│   │
│   └── Src/
│       └── main.c
│
├── Drivers/
│   └── STM32 HAL Drivers
│
├── README.md
│
├── LICENSE
│
└── .ioc
```

The main application logic is implemented in:

```text
Core/Src/main.c
```

---

# 26. How to Build the Project

## Step 1 — Install STM32CubeIDE

Install STM32CubeIDE on your computer.

## Step 2 — Open the Project

Import or open the STM32CubeIDE project.

## Step 3 — Check Hardware Connections

Verify:

```text
HC-SR04 VCC  → 5 V
HC-SR04 GND  → GND
HC-SR04 TRIG → PE11
HC-SR04 ECHO → Voltage Divider → PE9

Buzzer +     → PA5
Buzzer GND   → GND
```

## Step 4 — Build

Build the project in STM32CubeIDE.

## Step 5 — Connect the NUCLEO Board

Connect the NUCLEO board to the computer using the appropriate USB connection.

## Step 6 — Program the Microcontroller

Flash/download the compiled firmware to the STM32U575ZIT6Q.

## Step 7 — Test the Sensor

Place an object or water surface at different distances from the HC-SR04.

Observe:

- Distance measurement
- Status LED
- Buzzer response

---

# 27. Testing Procedure

A basic testing procedure is:

### Test 1 — Sensor Connection

Check that the HC-SR04 is powered correctly.

### Test 2 — Trigger Signal

Verify that PE11 generates the trigger pulse.

### Test 3 — Echo Signal

Verify that the Echo signal reaches PE9 through the voltage divider.

### Test 4 — Timer Capture

Check that TIM1 captures the rising and falling Echo edges.

### Test 5 — Distance Calculation

Verify that:

```text
Echo duration → Distance
```

is calculated correctly.

### Test 6 — Warning Condition

Place the target close enough to satisfy the configured warning threshold.

The status LED and buzzer should activate.

### Test 7 — Normal Condition

Move the target into the configured normal range.

The LED and buzzer should turn off.

### Test 8 — Buzzer Timeout

Verify that the buzzer is switched off after the configured:

```text
5000 ms
```

duration.

---

# 28. Advantages

- Non-contact distance measurement.
- Simple hardware design.
- Low-cost ultrasonic sensor.
- Uses hardware timer capture instead of measuring the Echo pulse only through software delays.
- Interrupt-based Echo measurement.
- Visual and audible alerts.
- Timer overflow is handled in firmware.
- Can be expanded into a complete IoT water-management system.

---

# 29. Limitations

The current implementation has several practical limitations:

1. The system does not currently display the measured distance on an LCD or OLED.
2. The system does not store historical measurements.
3. There is no wireless connectivity in the supplied firmware.
4. The supplied code uses fixed threshold constants.
5. The current source code does not implement automatic water-pump control.
6. The system does not send notifications to a mobile application.
7. The distance calculation depends on the timer configuration being correctly matched to the timing assumptions used by the formula.
8. The supplied firmware uses the HC-SR04's Echo signal through a resistor divider and therefore requires correct hardware voltage interfacing.
9. The threshold values should be calibrated for the actual physical tank rather than being treated as universal values.

---

# 30. Possible Improvements

The project can be developed further into a smart water-management system.

### 30.1 OLED/LCD Display

Add an OLED or LCD to display:

```text
Water Level
Distance
Warning Status
```

Example:

```text
--------------------
 WATER MONITOR
--------------------
Distance : 42.5 cm
Status   : NORMAL
--------------------
```

### 30.2 Multiple Warning Levels

Instead of a single warning condition, implement:

```text
NORMAL
LOW LEVEL
HIGH LEVEL
CRITICAL LEVEL
```

### 30.3 Automatic Pump Control

A relay or suitable motor-control circuit can be added so that the STM32 automatically controls a water pump.

### 30.4 IoT Monitoring

Add a Wi-Fi/Bluetooth communication module to send water-level information to a mobile or web application.

### 30.5 Data Logging

Store measurements so that water-level history can be analyzed.

### 30.6 Mobile Notifications

A connected system could send alerts when the water reaches a critical level.

### 30.7 Calibration

Add configurable parameters for:

- Tank height
- Minimum water level
- Maximum water level
- Sensor offset
- Warning threshold

---

# 31. Example Extended System

A future version could operate as:

```text
                HC-SR04
                   |
                   ↓
             STM32U575
                   |
        +----------+----------+
        |          |          |
        ↓          ↓          ↓
      OLED       Buzzer      Wi-Fi
        |          |          |
        ↓          ↓          ↓
    Display     Warning    Mobile App
                              |
                              ↓
                         Cloud Storage
```

A pump-control output could also be added:

```text
STM32
  |
  ↓
Relay / Motor Driver
  |
  ↓
Water Pump
```

---

# 32. Applications

The project can be adapted for:

- Household overhead water tanks
- Underground water tanks
- Industrial storage tanks
- Water treatment systems
- Smart buildings
- Automated water-management systems
- Industrial liquid-level monitoring
- IoT-based water monitoring
- Educational embedded-system demonstrations

---

# 33. Learning Outcomes

This project demonstrates practical knowledge of:

### Embedded C

The firmware is written in C and uses STM32 HAL functions for hardware control.

### GPIO

GPIO pins are used for:

- Sensor trigger
- Buzzer control
- LED control

### Timers

TIM1 is used to measure the duration of the ultrasonic Echo signal.

### Input Capture

Input Capture records the timer value when the Echo signal changes state.

### Interrupts

The Echo measurement is handled through the TIM1 Input Capture callback.

### Sensor Interfacing

The project demonstrates how a microcontroller can communicate with an ultrasonic sensor.

### Signal-Level Interfacing

The voltage divider demonstrates the importance of adapting a sensor's signal level before connecting it to the microcontroller.

### Real-Time Monitoring

The microcontroller repeatedly performs measurements and responds to detected conditions.

---

# 34. Complete Working Sequence

The complete operation can be summarized as:

```text
                 START
                   |
                   ↓
             HAL_Init()
                   |
                   ↓
        Configure System Clock
                   |
                   ↓
           Initialize GPIO
                   |
                   ↓
            Initialize TIM1
                   |
                   ↓
       Start TIM1 Input Capture
                   |
                   ↓
          Generate TRIG pulse
                   |
                   ↓
            HC-SR04 measures
             ultrasonic echo
                   |
                   ↓
         Echo rising edge occurs
                   |
                   ↓
        Capture starting timer
                   |
                   ↓
       Switch to falling-edge mode
                   |
                   ↓
         Echo falling edge occurs
                   |
                   ↓
         Capture ending timer
                   |
                   ↓
          Calculate echo time
                   |
                   ↓
          Calculate distance
                   |
                   ↓
       Compare with thresholds
              /          \
             /            \
         WARNING         NORMAL
            |               |
            ↓               ↓
        LED ON            LED OFF
            |               |
            ↓               ↓
       Buzzer ON          Buzzer OFF
            |
            ↓
     Buzzer timeout check
            |
            ↓
         Wait 100 ms
            |
            ↓
       Repeat forever
```

---

# 35. Key Source-Code Functions

| Function | Purpose |
|---|---|
| `main()` | Main application and measurement loop |
| `SystemClock_Config()` | Configures the STM32 system clock |
| `MX_GPIO_Init()` | Configures GPIO pins |
| `MX_TIM1_Init()` | Configures TIM1 and Input Capture |
| `HCSR04_Trigger()` | Generates the ultrasonic trigger pulse |
| `HAL_TIM_IC_CaptureCallback()` | Captures Echo rising/falling edges |
| `Error_Handler()` | Handles critical initialization/runtime errors |

---

# 36. Core Formula

The main mathematical relationship used by the system is:

```text
             Echo Time × Speed of Sound
Distance = --------------------------------
                       2
```

In the firmware:

```c
distance_cm = (echo_time * 0.0343f) / 2.0f;
```

This converts the measured ultrasonic travel time into an estimated distance.

---

# 37. Important Hardware Note

The most important electrical interface in this project is the HC-SR04 Echo connection.

The circuit should be:

```text
HC-SR04 ECHO
     |
    1 kΩ
     |
     +------ PE9
     |
    2 kΩ
     |
    GND
```

Do **not** omit the intended voltage-level interface when connecting a 5 V Echo signal to a lower-voltage STM32 input.

---

# 38. Project Summary

This project demonstrates a complete embedded sensing and alert system using an STM32U575 microcontroller.

The **HC-SR04** provides ultrasonic distance information. The **STM32U575 TIM1 Input Capture peripheral** measures the Echo pulse duration. The firmware converts that timing information into distance and checks the result against configured thresholds.

When a warning condition is detected, the system provides two forms of feedback:

```text
Visual → Status LED
Audio  → Buzzer
```

The project therefore combines:

```text
Sensor Interfacing
       +
GPIO
       +
Timer
       +
Input Capture
       +
Interrupts
       +
Embedded C
       +
Real-Time Decision Making
```

This makes the project a useful demonstration of fundamental **microcontroller and embedded-system concepts**.

---

# 39. Project Information

| Item | Details |
|---|---|
| Project Type | Embedded / Microcontroller Project |
| Application | Ultrasonic Water-Level Monitoring |
| Microcontroller Board | NUCLEO-U575ZI-Q |
| MCU | STM32U575ZIT6Q |
| Sensor | HC-SR04 |
| Sensor Interface | GPIO + Timer Input Capture |
| Timer | TIM1 |
| Trigger Pin | PE11 |
| Echo Pin | PE9 |
| Buzzer Pin | PA5 |
| Programming Language | C |
| Framework/Library | STM32 HAL |
| IDE | STM32CubeIDE |
| Warning Output | Status LED + Active Buzzer |
| Measurement Method | Ultrasonic Time-of-Flight |

---

# 40. Author / Project Details

**Project Name:** Ultrasonic Water-Level Monitoring and Alert System

**Platform:** STM32U575ZIT6Q NUCLEO

**Domain:** Embedded Systems / Microcontrollers / Sensor Interfacing

**Programming Language:** C

**Development Tool:** STM32CubeIDE

---

## 📄 License

This project is intended for educational and academic purposes.

The generated STM32 source code includes the standard STMicroelectronics software licensing information supplied with the project.

---

## ⭐ Project Highlights

```text
✓ STM32U575 Microcontroller
✓ HC-SR04 Ultrasonic Sensor
✓ Timer Input Capture
✓ Interrupt-Based Echo Measurement
✓ Distance Calculation
✓ Voltage-Level Interfacing
✓ LED Warning
✓ Buzzer Warning
✓ Timer Overflow Handling
✓ Continuous Monitoring
✓ Expandable to IoT
✓ Expandable to Automatic Pump Control
```

---

**Built as an embedded-systems project using STM32U575 and HC-SR04.**
