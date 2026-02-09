# R700 Vector Robotic Arm (vektora robota brako)

## Listo d'akcioj

- bazcirkvito
  - mikroprocesoroj : `ATMega64`
  - memoro : 64kB Flash, 4KB SRAM, 2KB EEPROM
  - programlingvo : **`C`**
  - vastiĝa sistemo : $\mathrm{I^2C}$ Bus
- I/O modulo
- brako
  - 6 aktuatoroj
  - DOF : 6
    - **limigitaj al** laborregiono
  - materialo : dika aluminio
- robota kontrolo
- robota stando

## Programado

### malaltnivela programlingvo : **`C`**

- dizajnita en la 1970-aj jaroj
  - Bell Lab, Usono
  - D. Ritchie, K. Thompson
- posteulo de **`B`**

### ekzemploj

```C
// Moves the base servo to three angles with delays
#include <avr/io.h>
#include <util/delay.h>

void pwm_init() {
    DDRB |= (1 << PB5);              // OC1A output
    TCCR1A = (1 << COM1A1) | (1 << WGM11);
    TCCR1B = (1 << WGM13) | (1 << WGM12) | (1 << CS11); // Prescaler 8
    ICR1 = 40000;                    // 20 ms period (50 Hz)
}

void servo_set(uint16_t us) {
    OCR1A = (us * 2);                // 1 µs = 2 ticks at prescaler 8
}

int main() {
    pwm_init();

    while (1) {
        servo_set(1000);  // 1.0 ms → left
        _delay_ms(1000);

        servo_set(1500);  // 1.5 ms → center
        _delay_ms(1000);

        servo_set(2000);  // 2.0 ms → right
        _delay_ms(1000);
    }
}
```

```C
// Uses Timer1 + Timer3 to drive three servos simultaneously
#include <avr/io.h>
#include <util/delay.h>

void pwm_init() {
    // Base servo on OC1A
    DDRB |= (1 << PB5);
    TCCR1A = (1 << COM1A1) | (1 << WGM11);
    TCCR1B = (1 << WGM13) | (1 << WGM12) | (1 << CS11);
    ICR1 = 40000;

    // Shoulder servo on OC3A
    DDRE |= (1 << PE3);
    TCCR3A = (1 << COM3A1) | (1 << WGM31);
    TCCR3B = (1 << WGM33) | (1 << WGM32) | (1 << CS31);
    ICR3 = 40000;
}

void base_set(uint16_t us)    { OCR1A = us * 2; }
void shoulder_set(uint16_t us){ OCR3A = us * 2; }

int main() {
    pwm_init();

    while (1) {
        base_set(1500);
        shoulder_set(1200);
        _delay_ms(1000);

        base_set(1800);
        shoulder_set(1600);
        _delay_ms(1000);
    }
}
```

```C
// Simulates a full motion routine: rotate base, lower arm, close gripper, lift, rotate back, release
#include <avr/io.h>
#include <util/delay.h>

void servo(uint8_t channel, uint16_t us) {
    switch(channel) {
        case 0: OCR1A = us * 2; break;   // Base
        case 1: OCR1B = us * 2; break;   // Shoulder
        case 2: OCR1C = us * 2; break;   // Elbow
        case 3: OCR3A = us * 2; break;   // Wrist
        case 4: OCR3B = us * 2; break;   // Gripper
    }
}

void move(uint16_t b, uint16_t s, uint16_t e, uint16_t w, uint16_t g) {
    servo(0,b); servo(1,s); servo(2,e); servo(3,w); servo(4,g);
}

int main() {
    // Assume timers already initialized as in previous examples

    while (1) {
        // Move to object
        move(1600, 1400, 1500, 1500, 1800);
        _delay_ms(1000);

        // Lower arm
        move(1600, 1700, 1700, 1500, 1800);
        _delay_ms(1000);

        // Close gripper
        servo(4, 1000);
        _delay_ms(800);

        // Lift object
        move(1600, 1300, 1400, 1500, 1000);
        _delay_ms(1000);

        // Rotate to drop zone
        move(1200, 1300, 1400, 1500, 1000);
        _delay_ms(1000);

        // Release
        servo(4, 1800);
        _delay_ms(800);
    }
}
```
