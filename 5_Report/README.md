INTRODUCTION

In this project we are going to design a circuit for measuring temperature. This circuit is developed using “LM35”, a linear voltage sensor. Temperature is usually measured in “Centigrade” or “Faraheite”. “LM35” sensor provides output based on scale of centigrade.

LM35 is three pin transistor like device. It has VCC, GND and OUTPUT. This sensor provides variable voltage at the output based on temperature.



As shown in above figure, for every +1 centigrade raise in temperature there will be +10mV higher output. So if the temperature is 0◦centigrade the output of sensor will be 0V, if the temperature is 10◦ centigrade the output of sensor will be +100mV, if the temperature is 25◦ centigrade the output of sensor will be +250mV.

So for now with LM35 we get temperature in the form of variable voltage. This temperature dependent voltage is given as input to ADC (Analog to Digital Converter) of ATMEGA32A. The digital value after conversion obtained is shown in the 16x2 LCD as temperature.

Components Required
Hardware: ATMEGA32 Microcontroller, power supply (5v), AVR-ISP PROGRAMMER, JHD_162ALCD (16x2LCD), 100uF capacitor (two pieces), 100nF capacitor, LM35 Temperature Sensor.

Software: Atmel studio 6.1, progisp or flash magic.

Circuit Diagram and Explanation

In the circuit, PORTB of ATMEGA32 is connected to data port of LCD. Here one should remember to disable the JTAG communication in PORTC ot ATMEGA by changing the fuse bytes, if one wants to use the PORTC as a normal communication port. In 16x2 LCD there are 16 pins over all if there is a back light, if there is no back light there will be 14 pins. One can power or leave the back light pins. Now in the 14 pins there are 8 data pins (7-14 or D0-D7), 2 power supply pins (1&2 or VSS&VDD or gnd&+5v), 3rd pin for contrast control (VEE-controls how thick the characters should be shown), 3 control pins (RS&RW&E).

In the circuit, you can observe I have only took two control pins as this give the flexibility of better understanding. The contrast bit and READ/WRITE are not often used so they can be shorted to ground. This puts LCD in highest contrast and read mode. We just need to control ENABLE and RS pins to send characters and data accordingly.

The connections which are done for LCD are given below:

PIN1 or VSS ------------------ground

PIN2 or VDD or VCC------------+5v power

PIN3 or VEE---------------ground (gives maximum contrast best for a beginner)

PIN4 or RS (Register Selection) ---------------PD6 of uC

PIN5 or RW (Read/Write) -----------------ground (puts LCD in read mode eases the communication for user)

PIN6 or E (Enable) -------------------PD5 of uC

PIN7 or D0-----------------------------PB0 of uC

PIN8 or D1-----------------------------PB1 of uC

PIN9 or D2-----------------------------PB2 of uC

PIN10 or D3-----------------------------PB3 of uC

PIN11 or D4-----------------------------PB4 of uC

PIN12 or D5-----------------------------PB5 of uC

PIN13 or D6-----------------------------PB6 of uC

PIN14 or D7-----------------------------PB7 of uC

In the circuit you can see we have used 8bit communication (D0-D7) however this is not a compulsory, we can use 4bit communication (D4-D7) but with 4 bit communication program becomes a bit complex so I have choosed the 8 bit communication.

So from mere observation from above table we are connecting 10 pins of LCD to controller in which 8 pins are data pins and 2 pins for control. The voltage output provided by sensor is not completely linear; it will be a noisy one. To filter out the noise a capacitor needs to be placed at the output of the sensor as shown in figure.

Before moving ahead we need to talk about ADC of ATMEGA32A. In ATMEGA32A, we can give Analog input to any of eight channels of PORTA, it doesn’t matter which channel we choose as all are same. We are going to choose channel 0 or PIN0 of PORTA. In ATMEGA32A, the ADC is of 10 bit resolution, so the controller can detect a sense a minimum change of Vref/2^10, so if the reference voltage is 5V we get a digital output increment for every 5/2^10 = 5mV. So for every 5mV increment in the input we will have a increment of one at digital output. 

Now we need to set the register of ADC based on the following terms:

1.First of all we need to enable the ADC feature in ADC.

2. Since we  are measuring room temperature, we don’t really need values beyond hundred degrees (1000mV output of LM35). So we can set up maximum value or reference of ADC to 2.5V.

3. The controller has a trigger conversion feature, that means ADC conversion takes place only after an external trigger, since we don’t want that we need to set the registers for the ADC to run in continuous free running mode.

4. For any ADC, frequency of conversion (Analog value to Digital value) and accuracy of digital output are inversely proportional. So for better accuracy of digital output we have to choose lesser frequency. For lesser ADC clock we are setting the presale of ADC to maximum value (128). Since we are using the internal clock of 1MHZ, the clock of ADC will be (1000000/128).

These are the only four things we need to know to getting started with ADC. All the above four features are set by two registers.



RED (ADEN): This bit has to be set for enabling the ADC feature of ATMEGA.

BLUE(REFS1,REFS0): These two bits are used to set the reference voltage (or max input voltage we are going to give).  Since we want to have reference voltage 2.56V, REFS0 and REFS1 both should be set, by the table.


LIGHT GREEN (ADATE): This bit must be set for the ADC to run continuously (free running mode).

PINK (MUX0-MUX4): These five bits are for telling the input channel. Since we are going to use ADC0 or PIN0, we need not set any bits as by the table.


 BROWN (ADPS0-ADPS2): these three bits are for setting the prescalar for ADC. Sice we are using a prescalar of 128, we have to set all three bits.


DARK GREEN (ADSC): this bit set for the ADC to start conversion. This bit can be disabled in the program when we need to stop the conversion.

