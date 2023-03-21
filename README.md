# EPROM
## Electronic EPROM programmer

The EPROMs programmer prototype describe in this Electronic article is designed to flash 
EPROMs memories (read-only memory whose contents can be erased by ultraviolet light 
or other means and reprogrammed using a pulsed voltage) such as the SMT27C256B from Texas instrument. 


![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/27C256.jpg)

The chip SMT27C256B is composed of 256K bit electrically Programmable Read Only Memory (EPROM).  
The device is organized as 32K words by 8 bits  (32K  bytes). 

The version 1.0 is compatible only with the memory chip SMT27C256B (256M-Bytes), however the version 2.0 will
allow larger chip, such as the SMT27C512 to be recognized during the boot sequence with its electronic signature  
(mannufacturer and product code). The SMT27C512 series are 65536 by 8-bit(524 288-bit), ultraviolet (UV) 
light erasable, electrically programmable read-only  memories.

The version 3.0 (still on the drawing board) will be desgin with the 8-bit microprocessor Zilog Z80 
to allow the recognition of many other type and sizes of EPROM memories. 

**M27C256 Logic diagram:**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/STM27C256B_logic_diagram.PNG?raw=true)

**M27C256 Dip connections:**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/M27C256B_pin_connections.PNG?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/M27C256B_signal_names.PNG?raw=true)

### EPROM programmer functional diagram version 1.0


![image](https://github.com/yoyoberenguer/EPROM/blob/main/schematic1_version1.PNG?raw=true)

---

### DC to DC step up converter
We are using the chip LT1073CN8 to provide a voltage around 12.75v. 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/LT1073_pin_configuration.PNG?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/LT1073_typical_application.PNG?raw=true)

This component is versatile and easy to setup, just few components needs to be added to provide 
a steady low ripples DC voltage. The LT1073CN comes into various packages LT1073-5 and LT1073-12 
including internal resistors R1 and R2 between GND (pin 5) and the SENSE (pin 8) input. 
However the version LT1073CN8 does not have these resistors and they will be included in our 
project separately to define the output voltage gain. 
Advantages of the LT1073CN are the operating voltage range from 1V to 30V, the low battery  
detection and the optional output current capping

This chip works with a wide range of batteries voltage sources such as 1.5V AA alkaline or 
lithium battery composing a source voltage within 1.5 - 5.5v. 
In our case, The DC to DC step up converter will be used with a +5V source voltage delivering
a voltage arround 12.75V with a maximum current of 130mA.

The DC to DC converter will be used for the SMT27C256B device programming mode.
It will supply a steady DC voltage (VPP) range 12.75v ±0.25v to set the EPROM is programming
mode. Please refer to the datasheet for the absolute VPP voltage values (-2V to 14V for the SMT27C256).
The reading mode requires 100uA on pin 1 and the programming mode requires 50mA maximum. 
These current's values (IPP) and the voltage range (VPP) will set the DC to DC step up converter caracteristics 
for the programming mode.
The DC to DC converter can also be used for the electronic signature mode, supplying a 12.5 voltage on 
the address line A9 (pin 24) of the SMT27C256. As the DC converter supply 12.75V a diode will be added to the pin 
A9 to drop few mV.
 
To activate the electronic signature mode, the programming equipment must force 11.5V to 12.5V on address line A9 of the
M27C256B, with VCC = VPP = 5V. Two identifier bytes may then be sequenced from the device out-puts by toggling 
address line A0 from VIL to VIH . All other address lines must be held at V IL during Electronic Signature mode. 
Byte 0 (A0 = VIL ) rep-resents the manufacturer code and byte 1 (A0 = VIH ) the device identifier code. 
For the ST-Microelectronics M27C256B, these two identifier bytes are given in Table 4 and can be read-out on
outputs Q7 to Q0

**Example of 5 to 12v converter: (note this LT1073N-12)**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/LT1073_5_to_12_converter.PNG?raw=true)

**DC to DC converter considerations:**

A9 (pin 29) DC voltage must not exceed –2 to 13.5

