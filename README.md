# AVR-Measure-VCC-Battery-Voltage-Without-Using-I-O-Pin-on-tinyAVR-and-mega-AVR
Atmel's days, the community has succeeded with this trick, but until Microchip acquired & merged and published the APP note, there was no code available.  

Microchip AN2447 [http://ww1.microchip.com/downloads/en/AppNotes/00002447A.pdf](http://ww1.microchip.com/downloads/en/AppNotes/00002447A.pdf)  

there is a example and code available [https://wp.josh.com/2014/11/06/battery-fuel-guage-with-zero-parts-and-zero-pins-on-avr/](https://wp.josh.com/2014/11/06/battery-fuel-guage-with-zero-parts-and-zero-pins-on-avr/)  
[https://gist.github.com/bigjosh/ae9ffcd9fd0da9a25807#file-nopartsbatterygauge-c](https://gist.github.com/bigjosh/ae9ffcd9fd0da9a25807#file-nopartsbatterygauge-c)



### Test on ATmega168p, code example for avr-gcc

The Theory, make it more simple and straight.

Vbg = 1.1V, the internal voltage reference, [https://xiaolaba.wordpress.com/2016/12/01/avr-mcu-bandgap-reference/](https://xiaolaba.wordpress.com/2016/12/01/avr-mcu-bandgap-reference/)  
VCC = battery voltage  

Equation of ADC coversion by AVR MCU,  
```
ADC value = Vref x 1024 / Vin  
```

Re-arrange this equation abit  

Vin = Vref x 1024 / ADC value  

Vin = Battery voltage, to be determined
Vref = Vbg = 1.1V = 1100 mV
ADC value = read out
plug in those to the equation above, 
```  
Vin = 1100mV x 1024 / ADC value  
```  