To make this project with Arduino, see this tutorial: Digital Thermometer using Arduino

Programming Explanation
Working of TEMPARATURE MEASUREMENT is best explained in step by step of C code given below:

 #include <avr/io.h> //header to enable data flow control over pins

#define F_CPU 1000000  //telling controller crystal frequency attached

#include <util/delay.h> //header to enable delay function in program

#define    E   5 //giving name “enable”  to 5th pin of PORTD, since it Is connected to LCD enable pin

#define RS  6 //giving name “registerselection” to 6th pin of PORTD, since is connected to LCD RS pin

void send_a_command(unsigned char command);

void send_a_character(unsigned char character);

void send_a_string(char *string_of_characters);    

int main(void)

{

DDRB = 0xFF; //putting portB and portD as output pins

DDRD = 0xFF;

_delay_ms(50);//giving delay of 50ms

DDRA = 0;//Taking portA as input.

ADMUX |=(1<<REFS0)|(1<<REFS1);//setting the reference of ADC

ADCSRA |=(1<<ADEN)|(1<<ADATE)|(1<<ADPS0)|(1<<ADPS1)|(1<<ADPS2);

//enabling the ADC, setting free running mode, setting prescalar 128

int16_t COUNTA = 0;//storing digital output

char SHOWA [3];//displaying digital output as temperature in 16*2 lcd

send_a_command(0x01); //Clear Screen 0x01 = 00000001

_delay_ms(50);

send_a_command(0x38);//telling lcd we are using 8bit command /data mode

_delay_ms(50);

send_a_command(0b00001111);//LCD SCREEN ON and courser blinking

ADCSRA |=(1<<ADSC);//starting the ADC conversion

while(1)

{

COUNTA = ADC/4; //since the resolution (2.56/2^10 = 0.0025) is 2.5mV there will be an increment of 4 for every 10mV input, that means for every degree raise there will be increment of 4 in digital value. So to get the temperature we have to divide ADC output by four.

send_a_string ("CIRCUIT DIGEST ");//displaying name

send_a_command(0x80 + 0x40 + 0); // shifting cursor  to 1st  shell  of second line

send_a_string ("Temp(C)=");// displaying name

send_a_command(0x80 + 0x40 + 8); // shifting cursor  to 9st  shell  of second line

itoa(COUNTA,SHOWA,10); //command for putting variable number in LCD(variable number, in which character to replace, which base is variable(ten here as we are counting number in base10))

send_a_string(SHOWA); //telling the display to show character(replaced by variable number) of first person after positioning the courser on LCD

send_a_string ("      ");

send_a_command(0x80 + 0);//retuning to first line first shell

}

}

void send_a_command(unsigned char command)

{

PORTA = command;

PORTD &= ~ (1<<RS); //putting 0 in RS to tell lcd we are sending command

PORTD |= 1<<E; //telling lcd to receive command /data at the port

 _delay_ms(50);

PORTD &= ~1<<E;//telling lcd we completed sending data

PORTA= 0;

}

void send_a_character(unsigned char character)

{

PORTA= character;

PORTD |= 1<<RS;//telling LCD we are sending data not commands

PORTD |= 1<<E;//telling LCD to start receiving command/data

_delay_ms(50);

PORTD &= ~1<<E;//telling lcd we completed sending data/command

PORTA = 0;

}

void send_a_string(char *string_of_characters)

{

while(*string_of_characters > 0)

{

send_a_character(*string_of_characters++);

}

}

Code

#include <avr/io.h>
#define F_CPU 1000000
#include <util/delay.h>
#include <stdlib.h>

#define enable            5
#define registerselection 6

void send_a_command(unsigned char command);
void send_a_character(unsigned char character);
void send_a_string(char *string_of_characters);

int main(void)
{
    DDRB = 0xFF;
    DDRA = 0;
    DDRD = 0xFF;
    _delay_ms(50);
    
    ADMUX |=(1<<REFS0)|(1<<REFS1);
    ADCSRA |=(1<<ADEN)|(1<<ADATE)|(1<<ADPS0)|(1<<ADPS1)|(1<<ADPS2);
    
    int16_t COUNTA = 0;
    char SHOWA [3];
     

    send_a_command(0x01); //Clear Screen 0x01 = 00000001
    _delay_ms(50);
    send_a_command(0x38);
    _delay_ms(50);
    send_a_command(0b00001111);
    _delay_ms(50);
    
    ADCSRA |=(1<<ADSC);
    while(1)
    {
        COUNTA = ADC/4;
        send_a_string ("CIRCUIT DIGEST");
        send_a_command(0x80 + 0x40 + 0);
        send_a_string ("Temp(C)=");
        send_a_command(0x80 + 0x40 + 8);
        itoa(COUNTA,SHOWA,10);
        send_a_string(SHOWA);
        send_a_string ("      ");
        send_a_command(0x80 + 0);
        
    }    
}

void send_a_command(unsigned char command)
{
    PORTB = command;
    PORTD &= ~ (1<<registerselection);
    PORTD |= 1<<enable;
    _delay_ms(20);
    PORTD &= ~1<<enable;
    PORTB = 0;
}

void send_a_character(unsigned char character)
{
    PORTB = character;
    PORTD |= 1<<registerselection;
    PORTD |= 1<<enable;
    _delay_ms(20);
    PORTD &= ~1<<enable;
    PORTB = 0;
}
void send_a_string(char *string_of_characters)
{
    while(*string_of_characters > 0)
    {
        send_a_character(*string_of_characters++);
    }
}

CONCULSION

Hence,we conclude that Thermometer is a device which helps to measure the temperature gradient or temperature of human beings by following some principles. Digital thermometers are the developed version of mercury thermometer. They offer fast and accurate result, are eco friendly and any one can use them.