VPP (pin 1) DC voltage must not exceed -2  to 14

Minimum DC current of 100uA and maximum current 50mA

Source DC voltage +5V 

Output DC voltage 12.5V to be compatible with the electronic signature mode

### EPROM programmer DC to DC converter version 1.0

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/DC2DC_converter.PNG?raw=true)



Notes:

Two options for the prototype power supply voltage: 

1 - Build a DC to DC step up converter from a single AA 1.5V battery to output 5V DC chain with 
    another DC converter to build the +12.75V for the programming mode 
 
2 - Use a +5V bench power supply for the main project and build a DC to DC converter 5V to +12.75V for the 
    programming mode.

I choose the option 2, since the current requirement used by the prototype was close to 
200 - 300 mA during the prototype testing (mainly due to 7-seg display and diodes used for the 15-bit counter). 
A DC to DC +5v converter build from one or 2 AA battery will produce a maximum current of 200mA and 
not provide enough current for the prototype.  
This circuit is protected against the main power supply voltages inversion with the schotky diode 1N5817, 
therfore the output is not protected against short.If the output is directly connected to the ground
the maximum current will be provided except if RLIM is connected to pin 1 of the LT1073 (Rlim is set to 50 ohms
and limits the output current).Rlim must be at least 1/2W has the amount current might exceed the power
dissapation of an 1/4W resitor if the output is shorted.

Without Rlim the amount of current flowing through the inverse protection diode 1N5817 will be 
high and limited only by the bench power supply current carateristic and the diode will certainly break down. 
For a single 1.5V alkaline battery the max current going through the protection diode will be around 
100-200mA and within the diode tolerances 

The circuit is desiogned with a low voltage detection, the low voltage value is given by 
(330K/18K + 1) * 0.212 = 4.09 V
If the DC supply voltage (VCC) goes below 4.09V the signal LowVoltage_4.1V will be low (and close 
to zero volts). This signal can be used to determine if the power supply VCC is set correctly set when
powering up the prototype.
Output voltage is given by (910K/15K + 1) * 0.212 = 13.1V, the output value may vary with the choice 
of components. 
Choose inductor with low ESR.
All resistors are preferably 1% metal film 1/2W for better output voltage precision
Toggle_13V is 0/5V signal to enable the 13V output voltage 
When Toggle_13V is set to VCC the 13V output voltage is disabled and the voltage will 
start to drop to VCC or reach VCC if the circuit is turned on. 
 
Required capacitors with low ESR (tantalum preferably) 50V 

---

### Programming pulse 

The EPROM SMT27C256 datasheet requires 95 - 100 micro seconds for the chip Enable program pulse width to write
a single word. This value may vary for each memory type and device operation mode.
To comply with a larger number of products, the writing pulse width will be variable to match 
the component programming requirement.

As the minimal programming pulse cannot be below 95us this will give us a reference for the maximum 
programming pulse frequency that can be delivered to the EPROM.

Based on consecutives minimal period of 95us the maximum frequency is around 10Khz (without taking into account 
the propagation delays and the rise and fall times of each signals).

with a fixed time of 95us per word, the best timing for programming the SMT27C256 would be 32768 * 100us ≈ 3.2 seconds
This is by far an approximation and to stay within a safe margin for reliability, we will use a lower frequency to take
into account the rise and fall times of each signals and the various chip access time (data bus and address line)

Note: 
For the version 2.0 the chip TMS27C512 and SMT27C512 have respectively programming modes call **SNAP! Pulse programming**
& **PRESTO II Programming Algorithm**

Programming with PRESTO II involves the application of a sequence of 100μs program pulses to each byte until a correct verify occurs. 
During programming and verify operation, a MARGIN MODE circuit is automatically activated in order to guarantee that each cell is programmed with enough
margin. No overprogram pulse is applied since the verify in MARGIN MODE provides necessary margin to each programmed cell.




**Considerations:**


Programming pulse must be adjustable and within the Eprom
 specification (95 - 100 us)

The programming pulse is an active low signal impulse of few micro-secondes 5 - 0v sent to the pin NOT E (pin 20)

