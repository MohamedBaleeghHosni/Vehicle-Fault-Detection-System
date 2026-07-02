# 🚗 Vehicle Fault Detection and Logging System (VFDLS)

> A professional-grade dual-ECU embedded system built on two ATmega32 microcontrollers communicating via UART — continuously monitoring vehicle subsystems, detecting faults in real time, logging Diagnostic Trouble Codes (DTCs) to external EEPROM, and reporting them to the driver through a keypad-driven LCD interface.

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Platform](https://img.shields.io/badge/Platform-ATmega32%20×2-blue)
![Language](https://img.shields.io/badge/Language-C%20%2F%20Embedded-orange)
![Architecture](https://img.shields.io/badge/Architecture-Dual%20ECU%20%7C%20Layered%20Model-purple)
![Protocol](https://img.shields.io/badge/Protocol-UART%20%7C%20I2C-red)
![Frequency](https://img.shields.io/badge/Frequency-8%20MHz-lightgrey)

---

## 📸 Demo

<img width="1210" height="1192" alt="image" src="https://github.com/user-attachments/assets/9813f8af-a22e-4896-8a9a-a864357880df" />
---

## 📋 Table of Contents

- [Overview](#-overview)
- [Why This Project Matters](#-why-this-project-matters)
- [System Architecture](#-system-architecture)
- [Dual ECU Layered Model](#-dual-ecu-layered-model)
- [Hardware Components](#-hardware-components)
- [Subsystems & Fault Detection](#-subsystems--fault-detection)
- [UART Communication Protocol](#-uart-communication-protocol)
- [System Workflow](#-system-workflow)
- [DTC Error Codes](#-dtc-error-codes)
- [Driver Specifications](#-driver-specifications)
- [Source Code](#-source-code)
- [Getting Started](#-getting-started)

---

## 🔭 Overview

The **Vehicle Fault Detection and Logging System (VFDLS)** is a final embedded systems diploma project that simulates a real automotive diagnostic system — similar in concept to the OBD-II systems found in modern cars.

The system uses **two ATmega32 microcontrollers** working together as independent ECUs (Electronic Control Units), communicating over **UART**:

| ECU | Role |
|-----|------|
| **HMI ECU** (ATmega32 #1) | Driver interface — Keypad input, LCD output, menu navigation |
| **Control ECU** (ATmega32 #2) | Core engine — Sensor reading, fault detection, actuator control, EEPROM logging |

Faults are stored permanently as **Diagnostic Trouble Codes (DTCs)** in an external **24C16 EEPROM** via I2C — surviving even after power loss — and can be retrieved and displayed on demand, exactly like a real vehicle diagnostic scanner.

Built as the **Final Project** of the Standard Embedded Diploma under Edges for Training.

---

## 💡 Why This Project Matters

Most embedded projects use a single microcontroller. This project deliberately mirrors how real automotive systems are designed:

- **Separation of concerns** — the HMI never talks to sensors directly; the Control ECU never drives the display
- **UART inter-ECU communication** — structured command/response protocol between two independent processors
- **Persistent fault logging** — DTCs stored in non-volatile EEPROM, readable after power reset
- **Configurable drivers** — UART, ADC, I2C, and Timer all initialized via configuration structs, not hardcoded values
- **Full layered architecture** — two separate layered stacks, one per ECU

This is the same fundamental design pattern used in automotive ECUs, industrial PLCs, and safety-critical embedded systems.

---

## 🏗 System Architecture

```
                    ┌─────────────────────────────────────────────────────┐
                    │                   VFDLS SYSTEM                      │
                    │                                                     │
  ┌─────────────────┴──────────┐  UART  ┌───────────────────────────────┐│
  │       HMI ECU              │◄──────►│        Control ECU            ││
  │     ATmega32 #1            │        │       ATmega32 #2             ││
  │       @ 8 MHz              │        │         @ 8 MHz               ││
  │                            │        │                               ││
  │  ┌──────────┐ ┌─────────┐  │        │  ┌───────────┐ ┌──────────┐  ││
  │  │ 4×4      │ │ 4×16    │  │        │  │ HC-SR04   │ │  LM35    │  ││
  │  │ Keypad   │ │  LCD    │  │        │  │Ultrasonic │ │ Temp     │  ││
  │  └──────────┘ └─────────┘  │        │  └─────┬─────┘ └────┬─────┘  ││
  │                            │        │        │ ICU         │ ADC    ││
  │  Drivers: GPIO, UART,      │        │  ┌─────┴─────────────┴──────┐ ││
  │           Timer, LCD,      │        │  │     ATmega32 #2 Core     │ ││
  │           Keypad           │        │  └──┬──────────┬────────────┘ ││
  │                            │        │     │ I2C      │ GPIO/PWM    ││
  └────────────────────────────┘        │  ┌──┴───┐  ┌───┴────────┐    ││
                                        │  │24C16 │  │ DC Motors  │    ││
                                        │  │EEPROM│  │ (Windows)  │    ││
                                        │  └──────┘  └────────────┘    ││
                                        └───────────────────────────────┘│
                    └─────────────────────────────────────────────────────┘
```

---

## 🧱 Dual ECU Layered Model

Each ECU has its own independent layered software stack:

### HMI ECU Stack

```
┌────────────────────────────────────────┐
│           APPLICATION LAYER            │  ← Menu logic, command sending,
│                                        │    LCD display management
├──────────────────┬─────────────────────┤
│     Keypad       │        LCD          │  ← HAL Layer
├────────┬─────────┴──┬──────────────────┤
│  GPIO  │   Timer    │      UART        │  ← MCAL Layer
└────────┴────────────┴──────────────────┘
         ATmega32 #1 Hardware
```

### Control ECU Stack

```
┌────────────────────────────────────────┐
│           APPLICATION LAYER            │  ← Fault detection, DTC logging,
│                                        │    sensor polling, actuator control
├────────┬──────────┬────────────────────┤
│ Motor  │   LM35   │      EEPROM        │  ← HAL Layer
│        │          │  Ultrasonic        │
├────────┴──┬───────┴──┬──────┬──────────┤
│   GPIO    │   PWM    │ ADC  │ UART │I2C│  ← MCAL Layer
└───────────┴──────────┴──────┴──────┴───┘
            ATmega32 #2 Hardware
```

---

## 🔧 Hardware Components

| Component | Connected To | Interface |
|-----------|-------------|-----------|
| 4×4 Keypad | HMI ECU | GPIO |
| 4×16 LCD Display | HMI ECU | GPIO (8-bit) |
| HC-SR04 Ultrasonic | Control ECU | ICU (PD6 echo, PD7 trig) |
| LM35 Temperature Sensor | Control ECU | ADC channel |
| 24C16 External EEPROM | Control ECU | I2C |
| DC Motors × 2 (Car Windows) | Control ECU | GPIO + PWM |
| ATmega32 #1 (HMI ECU) | — | UART TX/RX |
| ATmega32 #2 (Control ECU) | — | UART TX/RX |

---

## 🔍 Subsystems & Fault Detection

### 🅿️ Parking Assistance Unit (HC-SR04 Ultrasonic)

- Continuously measures distance between car and obstacles
- Uses ICU (Input Capture Unit) for precise echo timing
- Distance calculated from echo pulse width

| Condition | Action | DTC |
|-----------|--------|-----|
| Distance < 10 cm | Log fault to EEPROM | **P001 — ACCIDENT_MIGHT_HAPPENED** |
| Distance ≥ 10 cm | Normal operation | — |

### 🌡️ Engine Temperature Monitoring Unit (LM35)

- Reads engine temperature via ADC
- LM35 output: 10mV per °C, converted via ADC

| Condition | Action | DTC |
|-----------|--------|-----|
| Temperature > 90°C | Log fault to EEPROM | **P002 — ENGINE_HIGH_TEMPERATURE** |
| Temperature ≤ 90°C | Normal operation | — |

### 🪟 Window Control Unit (DC Motors)

- Two DC motors simulate car windows (Open / Close)
- Controlled by 2 push buttons per window
- State (Open/Closed) reported to HMI ECU for LCD display
- Motors always run at maximum speed via PWM Timer0

### 💾 External EEPROM (24C16 via I2C)

- Stores all DTCs permanently — survives power reset
- Each fault logged once per occurrence
- DTCs retrievable on demand via HMI keypad command

---

## 📡 UART Communication Protocol

All communication between the two ECUs follows a structured command/response protocol:

### HMI ECU → Control ECU (Commands)

| Command | Action |
|---------|--------|
| `Command 1` | Start Monitoring — activate all sensors and actuators |
| `Command 2` | Send Live Sensor Values — return Temp, Distance, Window states |
| `Command 3` | Send Logged Faults — return all DTCs from EEPROM |
| `Command 4` | Stop Monitoring — halt all sensor polling |

### Control ECU → HMI ECU (Responses)

| Response | Content |
|----------|---------|
| Sensor data | Temperature (°C), Distance (cm), Window 1 state, Window 2 state |
| DTC list | All stored P001/P002 codes with descriptions |

### UART Configuration (both ECUs)

```c
UART_ConfigType uart_config = {
    .bit_data = UART_8_BIT_DATA,
    .parity   = UART_NO_PARITY,
    .stop_bit = UART_1_STOP_BIT,
    .baud_rate = 9600
};
UART_init(&uart_config);
```

---

## 🖥️ System Workflow

### Main Menu (LCD)
```
┌──────────────────────┐
│ 1. Start Operation   │
│ 2. Display Values    │
│ 3. Retrieve Faults   │
│ 4. Stop Monitoring   │
└──────────────────────┘
```

### Key 1 — Start Operation
```
User presses 1
      │
      ▼
HMI sends "Start" → Control ECU
      │
      ▼
Control ECU activates: Ultrasonic + LM35 + Motors
Fault detection & DTC logging begin
      │
      ▼
LCD shows:
┌──────────────────────┐
│  Operation Started!  │
│  Monitoring Active...│
└──────────────────────┘
      │
  10 seconds → back to Main Menu
```

### Key 2 — Display Live Values
```
LCD shows for 10 seconds:
┌──────────────────────┐
│  Temp: 82 C          │
│  Distance: 40 cm     │
│  Win1: Closed        │
│  Win2: Open          │
└──────────────────────┘
Then prompts:
┌──────────────────────┐
│  Display again?      │
│  Press 2=YES         │
│  Other key=MAIN MENU │
└──────────────────────┘
```

### Key 3 — Retrieve Logged Faults
```
LCD scrolls through all stored DTCs:
┌──────────────────────┐
│  Logged Faults:      │
│  P001:DistanceTooClose│
│  P002: Overheat      │
│  --End of List--     │
└──────────────────────┘
```

### Key 4 — Stop Monitoring
```
LCD shows:
┌──────────────────────┐
│  System Monitoring   │
│  Stopped!            │
│  Returning to Menu…  │
└──────────────────────┘
10 seconds → back to Main Menu
```

---

## ⚠️ DTC Error Codes

| DTC Code | Description | Trigger Condition | Logged |
|----------|-------------|-------------------|--------|
| **P001** | ACCIDENT_MIGHT_HAPPENED | Distance < 10 cm | Once per occurrence |
| **P002** | ENGINE_HIGH_TEMPERATURE | Temperature > 90°C | Once per occurrence |

DTCs are stored permanently in the 24C16 EEPROM and persist after power reset.

---

## 🔌 Driver Specifications

### HMI ECU — MCAL Layer

#### UART Driver (configurable)
```c
typedef struct {
    UART_BitDataType  bit_data;
    UART_ParityType   parity;
    UART_StopBitType  stop_bit;
    UART_BaudRateType baud_rate;
} UART_ConfigType;

void UART_init(const UART_ConfigType *Config_Ptr);
```

#### Timer Driver (Timer0 / Timer1 / Timer2 — dynamic config)
```c
typedef struct {
    uint16         timer_InitialValue;
    uint16         timer_compare_MatchValue; // compare mode only
    Timer_ID_Type  timer_ID;
    Timer_ClockType timer_clock;
    Timer_ModeType  timer_mode;
} Timer_ConfigType;

void Timer_init(const Timer_ConfigType *Config_Ptr);
void Timer_deInit(Timer_ID_Type timer_type);
void Timer_setCallBack(void(*a_ptr)(void), Timer_ID_Type a_timer_ID);
```
> Project uses **Timer1 in Compare mode** for HMI ECU timing.

#### GPIO Driver
```c
// Same GPIO driver used across all projects
// Handles pin direction and read/write for all ports
```

---

### HMI ECU — HAL Layer

#### LCD Driver
```c
// 4×16 LCD in 8-bit mode
// Connected to HMI ECU
```

#### Keypad Driver
```c
// 4×4 Keypad
// Connected to HMI ECU
```

---

### Control ECU — MCAL Layer

#### ADC Driver (configurable)
```c
typedef struct {
    ADC_ReferenceVoltage ref_volt;
    ADC_Prescaler        prescaler;
} ADC_ConfigType;

void ADC_init(const ADC_ConfigType *Config_Ptr);
uint16 ADC_readChannel(uint8 channel_num);
```

#### I2C Driver (configurable)
```c
typedef struct {
    TWI_AddressType address;
    TWI_BaudRateType bit_rate;
} TWI_ConfigType;

void TWI_init(const TWI_ConfigType *Config_Ptr);
```

#### PWM Driver (Timer0)
```c
void PWM_Timer0_Start(uint8 duty_cycle);
// Non-inverting mode | Prescaler: F_CPU/64 | Output: OC0
```

#### ICU Driver
```c
// F_CPU/8 prescaler | First edge: Rising
// Used by Ultrasonic driver
```

#### UART Driver
```c
// Same configurable UART driver as HMI ECU
// Shared between both ECUs
```

---

### Control ECU — HAL Layer

#### Ultrasonic Driver (HC-SR04)
```c
void Ultrasonic_init(void);
void Ultrasonic_Trigger(void);
uint16 Ultrasonic_readDistance(void);
void Ultrasonic_edgeProcessing(void); // ICU callback
```

#### LM35 Temperature Sensor
```c
// ADC channel input | Returns temperature in °C
```

#### DC Motor Driver
```c
void DcMotor_Init(void);
void DcMotor_Rotate(DcMotor_State state, uint8 speed);
// Always runs at 100% speed via Timer0 PWM
```

#### External EEPROM Driver (24C16)
```c
// Controlled via I2C driver
// Stores and retrieves DTCs permanently
```

---

## 💻 Source Code

### HMI ECU

#### `HMI_main.c` — HMI Application Layer
```c
/*
 * HMI_ECU_(ATmega32_1).c
 *
 *  Created on: Oct 5, 2025
 *      Author: Mohamed Baleegh
 */
#include <avr/io.h>
#include "TIMER.h"
#include "LCD.h"
#include "std_types.h"
#include "keypad.h"
#include "gpio.h"
#include "uart.h"
#include "common_macros.h"
#include "util/delay.h"

/*******************************************************************************
 *                              Pin Definitions
 *******************************************************************************/
#define MOTOR1_UP_BUTTON_PORT    PORTA_ID
#define MOTOR1_UP_BUTTON_PIN     PIN0_ID
#define MOTOR1_DOWN_BUTTON_PORT  PORTA_ID
#define MOTOR1_DOWN_BUTTON_PIN   PIN1_ID
#define MOTOR2_UP_BUTTON_PORT    PORTA_ID
#define MOTOR2_UP_BUTTON_PIN     PIN2_ID
#define MOTOR2_DOWN_BUTTON_PORT  PORTA_ID
#define MOTOR2_DOWN_BUTTON_PIN   PIN3_ID






/*******************************************************************************
 *                          Definitions                                *
 *******************************************************************************/
#define START_OPERATION         1
#define SEND_LIFE_SENSOR_VALUES 2
#define SEND_FAULTS             3
#define STOP_CAR                4
#define WINDOW1_UP              5
#define WINDOW1_DOWN            6
#define WINDOW2_UP              7
#define WINDOW2_DOWN            8
#define WINDOWS_STOP            9
#define ON                      1
#define OFF                     0
#define SENT			       	1
#define YES                     1
#define NO                      0
#define OPEN                    1
#define CLOSE                   0
#define MAX_NUMBER_OF_ERRORS    25
#define RECEIVED                 1


/*******************************************************************************
 *                         Global Variables                                *
 *******************************************************************************/
uint8 key;
uint8 menu_option;
uint8 tick = 0;
uint8 tick_flag = 0;
uint8 WINDOW1_STATE = CLOSE ;
uint8 WINDOW2_STATE = CLOSE;

/*******************************************************************************
 *                         Functions                                   *
 *******************************************************************************/
void CheckWindowsButtons(void)/* function to check if the buttons are pressed or no..*/
{

    /* --- Window 1 Controls --- */
    if (GPIO_readPin(MOTOR1_UP_BUTTON_PORT, MOTOR1_UP_BUTTON_PIN) == LOGIC_HIGH)
    {
        UART_sendByte(WINDOW1_UP);
        WINDOW1_STATE = OPEN;
    }
    else if (GPIO_readPin(MOTOR1_DOWN_BUTTON_PORT, MOTOR1_DOWN_BUTTON_PIN) == LOGIC_HIGH)
    {
        UART_sendByte(WINDOW1_DOWN);
        WINDOW1_STATE = CLOSE;
    }

    /* --- Window 2 Controls --- */
    else if (GPIO_readPin(MOTOR2_UP_BUTTON_PORT, MOTOR2_UP_BUTTON_PIN) == LOGIC_HIGH)
    {
        UART_sendByte(WINDOW2_UP);
        WINDOW2_STATE = OPEN;
    }
    else if (GPIO_readPin(MOTOR2_DOWN_BUTTON_PORT, MOTOR2_DOWN_BUTTON_PIN) == LOGIC_HIGH)
    {
        UART_sendByte(WINDOW2_DOWN);
        WINDOW2_STATE = CLOSE;
    }
}

void Timer1_10Seconds(void)/* 10 seconds function */
{
	tick++;

	if (tick >= 10)
	{
		tick_flag = 1;
		tick = 0;
	}
}
void Display_MainMenu(void)/* This function is to be used in the code while car is ON */
{


	LCD_clear_Display();
	LCD_displayStringRowColumn(0, 0, "1-Start Car");
	LCD_displayStringRowColumn(1, 0, "2-Display Values");
	LCD_displayStringRowColumn(2, 0, "3-Read Faults");
	LCD_displayStringRowColumn(3, 0, "4-Stop Car");



}
/* Displaying the Errors and faults function*/
void DisplayFaultsOnLCD(char errorData[MAX_NUMBER_OF_ERRORS][5], uint8 ErrorNumber)
{
    uint8 i;

    for (i = 0; i < ErrorNumber; i +=3 )/* 3 FAULTS SCROLL TECHNIQUE */
    {
		CheckWindowsButtons();

        LCD_clear_Display();
        LCD_displayStringRowColumn(0, 0, "Logged Faults:");
        LCD_moveCursor(0,14);
        LCD_intgerToString(ErrorNumber);

        if (i < ErrorNumber)
        {
            if (errorData[i][3] == '1')
                LCD_displayStringRowColumn(1, 0, "P001: Over Heat ");
            else if (errorData[i][3] == '2')
                LCD_displayStringRowColumn(1, 0, "P002: Too Close ");
        }

        if ((i + 1) < ErrorNumber)
        {
            if (errorData[i + 1][3] == '1')
                LCD_displayStringRowColumn(2, 0, "P001: Over Heat ");
            else if (errorData[i + 1][3] == '2')
                LCD_displayStringRowColumn(2, 0, "P002: Too Close ");
        }

        if ((i + 2) < ErrorNumber)
        {
            if (errorData[i + 2][3] == '1')
                LCD_displayStringRowColumn(3, 0, "P001: Over Heat ");
            else if (errorData[i + 2][3] == '2')
                LCD_displayStringRowColumn(3, 0, "P002: Too Close ");
        }
        else
        {
            LCD_displayStringRowColumn(3, 0, "==End of List==");
        }

        _delay_ms(1500); // Scroll delay before showing next group
    }

    LCD_clear_Display();
    LCD_displayStringRowColumn(3, 0, "==End of List==");
    _delay_ms(1000);
}



/*******************************************************************************
 *                         main function                                   *
 *******************************************************************************/
int main(void)
{

	/*******************************************************************************
	 *                         Types Declaration                                 *
	 *******************************************************************************/

	/* Timer ConfigurationS for 1 second per tick*/
	Timer_ConfigType Timer1_Configuration = {
			0,
			7812, /* 1 second per tick*/
			TIMER1_ID,
			TIMER_FCPU_1024,
			TIMER_MODE_COMPARE
	};

	/*  UART Configurations  */
	UART_ConfigType UART_Configuration = {
		UART_8_BIT,           /* 8-bit data */
		UART_PARITY_DISABLED, /* No parity */
		UART_ONE_STOP_BIT,    /* 1 stop bit */
		UART_BAUD_9600        /* Baud rate 9600 */
	};
	 uint8 CAR = OFF;

	  uint8 UserChoice;
	 /* For the key - 2*/
	uint8 temperature = 0;
	uint8 distance = 0;
	uint8 display_again = YES;
	uint8 display_again2 = YES;

	 /* For the KEY-3 */
	uint8 errorData[MAX_NUMBER_OF_ERRORS][5];
	uint8 i = 0;
	uint8 j = 0;

	 UART_init(&UART_Configuration);
     Timer_init(&Timer1_Configuration);/* initialize the timer */
	Timer_setCallBack(Timer1_10Seconds, TIMER1_ID);



	 SREG |= (1 << 7);/* Enable the  global interrupts*/

	/*LCD 16*4 */
	LCD_init();
	LCD_clear_Display();
	LCD_displayStringRowColumn(0, 0, "1-Start Car");
	LCD_displayStringRowColumn(1, 0, "2-Display Values");
	LCD_displayStringRowColumn(2, 0, "3-Read Faults");
	LCD_displayStringRowColumn(3, 0, "4-Stop Car");

	/* -----------Configure windows motor control buttons ----------- */
	GPIO_setupPinDirection(MOTOR1_UP_BUTTON_PORT, MOTOR1_UP_BUTTON_PIN,PIN_INPUT);
	GPIO_setupPinDirection(MOTOR1_DOWN_BUTTON_PORT, MOTOR1_DOWN_BUTTON_PIN,PIN_INPUT);
	GPIO_setupPinDirection(MOTOR2_UP_BUTTON_PORT, MOTOR2_UP_BUTTON_PIN,PIN_INPUT);
	GPIO_setupPinDirection(MOTOR2_DOWN_BUTTON_PORT, MOTOR2_DOWN_BUTTON_PIN,PIN_INPUT);


	while(1)
	{

		 key=KEYPAD_getPressedKey();
		 /* Starting the car using the keypad , by pressing 1 on the keypad */

		 if(key==START_OPERATION)
		 {

			CAR = ON;
			display_again=YES;
			display_again2=YES;
			UART_sendByte(START_OPERATION);
			UART_recieveByte();
			LCD_clear_Display();
			tick_flag = 0;
			tick = 0;
			Timer_init(&Timer1_Configuration);/* initialize the timer */
			Timer_setCallBack(Timer1_10Seconds, TIMER1_ID);
			while(tick_flag==0&& CAR==ON)
			{
				LCD_displayStringRowColumn(0,0,"Engine Started!");
				LCD_displayStringRowColumn(1, 0, "Reading Values");
				LCD_displayStringRowColumn(2,0,"                ");
				LCD_displayStringRowColumn(3,0,"                ");

			}
			tick_flag=0;
			tick=0;
			Display_MainMenu();

           Timer_deInit(TIMER1_ID);


		 }


		 /* Displaying the Values of the temperature,distance,windows states . by pressing 2 on the keypad */
		 else if(key==SEND_LIFE_SENSOR_VALUES && CAR==ON)
		 {
			tick_flag = 0;
			tick = 0;
			Timer_init(&Timer1_Configuration);/* initialize the timer */
			LCD_clear_Display();


			while(display_again==YES && tick_flag==0)
			{
				CheckWindowsButtons();

				LCD_displayStringRowColumn(0, 0, "Temperature");
				LCD_displayStringRowColumn(0, 15, "C");
				LCD_displayStringRowColumn(1, 0, "Distance");
				LCD_displayStringRowColumn(1,14,"CM");
				LCD_displayStringRowColumn(2, 0, "Win1:");
				LCD_displayStringRowColumn(3, 0, "WIN2:");
				UART_sendByte(SEND_LIFE_SENSOR_VALUES);
				temperature = UART_recieveByte();
				distance = UART_recieveByte();
				LCD_moveCursor(0, 12);
				if (temperature < 100)
				{
					LCD_intgerToString(temperature);
					LCD_dispCharacter(' ');
				}
				else
				{
					LCD_intgerToString(temperature);

				}
				LCD_moveCursor(1, 11);
				if (distance < 100)
				{
					LCD_intgerToString(distance);
					LCD_dispCharacter(' ');

				}
				else
				{
					LCD_intgerToString(distance);

				}

								if(WINDOW1_STATE==OPEN)
								{
									LCD_displayStringRowColumn(2,6,"OPEN  ");

								}
								else if(WINDOW1_STATE==CLOSE)
								{
									LCD_displayStringRowColumn(2,6,"CLOSED");


								}
								if (WINDOW2_STATE == OPEN)
								{
									LCD_displayStringRowColumn(3, 6, "OPEN  ");

								}
								else if (WINDOW2_STATE == CLOSE)
								{
									LCD_displayStringRowColumn(3, 6, "CLOSED");

								}


			}
					LCD_clear_Display();
					LCD_displayStringRowColumn(0, 0, "Display again?");
					LCD_displayStringRowColumn(1, 0, "Press");
					LCD_displayStringRowColumn(3, 0, "2 : again");
					LCD_displayStringRowColumn(2, 0, "Other: main menu");
					UserChoice = KEYPAD_getPressedKey();
					if (UserChoice ==2 )
					{
				display_again = YES;

			}
					else
					{

				display_again = NO;
				Display_MainMenu();
				tick_flag = 0;
				tick = 0;

					}




			}




		 /*--------------  Error Retrieval (Key 3 - HMI ECU)  -------------- */
		else if (key == SEND_FAULTS && CAR == ON)
		{

			uint8 ErrorNumber = 0;
			tick_flag = 0;
			tick = 0;
			Timer_init(&Timer1_Configuration);
			LCD_clear_Display();

			while (display_again2 == YES && tick_flag == 0)
			{

				UART_sendByte(SEND_FAULTS);
				ErrorNumber = UART_recieveByte();
				for (i = 0; i < ErrorNumber; i++)
				{
					for (j = 0; j < 4; j++)
					{
						/*Receive the bytes from the control ECU and store them in a
						 2D ARRAY by saving the first 4 bytes as the first array and
						 the next 4 bytes in the next array and so On, until we are
						 done with all bytes sent.
						 */
						errorData[i][j] = UART_recieveByte();

						UART_sendByte(RECEIVED);		    /*  Handshake with HMI ECU */

						CheckWindowsButtons();

					}

					}
				/* Case no errors or faults (Error Number =0 )*/
					if (ErrorNumber == 0)
					    {
					        LCD_displayStringRowColumn(0, 0, "No Faults Logged");
					    }
					else
					{

					DisplayFaultsOnLCD((char (*)[5])errorData, ErrorNumber);

					}
			}
			LCD_clear_Display();
			LCD_displayStringRowColumn(0, 0, "Display again?");
			LCD_displayStringRowColumn(1, 0, "Press");
			LCD_displayStringRowColumn(3, 0, "3 : again");
			LCD_displayStringRowColumn(2, 0, "Other: main menu");
			UserChoice = KEYPAD_getPressedKey();
			if (UserChoice == 3)
			{
				display_again2 = YES;
				tick_flag = 0;

			}
			else
			{

				display_again2 = NO;
				Display_MainMenu();

			}

		}
		else if (key == STOP_CAR && CAR == ON)
		{
			CAR = OFF;
			UART_sendByte(STOP_CAR);
			UART_recieveByte();
			LCD_clear_Display();
			tick_flag = 0;
			tick = 0;
			Timer_init(&Timer1_Configuration);

			while (tick_flag == 0)
			{
				LCD_displayStringRowColumn(0, 0, "ENGINE OFF");
				LCD_displayStringRowColumn(1, 0, "BACK TO MENU ...");
				LCD_displayStringRowColumn(2, 0, "                ");
				LCD_displayStringRowColumn(3, 0, "                ");

			}
			tick_flag = 0;
			tick = 0;
			Display_MainMenu();
			Timer_deInit(TIMER1_ID);


		}
}
}

```


---

### Control ECU

#### `Control_main.c` — Control Application Layer
```c
/*
 * Control_ECU_(ATmega32_2).c
 *
 *  Created on: Oct 5, 2025
 *      Author: Mohamed Baleegh
 */
#include "uart.h"
#include "ADC.h"
#include "DC-MOTOR.h"
#include "EEPROM.h"
#include "lm35_sensor.h"
#include "I2C.h"
#include "gpio.h"
#include "std_types.h"
#include "UltraSonic_Sensor.h"
#include "common_macros.h"
#include "ICU.h"
#include "PWM.h"
#include <avr/io.h>
#include <util/delay.h>
#include <avr/eeprom.h>

/*******************************************************************************
 *                          Definitions                                *
 *******************************************************************************/
#define START_OPERATION         1
#define SEND_LIFE_SENSOR_VALUES 2
#define SEND_FAULTS             3
#define STOP_CAR                4
#define WINDOW1_UP              5
#define WINDOW1_DOWN            6
#define WINDOW2_UP              7
#define WINDOW2_DOWN            8
#define WINDOWS_STOP            9
#define ON                      1
#define OFF                     0
#define SENT			       	1
#define YES                     1
#define NO                      0
#define UP                      1
#define DOWN                    0
#define RECEIVED                1
/*******************************************************************************
 *                      Global Configuration Structures
 *******************************************************************************/

/* UART Configurations */
UART_ConfigType UART_Configuration = { UART_8_BIT, /* 8-bit data */
UART_PARITY_DISABLED, /* No parity */
UART_ONE_STOP_BIT, /* 1 stop bit */
UART_BAUD_9600 /* Baud rate 9600 */
};

TWI_ConfigType I2C_configuration = { 0x01, TWI_PRESCALER_1 };
ADC_ConfigType ADC_configuration = { DIVISION_FACTOR_128, Internal_VRef /*  internal 2.56 V reference */
};
/*******************************************************************************
 *                                Main Function
 *******************************************************************************/

int main(void) {
	/*******************************************************************************
	 *                                Initializations
	 *******************************************************************************/
	UART_init(&UART_Configuration);
	DcMotor_Init();
	I2C_init(&I2C_configuration);
	ADC_init(&ADC_configuration);
	Ultrasonic_init();

	SREG |= (1 << 7);/* Enable the  global interrupts*/
	uint8 command;/* The variable i will be using to get the byte of the UART
	 and do the commands upon which we have commands are:(1,2,3,4,A,B,C,D)
	 and each is explained individually in the code
	 */
	uint8 errorNumber = 0;/* Number of errors collected at the EEPROM */

	uint8 temperature = 0;
	uint8 distance = 0;
//	uint8 OVERHEAT_logged = NO;
//	uint8 ACCIDENT_logged = NO;
	 uint8 data [5];
	uint16 baseAddress = 0x0000;
//	uint8 i=0;
//	uint8 j=0;
//	uint8 k=0;

	uint8 prev_temp_read=0;
	uint8 prev_dist_read=0;

	/*	volatile uint8 WINDOW1_STATE=UP;
	 volatile uint8 WINDOW2_STATE=UP;
	 */

	while (1)
	{




		/* ----------  ----------  ---------- */
		/* ----------  ----------  ---------- */
		/* ----------  Main Menu   ---------- */
		/* ----------  ERROR WRITING IN THE EEPROM USING THE I2C   ---------- */

		distance = Ultrasonic_readDistance();
		temperature = LM35_getTemperature();
				if (temperature > 90 && temperature != prev_temp_read) {

					prev_temp_read = temperature;

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 0, 'P');
					_delay_ms(10);

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 1, '0');
					_delay_ms(10);

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 2, '0');
					_delay_ms(10);

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 3, '1');
					errorNumber++;

				}

				if (distance < 10 && (distance != prev_dist_read)) {
					prev_dist_read = distance;

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 0, 'P');
					_delay_ms(10);

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 1, '0');
					_delay_ms(10);

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 2, '0');
					_delay_ms(10);

					EEPROM_writeByte(0x0000 + (errorNumber * 4) + 3, '2');
					errorNumber++;

				}

		command = UART_recieveByte();


		if (command == START_OPERATION) {
			UART_sendByte(RECEIVED);
			EEPROM_clearAll(0XFF);/*Function that clears the whole EEPROM before starting the car to make sure of system stability*/
		}

		else if (command == SEND_LIFE_SENSOR_VALUES) {
			temperature = LM35_getTemperature();
			distance = Ultrasonic_readDistance();

			UART_sendByte(temperature);
			_delay_ms(3);

			UART_sendByte(distance);



		}

		/********************* Faults Retrieval  *********************/

		else if (command == SEND_FAULTS)
		{
		    /* Step 1: Read sensors */
		    distance = Ultrasonic_readDistance();
		    temperature = LM35_getTemperature();

		    /* Step 2: Check for new Over heat fault (P001) */
		    if (temperature > 90 && temperature != prev_temp_read)
		    {
		        prev_temp_read = temperature;

		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 0, 'P');
		        _delay_ms(10);
		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 1, '0');
		        _delay_ms(10);
		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 2, '0');
		        _delay_ms(10);
		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 3, '1');
		        _delay_ms(10);

		        errorNumber++;
		    }

		    /* Step 3: Check for Too Close fault (P002) */
		    if (distance < 10 && distance != prev_dist_read)
		    {
		        prev_dist_read = distance;

		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 0, 'P');
		        _delay_ms(10);
		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 1, '0');
		        _delay_ms(10);
		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 2, '0');
		        _delay_ms(10);
		        EEPROM_writeByte(0x0000 + (errorNumber * 4) + 3, '2');
		        _delay_ms(10);

		        errorNumber++;
		    }

		    /* Step 4: Handshake with HMI ECU */
		    UART_sendByte(errorNumber);   // Send number of logged faults

		    /* Step 5: Wait for message "Retrieve Faults#" from HMI */

		    /* Step 6: Send all logged faults one by one */
		    for (uint8 i = 0; i < errorNumber; i++)
		        {
		            baseAddress = 0x0000 + (i * 4);

		            /* Read 4 bytes of error (e.g., P001) */
		            for (uint8 j = 0; j < 4; j++)
		            {
					EEPROM_readByte(baseAddress + j, &data[j]);/* Read From EEPROM AND store at an array */
					_delay_ms(5);

					UART_sendByte(data[j]);/* Send the ARRAY contents byte by byte */
					UART_recieveByte();
		            }

		        }
		}


		else if (command ==  STOP_CAR)
		{
			UART_sendByte(RECEIVED);
			DcMotor_Rotate(1, STOP, 0);
			DcMotor_Rotate(2, STOP, 0);

		}


		/********************* Window 1 UP *********************/
		else if (command == WINDOW1_UP)
		{

			DcMotor_Rotate(1, CW, 100);

		}

		/********************* Window 1 DOWN *********************/
		else if (command == WINDOW1_DOWN) {
			DcMotor_Rotate(1, ACW, 100);


		}

		/********************* Window 2 UP *********************/
		else if (command == WINDOW2_UP) {
			DcMotor_Rotate(2, CW, 100);


		}

		/********************* Window 2 DOWN *********************/
		else if (command == WINDOW2_DOWN) {
			DcMotor_Rotate(2, ACW, 100);


		}




	}
}
```


---

## 🚀 Getting Started

### Prerequisites

- **Atmel/Microchip Studio** or **Eclipse IDE with AVR plugin**
- **AVR-GCC** toolchain
- **AVRDUDE** + USBasp programmer ×2 (one per ECU)
- **Proteus** for simulation

### Project Structure

```
VFDLS/
├── HMI_ECU/
│   ├── APP/
│   │   └── HMI_main.c
│   ├── HAL/
│   │   ├── LCD/
│   │   └── Keypad/
│   └── MCAL/
│       ├── GPIO/
│       ├── UART/
│       └── Timer/
│
└── Control_ECU/
    ├── APP/
    │   └── Control_main.c
    ├── HAL/
    │   ├── LM35/
    │   ├── Ultrasonic/
    │   ├── DC_Motor/
    │   └── EEPROM/
    └── MCAL/
        ├── GPIO/
        ├── ADC/
        ├── UART/
        ├── PWM/
        ├── ICU/
        └── I2C/
```


---


> This was the **final capstone project** of the Standard Embedded Systems Diploma under **Edges for Training**, supervised by Eng. Mohamed Tarek. It represents the most complex project in the diploma — combining dual-ECU architecture, five communication protocols (UART, I2C, ADC, ICU, PWM), persistent non-volatile fault logging, and a complete automotive-grade software design pattern.

---

<p align="center">
  <strong>Final Capstone Project — Standard Embedded Systems Diploma | Edges for Training | 2025</strong>
</p>
