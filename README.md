# EPROM
## Electronic EPROM programmer

The EPROMs programmer circuit proposed in this Electronic article is designed to flash 
EPROMs memories (read-only memory whose contents can be erased by ultraviolet light 
or other means and reprogrammed using a pulsed voltage) such as the memory chip SMT27C256B from Texas instrument. 


![image](https://github.com/yoyoberenguer/EPROM/blob/main/27C256.jpg)

The chip SMT27C256B is composed of 256K bit electrically Programmable Read Only Memory (EPROM).  
The device is organized as 32K words by 8 bits  (32K  bytes). 

The version 1.0 is compatible only with the memory chip SMT27C256B (256M-Bytes), however the version 2.0 will
allow larger chip, such as the SMT27C512 to be recognized during the boot sequence with its electronic signature  
(mannufacturer and product code). The  TMS27C512  series  are 65536 by  8-bit(524 288-bit), ultraviolet (UV) 
light erasable, electrically programmable read-only  memories.

The version 3.0 (still on the drawing board) will be desgin with the 8-bit microprocessor Zilog Z80 
to allow the recognition of many other type and sizes of EPROM memories. 

### Synopsis version 1.0


![image](https://github.com/yoyoberenguer/EPROM/blob/main/schematic1_version1.PNG?raw=true)


### DC to DC step up converter
We are using the chip LT1073CN8 to provide a supply voltage around 13V. 
This component is versatile and easy to setup, just few components needs to be added to provide 
a steady DC voltage with low ripples. The LT1073CN comes into various packages LT1073-5 and LT1073-12 
including internal resistors R1 and R2 between GND (pin 5) and the SENSE (pin 8) input, however the 
version LT1073CN8 does not have these resistors and will be included in our project separately. 
The advantage of the LT1073CN is the operating voltage range provided from 1V to 30V, the low battery feature 
detection and the output current limiting.

This chip works with a wide range of batteries voltage sources such as 1.5V AA alkaline or 
lithium battery composing a source voltage within 1.5 - 5.5v. 
In our case, The DC to DC step up converter will be used with a +5V source voltage delivering
a voltage arround 13V with a maximum current of 130mA.

The DC to DC converter we be exclusively used for the device programming mode.
It will supply a steady DC voltage (VPP) range 12.75v ±0.25v to set the EPROM is programming
mode. During the programming mode the data bus Q0 - Q7 is placed in DATA IN mode  
Please refer to the datasheet for the absolute VPP voltage values (-2V to 14V for the SMT27C256).
The reading mode requires 100uA on pin 1 and the programming mode requires 50mA maximum. 
These current's values (IPP) and the voltage range (VPP) will set the DC to DC step up converter caracteristics 
for the programming operational mode.
The DC to DC converter can also be used optionally for the electronic signature mode by supplying a voltage on 
A9 (pin 24) of the SMT27C256
 
To activate the ES mode, the programming equipment must force 11.5V to 12.5V on address line A9 of the
M27C256B, with VCC = VPP = 5V. Two identifier bytes may then be sequenced from the device out-puts by toggling 
address line A0 from VIL to VIH . All other address lines must be held at V IL during Electronic Signature mode. 
Byte 0 (A0 = VIL ) rep-resents the manufacturer code and byte 1 (A0 = VIH ) the device identifier code. 
For the ST-Microelectronics M27C256B, these two identifier bytes are given in Table 4 and can be read-out on
outputs Q7 to Q0

DC to DC converter considerations: 
```
A9 (pin 29) DC voltage must not exceed –2 to 13.5
VPP (pin 1) DC voltage must not exceed -2  to 14
Minimum DC current of 100uA and maximum current 50mA
Source DC voltage +5V 
Output DC voltage 12.5V to be compatible with the electronic signature mode
```


### Programming pulse 

The EPROM SMT27C256 datasheet requires 95 - 100 micro seconds for the chip Enable program pulse width to write
a single word. This value may vary for each memory type and device operation mode.
To comply with a larger number of products, the writing pulse width will be variable to match 
the component programming requirement.

As the minimal programming pulse cannot be below 95us this will give us a reference for the theorital maximum 
programming pulse frequency that can be delivered to the EPROM to remain within its functional caracteristics. 

Based on consecutives minimal period of 95us the maximum frequency is around 10Khz (without taking into account 
the propagation delays and the rise and fall times of the signals) for the chip SMT27C256.

with a fixed time of 95us per word, the best timing for programming the SMT27C256 would be 32768 * 100us ≈ 3.2 seconds
This is by far an extrapolation and to stay within a safe margin for reliability, we will use a lower frequency to take
into account the rise and fall times of each signals and the various chip access time (data bus and address line)

Note: 
For the version 2.0 the chip TMS27C512 and SMT27C512 have respectively programming modes call **SNAP! Pulse programming**
& **PRESTO II Programming Algorithm**

Programming with PRESTO II involves the application of a sequence of 100μs program pulses to each byte until a correct verify occurs. 
During programming and verify operation, a MARGIN MODE circuit is automatically activated in order to guarantee that each cell is programmed with enough
margin. No overprogram pulse is applied since the verify in MARGIN MODE provides necessary margin to each programmed cell.




Considerations:

```
Programming pulse must be adjustable and within component specification (95 - 100 us)
The programming pulse is an active low signal impulse of few micro-secondes 5 - 0v sent to the pin NOT E (pin 20)
```

![image](https://github.com/yoyoberenguer/EPROM/blob/main/PulseGenerator_version2.0.PNG?raw=true)