### EPROM programmer pulse generator version 1.0

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/PulseGenerator_version1.0.PNG?raw=true)

---

### Frequence comparator version 1.0

Frequency comparator (range in Kz)

The maximum programming frequency tolarated by the EPROM is defined 
by the NE555 device (around 10Khz) and used by the below KICAD circuit for reference.

This frequency correspond to the programming pulse period 95 - 105us used on 
pin 20 (E) of the EPROM SMT27C256B. The maximum programming frequency will be adjustable with a variable
resistor to be compatible for most type of EPROMs.
This circuit compare the maximum frequency(B) with the user defined/customized frequency 
coming out of the data selector/multiplexer SN74LS153 called (A) and labbeled CLK in the below 
diagram. 
This circuit will provide 3 bit of information regarding these frequencies
 (A<B, A>B, A=B) in real time.
 
 SN74LS85 Pinout
 
 ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/74ls85_pinout.PNG?raw=true)
 
 SN74LS85 function table
 
 ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/74LS85_function_table.PNG?raw=true)
 
Purpose of this circuit: 

When the selected frequency (A) is above the maximum programming pulse frequency  
a red diode will be lit, below that threshold a green diode will be lit up,
and finally when both frequencies are equals a third diode (yellow) will be on.
Note that multiples led can be lit at the same time when both frequencies are 
closing-in. 
This feature will prevent checksum error and bad copy of the SOURCE EPROM during 
the programation mode.

How it works?:

Both frequencies are decomposed into sub-frequencies by each stages of the synchronous 
UP 4-bits binary counter SN74LS193 dividing the frequency by 2. 
Each stages of the counter are compared bit by bit into the 16 pins SN74LS85 and the 
result is display with 3 diodes (A<B green, A>B red, A=B yellow).

The first counter reaching the count 16 has the highest frequency. If both 
counter reach 16 at the same time, then both frequencies are equal. 
To be noted that a carry over signal is generated when the SN74LS193 counter reach the max 
count(16) on pin 12 NOT CO. 
This signal will be monitored with an AND gate SN74LS08 
NOT C01 & NOT C02 and passed into a NOT gate to trigger a reset of both counters.

Resestting both counters at the same time is a paramount condition in order 
to have an accurate frequency comparison.
The result is display when at least one of the MSB most significant bit is high level 
to avoid displaying false positive A=B for both frequencies.
QC and QD from (A) counter are used with an OR GATE SN74LS32 to enable a 2N2222 transistor 
and supply 22mA to the leds (resistor network of 220R).

This false positive occure each time the counters are reset to zero simultinously. 
At T = 0 when both counters are reset, the outputs QA, QB, QC, QD will all be 
at low level and will force the result A=B for a fraction of a second, hence raising 
a false condition A=B (yellow led flickering).

Choosing the MSB will guarantee the most accurate result since both frequency will diverge 
significantly for each stages of the SN74LS193.

SN74LS193 pinout

 ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/74LS193_pinout.PNG?raw=true)

To be noted that:
 - 16 pins chip SN74LS85 does not have any enable pin and will continuoulsy compare both frequencies.
 - Both components 74LS08 and 74LS04 can be replaced with a NAND gate 
 - The pin 11 LOAD is not used and connected to VCC and inputs ABCD are not used and connected to ground 
  


![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/Frequency_comparator_version1.0.PNG?raw=true)

---

### Frequency Generator 

The frequency generator will provide the main frequency needed by the 15-bit counter (A0 - A14) to sequence every possible 
addresses on the "address bus".
As discussed previously, the highest theoritical frequency of an EPROM 27C256 is around 10 Khz (pin 20). 
The main frequency value will be set for twice the frequency value used by the Pulse generator, the reason will be explain 
in the next electronic stage (Frequency multiplexer)

The prototype will have different customizable frequency sources to help troobleshooting.

The board will be designed with a Schmitt trigger oscillator with an adjustable resistor to have a wide range of 
frequencies available (time constant given by R1 x C1).

