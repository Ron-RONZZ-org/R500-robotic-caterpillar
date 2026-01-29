# Rapport R500 robotic caterpillar

## Introduction

Aujourd’hui, nous allons vous présenter le robot chenille. Il s’agit d’un robot mobile qui se déplace grâce à des chenilles, comme celles des chars ou des engins de chantier. Ce type de robot est particulièrement adapté aux terrains difficiles, car il peut avancer sur le sable, la boue, les gravats ou les pentes. Le robot chenille est utilisé dans plusieurs domaines, notamment l’exploration, le sauvetage, l’industrie et même le domaine militaire. Dans cet exposé, nous allons vous expliquer de quoi est composé ce robot, de sa programmation et son utilisation.

## Specification de robot

- DOF (degrés de liberté): 8
- programmable
- plusieurs portes d'entrée pour les capteurs supplémentaires

| Élément      | Valeur                                                       |
|--------------|--------------------------------------------------------------|
| Alimentation | 5,2 V à 6 V (4 piles AAA, non incluses)                      |
| Processeur   | ATMEGA16                                                     |
| Servomoteurs | 8 mini‑servomoteurs                                          |
| Consommation | ~100 mA au repos, jusqu’à 3 A avec tous les servos en action |
| Hauteur      | 90 mm                                                        |
| Longueur     | 500 mm                                                       |
| Largeur      | 85 mm                                                        |

## Liste de components

- [liste de components](components-fr.md)

## Assemblage

> Avertissement de fabriquer : 

- Points à retenir :
  - Vis autotaraudeuses

## Programmation

### Motion

Notre robot est équipé avec les actionneurs. Les actionneurs sont contrôlé par le micro-processeur `ATMEGA16A‑AU 1326`.

Un code exemple simple pour controller un actionneur :

```C
#include <avr/io.h>
#include <util/delay.h>

// Convertit un angle (0–180°) en largeur d'impulsion (OCR1A)
uint16_t angle_to_ocr(int angle) {
    // 1 ms = 1000 µs → 1000 * 2 = 2000 cycles (à 8 MHz / prescaler 8)
    // 2 ms = 2000 µs → 4000 cycles
    return 2000 + (angle * 2000) / 180;
}

int main(void) {
    // PB1 = OC1A → sortie PWM
    DDRB |= (1 << PB1);

    // Timer1 en mode Fast PWM, TOP = ICR1
    TCCR1A = (1 << COM1A1) | (1 << WGM11);
    TCCR1B = (1 << WGM13) | (1 << WGM12) | (1 << CS11); // prescaler 8

    // Période PWM = 20 ms → ICR1 = 20000 µs * (F_CPU / prescaler)
    // F_CPU = 8 MHz → 1 µs = 8 cycles → 20 ms = 20000 µs → 20000 * 1 = 20000
    ICR1 = 20000;

    while (1) {
        // Servo à 0°
        OCR1A = angle_to_ocr(0);
        _delay_ms(1000);

        // Servo à 90°
        OCR1A = angle_to_ocr(90);
        _delay_ms(1000);

        // Servo à 180°
        OCR1A = angle_to_ocr(180);
        _delay_ms(1000);
    }
}
```

Un exemple qui contrôle tous les 8 actionneurs ensemble :

```c
#include <avr/io.h>
#include <util/delay.h>

#define SERVO_MIN 1000   // 1 ms
#define SERVO_MAX 2000   // 2 ms

// Convertit un angle (0–180°) en durée d'impulsion (µs)
uint16_t angle_to_us(uint8_t angle) {
    return SERVO_MIN + (angle * (SERVO_MAX - SERVO_MIN)) / 180;
}

int main(void) {
    // 8 servos sur PB0..PB7
    DDRB = 0xFF;  // tout le port B en sortie

    // Angles initiaux des 8 servos
    uint8_t angles[8] = {90, 90, 90, 90, 90, 90, 90, 90};

    while (1) {
        uint16_t total_time = 0;

        for (uint8_t i = 0; i < 8; i++) {
            uint16_t pulse = angle_to_us(angles[i]);

            // Mettre à 1 la broche du servo i
            PORTB = (1 << i);
            _delay_us(pulse);

            // Remettre à 0
            PORTB = 0x00;

            total_time += pulse;
        }

        // Compléter le cycle de 20 ms
        if (total_time < 20000)
            _delay_us(20000 - total_time);

        // Exemple : faire bouger le servo 0
        angles[0] = (angles[0] + 5) % 180;
    }
}
```



