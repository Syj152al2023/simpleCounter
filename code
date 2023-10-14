#define F_CPU 8000000UL
#include <avr/io.h>
#include <util/delay.h>
#include <avr/interrupt.h>
uint8_t segm[12] = {0b00111111, 0b0000110, 0b01011011, 0b01001111, 0b01100110, 0b01101101,
0b01111101, 0b00000111, 0b01111111, 0b01101111, 0, 0b00000010};
uint8_t razr[5] = {0, 2, 6, 8, 4};
uint8_t e = 0, d = 0, s = 0, t = 0;
volatile uint8_t counter = 0;
volatile uint8_t buttonPressed = 0;
volatile uint8_t RE_SET = 0;
void start() {
  PORTB |= (1 << 0);
  PORTB |= (1 << 4);
}
void stop() {
  PORTB &= ~(1 << 4);
  PORTB &= ~(1 << 0);
}
void wr_i2c(uint8_t val) {
  uint8_t x = 0;
  while (x < 8) {
    val & (1 << (7 - x)) ? PORTB &= ~(1 << 0) : PORTB |= (1 << 0);
    PORTB &= ~(1 << 4);
    PORTB |= (1 << 4);
    x++;
  }
  PORTB &= ~(1 << 0);
  if (PINB & (1 << 3)) {
    asm("rjmp m1");
  }
  PORTB &= ~(1 << 4);
  PORTB |= (1 << 4);
  PORTB |= (1 << 0);
}
void brigness(uint8_t b) {
  if (b > 15) { b = 15;}
  start();
  wr_i2c(0b11100000);
  wr_i2c(0xE0 + b);
  stop();
}
void _blink(uint8_t val) {
  if (val > 0b11) {val = 0b11;}
  start();
  wr_i2c(0b11100000);
  wr_i2c((val << 1) | 0b10000001);
  stop();
}
void reset(uint8_t val = 1) {
  if (val > 1) {val = 1;}
  wr_i2c(0b11100000);
  wr_i2c(val | 0b10000000);
  stop();
}
void _print(uint8_t addr, uint8_t val) {
  start();
  wr_i2c(0b11100000);
  wr_i2c(razr[addr]);
  wr_i2c(segm[val]);
  stop();
}
int main(void) {
  DDRB |= (1 << 0) | (1 << 4);
  PORTB &= ~(1 << 0) & ~(1 << 4);
  // Configure the button pin as input with pull-up resistor
  DDRB &= ~(1 << 3);
  PORTB |= (1 << 3);
  // Enable external interrupt on the button pin
  GIMSK |= (1 << PCIE);
  PCMSK |= (1 << PCINT3);
// Configure the button pin as input with pull-up resistor
  DDRB &= ~(1 << 2);
  PORTB |= (1 << 2);
  // Enable external interrupt on the button pin2
  GIMSK |= (1 << PCIE);
  PCMSK |= (1 << PCINT2);
  // Initialize the display
  start();
  wr_i2c(0b11100000);
  wr_i2c(0b00100001);
  stop();
  start();
  wr_i2c(0b11100000);
  wr_i2c(0b10000001);
  stop();
  start();
  wr_i2c(0b11100000);
  wr_i2c(0xe0 + 15);
  stop(); 
  // Enable global interrupts
  sei();
  while (1) { 
    if(!(PINB & (1 << PB2))) {
      counter = 0;
      RE_SET =0;
      e = counter % 10;
      d = (counter / 10) % 10;
      s = (counter / 100) % 10;
      t = (counter / 1000) % 10;
      
      _print(0, t);
      _print(1, s);
      _print(2, d);
      _print(3, e);
      
    if (buttonPressed) {
      // Increment the counter
      counter++;
      buttonPressed = 0;
      // Update the display with the new counter value
      e = counter % 10;
      d = (counter / 10) % 10;
      s = (counter / 100) % 10;
      t = (counter / 1000) % 10;
    
      _print(0, t);
      _print(1, s);
      _print(2, d);
      _print(3, e);
    }     
  }
}
// Interrupt service routine for the button press
ISR(PCINT0_vect) {
  if(!(PINB & (1 << 2))) { RE_SET = 1;}
  // Check if the button pin is low (button pressed)
  if (!(PINB & (1 << 3))) { buttonPressed = 1;}
}
// Define the label m1
asm("m1:");
