# L476-I2C1-EEPROM
# STM32L476RG I2C EEPROM Demo (24LC256)

## Overview

This project demonstrates how to use the STM32L476RG Nucleo board to communicate with a 24LC256 EEPROM through the I2C interface.

The program:

1. Writes the string `"15"` into EEPROM memory location `0x1000`.
2. Waits for EEPROM write completion.
3. Reads the stored value back from EEPROM.
4. Converts the received ASCII characters into an integer.
5. Blinks the onboard LED (`PA5`) according to the value stored in EEPROM.

In this example, the EEPROM stores `"15"`, so the LED blinks **15 times**.

---

## Hardware

### MCU

* STM32L476RG

### EEPROM

* 24LC256
* Interface: I2C

### Connections

| EEPROM | STM32 |
| ------ | ----- |
| SDA    | PB9   |
| SCL    | PB8   |
| VCC    | 3.3V  |
| GND    | GND   |
| WP     | GND   |

### I2C Address

24LC256 default address:

A2 = 0
A1 = 0
A0 = 0

Device Address:

```c
#define I2C_ADDRESS 0xA0
```

---

## Software Flow

```text
Start
  |
  v
Initialize HAL
  |
Initialize GPIO
  |
Initialize I2C
  |
Write "15" to EEPROM (0x1000)
  |
Wait 5 seconds
  |
Read data from EEPROM
  |
Convert ASCII to integer
  |
Blink LED count times
  |
Endless loop
```

---

## EEPROM Write Function

```c
void WRITE(uint16_t MemLoc, uint8_t *pData, uint16_t len)
```

### Description

Writes data into EEPROM.

### Parameters

| Parameter | Description           |
| --------- | --------------------- |
| MemLoc    | EEPROM memory address |
| pData     | Data buffer           |
| len       | Number of bytes       |

### Example

```c
char wmsg[] = {'1','5'};

WRITE(0x1000,
      (uint8_t*)wmsg,
      strlen(wmsg)+1);
```

---

## EEPROM Read Function

```c
void READ(uint16_t MemLoc,
          uint8_t *pData,
          uint16_t len)
```

### Description

Reads data from EEPROM.

### Example

```c
char rmsg[10];

READ(0x1000,
     (uint8_t*)rmsg,
     strlen(rmsg)+1);
```

---

## Data Conversion

The EEPROM stores ASCII characters.

Example:

```text
'1' '5'
```

To convert to integer:

```c
count = 10*(rmsg[0]-'0')
      + (rmsg[1]-'0');
```

Result:

```text
count = 15
```

---

## LED Blink Operation

```c
for(i = 0; i < count; i++)
{
    HAL_GPIO_TogglePin(GPIOA, LED);
    HAL_Delay(1000);
}
```

### Expected Output

```text
LED Toggle #1
LED Toggle #2
...
LED Toggle #15
```

---

## Important Notes

### EEPROM Write Delay

24LC256 requires approximately 5 ms to complete an internal write cycle.

Current implementation:

```c
while(HAL_I2C_Master_Transmit(
      &hi2c1,
      I2C_ADDRESS,
      0,
      0,
      HAL_MAX_DELAY) != HAL_OK);
```

This technique performs ACK polling until the EEPROM becomes ready.

---

### Potential Improvement

Current code:

```c
READ(0x1000,
     (uint8_t*)rmsg,
     strlen(rmsg)+1);
```

Since `rmsg` is uninitialized, `strlen(rmsg)` may return an incorrect value.

Recommended:

```c
READ(0x1000,
     (uint8_t*)rmsg,
     sizeof(rmsg));
```

or

```c
READ(0x1000,
     (uint8_t*)rmsg,
     3);
```

for reading `"15\0"`.

---

## Expected Result

After reset:

1. EEPROM stores `"15"` at address `0x1000`.
2. STM32 reads the stored value.
3. Value is converted to integer `15`.
4. Onboard LED (PA5) blinks 15 times.
5. Program enters an infinite loop.

---