A (32768khz) quarkz frequency oscillator using a Schmitt trigger CD40106BE chip. 
(If you are not using a schmitt trigger, add 2 resistors 47k feeding the +VCC and ground of the IC power supply and  
add an extra inverter to reshape the signal).  


![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20generator/Frequency_generator.PNG?raw=true)

And finally a step by step clock generator to troobleshoot on demand, a pulse is generated each time the contact SW1 is 
pressed 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20generator/Manual_clock_pulse.PNG?raw=true)

---

### Binary Counter 



The below diagram represent the 15-bits binary counter (A0-A14) made up from different chips 

- SN74F163N (4 bit synchronous counter) flip-flops triggering on the rising (positive-going) edge of CLK.
  The clear function is synchronous, and a low logic level at the clear (CLR) input sets all four of the flip-flop outputs
  to low after the next low-to-high transition of the clock, regardless of the levels of ENP and ENT. This
  synchronous clear allows the count length to be modified easily by decoding the Q outputs for the maximum
  count desired. The active-low output of the gate used for decoding is connected to the clear input to
  synchronously clear the counter to 0000 (LLLL).
  
  ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/SN74f163_pinout.PNG?raw=true)
  
  
 - DM74LS112A (Dual Negative-Edge-Triggered Master-Slave J-K Flip-Flop) with Preset, Clear, and Complementary Outputs
   This device contains two independent negative edge triggered J-K flip-flops with complementary outputs.

- HEF4060B is a 14-stage ripple-carry binary counter/divider and oscillator with three
  oscillator terminals (RS, REXT and CEXT), ten buffered outputs (Q3 to Q9 and Q11 to Q13) and an overriding asynchronous 
  master reset input (MR). As shown in the functional diagram and the pinout circuit the outputs Q0, Q1, Q2, Q10 are 
  missing from the chip and would have to be build separately in our design.
  The external oscillator (RS, REXT and CEXT) will be disregarded and we will use an external oscillator to cadence the 
  HEF4060B and the remaining flip flops & 4 - bit counter (SN74F163N)
  
 To be noted:
 * SN74F163N flip flops trigger on rising edge while the DM74LS112A & HEF4060B are negative edge triggering, this is 
   important to know in order to avoid a de-synchronization of the address bus through out the bits Q0-Q2, Q10, Q14, Q15.
 * Extra JK flip flop outputs Q10, Q14, Q15 added to the counter are build in asynchronous mode contrasting with 
   (HEF4060B & SN74F163N).This minor change will not affect the binary count and address frequencies. 
 * Q15 is hight when the count has reached the last address 7FFF (EPROM 256K), Q16 is reserved for 512K EPROM (version 2.0). 
 
 
  HEF4060 pinout 
  
  ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/HEF4060B_pinout.PNG?raw=true)

  HEF4060 functional diagram 
  
  * Note that the outputs Q0 - Q2, Q10 are missing from the functional diagram 
  ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/HEF4060_Functional_diagram.PNG?raw=true)


![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/Counter.PNG?raw=true)

---

### Display multiplexer 7-segments


7 Segements multiplexing. 

Display the memory content `COPIED OVER` to the target EPROM (16 bits, 2 bytes) from the address lines (A0-A14). 
The word/value is present on the 8 bits data lines (first 4 LSB bits D0-D3 represent the first byte (char)
and last 4 bits MSB (D4-D7) for the last char.

For example, if the source EPROM address $0000 contains the value $0A (#00001010 in binary), 
D0-D3 (#1010) will represent the hex value A and D4-D7 the hex value 0 (#0000).

The chip 74LS157 is a quad 2 inputs multiplexer allowing to split the data present on the data bus D0-D7 
into two nibbles of data, the less significant bits (D0-D3) and the most significant bit (D4-D7).

74LS157 outputs (Za, Zb, Zc, Zd) are connected to the EPROM decoder via (A0-A3) to convert the decimal value into 
an hexadecimal charactere (0-F). The chip 27C256 act as an BCD to 7-segment Display Decoders and map a decimal value to 
its equivalent in hexadecimal.The rest of the addresse lines on the EPROM are set to zero e.g (A4-A14, not used and forced to zero). 

The first 32 words of the EPROM (addresses $00000000 - $00000010) are encoded to represent the hexadecimal 
value 0 to F (7 segments representation of the decimal value present on the address line A0-A3, including 
the decimal point).

The value must take into consideration the type of 7-seg display (common cathode or comon anode)
PS Adresses $00000010 - $00000020 can also be populated with 7 segments code for common anode) if this is the case, 
we can add an single switch to set +5/GND logic 1 or zero to the EPROM A4 input that way, the switch will toggle between
comon cathode or comon anode displays.

For example: 
The below image represent the data of an EPROM addresses from $00000000 to $00000010, the data map the conversion
BCD to hex representation on an 7-segment display

**comon cathode**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/BCD%207-seg%20decoder.PNG?raw=true)

**comon anode** 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/BCD%207-seg%20decoder_comon_anode.PNG?raw=true)

