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

Synopsis


![image](https://github.com/yoyoberenguer/EPROM/blob/main/schematic1_version1.PNG?raw=true)



