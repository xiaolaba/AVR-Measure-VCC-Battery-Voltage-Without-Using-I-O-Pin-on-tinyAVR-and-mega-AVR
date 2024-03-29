/*  

# AVR-Measure-VCC-Battery-Voltage-Without-Using-I-O-Pin-on-tinyAVR-and-mega-AVR
Atmel's days, the community has succeeded with this trick, but until Microchip acquired & merged and published the APP note, there was no code available.  

Microchip AN2447 [http://ww1.microchip.com/downloads/en/AppNotes/00002447A.pdf](http://ww1.microchip.com/downloads/en/AppNotes/00002447A.pdf)  

there is a example and code available   
[https://wp.josh.com/2014/11/06/battery-fuel-guage-with-zero-parts-and-zero-pins-on-avr/](https://wp.josh.com/2014/11/06/battery-fuel-guage-with-zero-parts-and-zero-pins-on-avr/)  

[https://gist.github.com/bigjosh/ae9ffcd9fd0da9a25807#file-nopartsbatterygauge-c](https://gist.github.com/bigjosh/ae9ffcd9fd0da9a25807#file-nopartsbatterygauge-c)
.  
.  
.  



### Test on ATmega168p, code example for avr-gcc

The Theory, make it more simple and straight forward.  
![ATmega168P_Vbat_xiaolaba.jpg](ATmega168P_Vbat_xiaolaba.jpg)  
.  
.  

Vbg = 1.1V, the internal voltage reference, [https://xiaolaba.wordpress.com/2016/12/01/avr-mcu-bandgap-reference/](https://xiaolaba.wordpress.com/2016/12/01/avr-mcu-bandgap-reference/)  

Vbat = battery voltage, TBD  

Equation of ADC coversion by AVR MCU,  
```
ADC value = Vin / Vref x 1024  

Re-arrange this equation abit  

Vref = Vin * 1024 / ADC value  

Vin = Vbg = 1.1V = 1100 mV, fix
Vref = Vbat, Battery voltage, to be determined
ADC value = read out
plug in those to the equation above, 
  
Vbat = Vbg * 1024 / ADC value
Vbat = 1100mV * 1024 / ADC value 
```  

.  
.  
.  

### circuit  
![xiaolaba_avr_bat_vol_circuit.jpg](xiaolaba_avr_bat_vol_circuit.jpg)  
.  
.  


### my code to the test  

  

```
*/

long detectVcc() {
    // Read 1.1Vbg or 1.3Vbg reference against AVcc
    // set the reference to Vcc and the measurement to the internal 1.1V or 1.3V reference
    // 2023-SEP-25, Atmega8, testing, 1.3Vbg (ADMUX, bit MUX3:0 = 1110)

    #if defined(__AVR_ATmega32U4__) || defined(__AVR_ATmega1280__) || defined(__AVR_ATmega2560__)
        ADMUX = _BV(REFS0) | _BV(MUX4) | _BV(MUX3) | _BV(MUX2) | _BV(MUX1);
    #elif defined (__AVR_ATtiny24__) || defined(__AVR_ATtiny44__) || defined(__AVR_ATtiny84__)
        ADMUX = _BV(MUX5) | _BV(MUX0);
    #elif defined (__AVR_ATtiny25__) || defined(__AVR_ATtiny45__) || defined(__AVR_ATtiny85__)
        ADMUX = _BV(MUX3) | _BV(MUX2);
    #elif defined (__AVR_ATmega168__) || (__AVR_ATmega168p__) || defined(__AVR_ATmega328p__) || defined(__AVR_ATmega328__)
        ADMUX = _BV(REFS0) | _BV(MUX3) | _BV(MUX2) | _BV(MUX1);   // AREF pin = Vcc, disable left adjusted result, uses 1.1Vbg
    #elif defined (__AVR_ATmega8__) || (__AVR_ATmega8L__)
        ADMUX = _BV(REFS0) | _BV(MUX3) | _BV(MUX2) | _BV(MUX1);   // AREF pin = Vcc, Atmega8/L, MUX3:0 = 1110, uses bandgrap reference 1.3Vbg, 2023-09-05 update       
    #else
        ADMUX = _BV(REFS0) | _BV(MUX3) | _BV(MUX2) | _BV(MUX1);
    #endif
    
    delay(2); // Wait for Vref to settle
    ADCSRA |= _BV(ADSC); // Start conversion
    while (bit_is_set(ADCSRA,ADSC)); // measuring
    
    uint8_t low  = ADCL; // must read ADCL first - it then locks ADCH
    uint8_t high = ADCH; // unlocks both
    
    long result = (high<<8) | low;
#if defined (__AVR_ATmega8__) || (__AVR_ATmega8L__)
    // Vbat = 1300mV * 1024 / ADC value
    result = 1331200L / result; // Calculate Vcc (in mV); 1331200 = 1.3*1023*1000
#else
    // Vbat = 1100mV * 1024 / ADC value
    result = 1125300L / result; // Calculate Vcc (in mV); 1125300 = 1.1*1023*1000
#endif

    return result; // Vcc in millivolts
}
 
void setup() { Serial.begin(115200); Serial.println("bandgap measurement\n"); }
 
void loop() { Serial.print( detectVcc(), DEC ); Serial.println (" mV"); delay(1000); }

/*
```
*/
