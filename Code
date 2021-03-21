/*
 * Reaction_Tester.cpp
 *
 * Created: 10/03/2021 10.06.52
 * Author : DunjinnLord
 */ 

#define F_CPU 16000000
#include <avr/io.h>
#include <avr/interrupt.h>
#include <stdlib.h>
#include <avr/eeprom.h>
#include <time.h>
#define __DELAY_BACKWARD_COMPATIBLE__
#include <util/delay.h>

int number_from_eeprom;

volatile uint8_t overflow_count;

volatile int waiter;
int ticks;

void servo() // Timer1 setup  for servo
{
	DDRB |= (1 << 1); // Set pin PB1 to output
	TCCR1A |= 0b10000000;  // Clear OC1A on Compare Match when upcounting and set when downcounting
	TCCR1B |= 0b00010010;  // Set WGM to be PWM, Phase and Frequency Correct with ICR1 as TOP
	ICR1 = 20000;  // Timer counts up 20000 and down 20000 for a total of 40000 to create a 20ms cycle
	OCR1A = 3000;  // Set pulse to move servo arm into position
}

void wait()  // Generates a random number used for delaying until the LED turns on
{
	//srand (time(NULL));
	waiter = rand() % (500 + 1 - 100) + 100;
	_delay_ms(waiter);
}

void timer_start()
{
	TCCR0B = 0b00000100;  //Prescaler 256
	TCNT0 = 0;  //Start timer
	TIMSK0 |= (1 << TOIE0);  //Enable overflow interrupt
	sei();  //Enable global interrupts
	overflow_count = 0;
}

ISR(TIMER0_OVF_vect)
{
	overflow_count++;  //Counts number of overflows
}

int main(void)
{
	DDRC |= (1 << 0); // Set PC0 as output
	PORTC = 0b00000000;
	PORTC |= (1 << PC5); // Button pullup
	PORTC |= (1 << PC4); // Button pullup
	
	
    while (1) 
    {
		if (!(PINC & (1 << PC5)))  // When the button is pressed
		{
			wait();  // Wait a random amount of time
			PORTC |= (1 << 0);  // Turn on LED
			timer_start();  // Start measuring the time
			
		}
		
		if (!(PINC & (1 << PC4)))  // When the other button is pressed
		{
			_delay_ms(20);  // Debounce
			ticks = (overflow_count * 255) + TCNT0;  // Calculate the total number of ticks
			eeprom_busy_wait();
			eeprom_write_word((uint16_t*) 0, ticks);  // Store the number of ticks in EEPROM
			eeprom_busy_wait();
			number_from_eeprom = eeprom_read_word((uint16_t*)0);  // This was just for testint the right number was stored in EEPROM
			PORTC ^= (1 << PC0);  // Turn LED off
			servo();  // Make the servo move into position according to the number of ticks (unfinished).
		}
    }
}