The table below represent the addresse lines A0 - A4 and the 7-segements decoding. 
The first part of the table is the decoding for comon cathode display and the second part addresses starting 
at 00000010 to 0000001F is the decoding for comon anode.

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/BCD-7Segments.PNG?raw=true)

**EPROM outputs connection with 7-segment display:**

**7-Segment**      | EPROM data bus 
-------------------|-----------------------------------------------------
**a**              | D0
**b**              | D1
**c**              | D2 
**d**              | D3
**e**              | D4
**f**              | D5
**g**              | D6
**dot**            | D7

Both 7 segs displays are common anode, each displays will be lit during a short period
when a signal is sent (+5V) to the corresponding transistor to turn on the display. 
As the transistors Q2 and Q4 are connected to Q and ~{Q} they will be operational at different time.
CLK must be > 60hz to avoid flickering between the 7-segs HDSP-7501

The IC 74LS112 (JK flip flop) is producing a clock signal (half of the CLK frequency) and provide
a signal to turn on and off the 7 segments displays alternatively.

**Diagram**
![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/Multiplexing_diagram.PNG?raw=true)

---

### EPROM flashing 

The right EPROM is the source EPROM containing all the data that we want to transfer to 
the target EPROM on the left. 
The address bus A0 - A14 is common for both EPROM but it is also connected to the 15-bit counter used 
for incrementing the current addresses where to read the data. 
The data bus D0 - D7 is also comon to both EPROM and connected to the Multiplexing stage in order to display the 
value loaded on the bus from the source EPROM. 

During the rising edge and the first demi period of the clock signal (CLK), 
the source EPROM in READ mode, transfer the data to the data bus D0-D7 and at the same time, 
the destination EPROM is in PROGRAM mode. Please check the table below for the operation modes that 
explain how to set this modes.
We can see that the EPROM in READ mode will output the data on the data bus **Data out** and the EPROM in 
PROGRAM mode will receive the data **Data in**

These data are recorder into a 8 bits register made with 2 x 74LS173 IC when a pulse 
signal NOT pulse is sent to the common clock of the IC 74LS173 (pin 7). 

The NOT pulse signal (100us) will trigger the values present on data bus D0 - D7 to 
Q0 - Q7 (74LS688 of the comparator inputs) from Q0-Q3 of both 74LS173 registers.

The 74LS173 registers will keep the WORD until the next NOT pulse (next writing period), 
the WORD can then be compared with the SOURCE EPROM D0-D7 values during the VERIFY MODE

At the same time a positive pulse is sent to the EPROM (100us) on NOT CE entry to write the 
WORD in the EPROM at the current address A0-A14. 

On the falling edge of the main clock cycle, the DEST EPROM is shifting into the VERIGFY MODE and 
Q0 - Q7 present the data on the DATA bus (recorder WORD), While the source EPROM shift into the STANDBY MODE.
The WORD from the target EPROM is then compared with the 8-bit register (values present on Q0-Q7)

If both WORDS are identical the output (pin 19) of the comparator 74LS688 is low otherwise the 
output is +5V 


![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/27C256_operating_modes.PNG?raw=true)



![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/Flashing_diagram.PNG?raw=true)
