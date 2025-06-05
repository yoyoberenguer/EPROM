# EPROM
## Electronic EPROM programmer

The EPROM programmer prototype described in this article is designed to program EPROM memory devices—read-only memory chips that can be erased
(typically using ultraviolet light) and reprogrammed using a controlled programming voltage pulse. 
This includes devices such as the **SMT27C256B** from **Texas Instruments**.
The **SMT27C256B** is a 256K-bit Electrically Programmable Read-Only Memory (EPROM), organized as **32K × 8 bits** (32 KB).

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/27C256.jpg)


**M27C256 Logic diagram:**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/STM27C256B_logic_diagram.PNG?raw=true)

**M27C256 Dip connections:**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/M27C256B_pin_connections.PNG?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/M27C256B_signal_names.PNG?raw=true)


## Future Versions of the Project

**Version 1.0** (current version) of the EPROM programmer is designed exclusively for compatibility with the SMT27C256B device.

**Version 2.0** introduces support for larger EPROMs, such as the **SMT27C512**, by detecting the memory’s **electronic signature** (manufacturer and product codes) 
during the boot sequence. The SMT27C512 series features **64K × 8 bits** (512 Kbit) organization and, like the 256B, is ultraviolet (UV) light erasable and electrically programmable.

**Version 3.0** (currently in development) will incorporate the **Zilog Z80 8-bit microprocessor**, enabling broader compatibility with a wide range of EPROM types and sizes, 
further extending the programmer’s capabilities.


---



## Project functional diagram

![image](https://github.com/yoyoberenguer/EPROM/blob/main/schematic1_version1.PNG?raw=true)

---

## DC to DC step up converter module
We are using the chip LT1073CN8 to provide a voltage around 12.75v. 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/LT1073_pin_configuration.PNG?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/LT1073_typical_application.PNG?raw=true)

The **LT1073CN8** is a versatile and easy-to-configure `DC-DC converter`. 
Only a few external components are required to achieve a stable output voltage with minimal ripple.

This device is available in several package variants, such as the LT1073-5 and LT1073-12, which include internal feedback 
resistors (R1 and R2) connected between **GND** (pin 5) and the **SENSE input** (pin 8). 
However, the **LT1073CN8** version does not include these internal resistors. 
In our project, we will add them externally to define the desired output voltage and gain.

Key advantages of the **LT1073CN8** include:

  . Wide input voltage range: 1V to 30V

  . Built-in low battery detection

  . Optional output current limiting

This makes it well-suited for battery-powered and low-voltage applications where efficiency and reliability are critical.

This chip is compatible with a wide range of battery voltage sources, including 1.5V AA alkaline or lithium cells, typically 
providing input voltages between **1.5V** and **5.5V**.In our application, the DC-DC step-up converter will operate from 
a **+5V** input and output approximately **12.75V**, with a maximum load current of **130mA**.

The **DC-DC** converter will be used to supply the programming voltage (V<sub>PP</sub>) required by the **SMT27C256B** device during **programming mode**.
It provides a stable DC output of **12.75V ±0.25V**, which is within the required range to enable EPROM programming mode.

Please refer to the **SMT27C256** datasheet for the absolute V<sub>PP</sub> voltage ratings, which range from **-2V to 14V**.

- In **read mode**, pin 1 (V<sub>PP</sub>) draws a minimal current of approximately **100µA**.  
- In **program mode**, the current requirement increases significantly, with a maximum of **50mA**.

These current requirements (**I<sub>PP</sub>**) and the specified voltage range (**V<sub>PP</sub>**) directly determine 
the performance criteria for the DC-DC step-up converter used during programming.

The DC-DC converter will also support the **electronic signature mode**, which requires a voltage of **12.5V** on the **A9 address line** (pin 24).
Since the converter outputs **12.75V**, a **diode** will be placed in series with pin A9 to drop the voltage slightly, bringing it closer to the required level.
  
To activate **Electronic Signature mode** on the **M27C256B**, the programming equipment must apply a voltage between **11.5V and 12.5V** 
to address line **A9**, while keeping **V<sub>CC</sub> = V<sub>PP</sub> = 5V**. Once in this mode, two identifier bytes can be read from the
device outputs by toggling **address line A0** between logic low (**V<sub>IL</sub>**) and logic high (**V<sub>IH</sub>**).
All other address lines must be held at **logic low (V<sub>IL</sub>)** during this process.

- **A0 = V<sub>IL</sub>**: Byte 0 (Manufacturer Code) is output.
- **A0 = V<sub>IH</sub>**: Byte 1 (Device Identifier Code) is output.

For the **STMicroelectronics M27C256B**, the values of these two identifier bytes are provided in **Table 4** and are read on **outputs Q7 to Q0**.

**Example of 5 to 12v converter: (note this LT1073N-12)**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/LT1073_5_to_12_converter.PNG?raw=true)


#### DC to DC Converter Considerations

When designing or selecting a DC to DC converter for use with the **M27C256B** in Electronic Signature mode, the following electrical constraints must be observed:

- **A9 (Pin 29)**:  
  - DC voltage must remain within the range **–2V to +13.5V**

- **V<sub>PP</sub> (Pin 1)**:  
  - DC voltage must remain within the range **–2V to +14V**

- **Current Requirements**:  
  - Minimum DC current: **100 µA**  
  - Maximum DC current: **50 mA**

- **Input Voltage**:  
  - Source DC voltage: **+5V**

- **Output Voltage**:  
  - Required DC output: **12.5V**  
  - This output must be compatible with the requirements for activating **Electronic Signature mode**


### DC to DC converter version 1.0 KICAD

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/DC2DC_converter.PNG?raw=true)



#### Notes: Prototype Power Supply Voltage Options

Two options were considered for the prototype's power supply configuration:

##### Option 1:
- Build a **DC-to-DC step-up converter** powered by a **single AA 1.5V battery** to output **5V DC**.
- Chain this with another **DC-to-DC converter** to generate **+12.75V** for the programming mode.

##### Option 2:
- Use a **+5V bench power supply** as the main power source.
- Build a **DC-to-DC converter (5V to +12.75V)** to supply the voltage required for the programming mode.


#### Chosen Solution: Option 2

Option 2 was selected because the prototype's current consumption during testing was in the range of **200–300 mA**, mainly due to:
- **7-segment display**
- **Diodes** used in the **15-bit counter**

A DC-to-DC converter powered by **one or two AA batteries** would typically be limited to a maximum current of around **200 mA**, which would not meet the prototype's requirements.


#### Protection and Current Limiting

The circuit includes **reverse voltage protection** using a **Schottky diode (1N5817)**. However, the **output is not protected against short circuits**.

- If the output is directly connected to ground, the **maximum current will flow**, unless current limiting is implemented using the **R<sub>LIM</sub> resistor** on **Pin 1 of the LT1073**.
- **R<sub>LIM</sub> is set to 50 Ω**, which helps limit the output current.
- **R<sub>LIM</sub> must be rated at least 1/2W**, as a 1/4W resistor may not handle the power dissipation if the output is shorted.


#### Diode Stress Considerations

Without the R<sub>LIM</sub> resistor:
- A high current will flow through the **1N5817 protection diode**, limited only by the bench power supply’s current characteristics.
- This may lead to **diode failure**.

If using a **single 1.5V alkaline battery**:
- The maximum current through the 1N5817 is around **100–200 mA**, which is within the diode’s safe operating limits.


#### Low Voltage Detection and Output Voltage Control

The circuit includes a **low voltage detection** mechanism. The threshold voltage is calculated as:

$$
V_{\text{low}} = \left( \frac{330\text{k}\Omega}{18\text{k}\Omega} + 1 \right) \times 0.212\,\text{V} = 4.09\,\text{V}
$$

If the DC supply voltage (**V<sub>CC</sub>**) drops below **4.09 V**, the `LowVoltage_4.1V` signal goes **low** (close to 0 V).
This signal is useful for verifying whether V<sub>CC</sub> is properly set when powering up the prototype.


#### Output Voltage

The expected output voltage is given by:

$$
V_{\text{out}} = \left( \frac{910\text{k}\Omega}{15\text{k}\Omega} + 1 \right) \times 0.212\,\text{V} = 13.1\,\text{V}
$$

> **Note:** The actual output may vary depending on component tolerances.


#### Component Guidelines

- **Inductor**: Select one with **low ESR** (Equivalent Series Resistance).
- **Resistors**: Use **1% tolerance**, **metal film**, **½ W** resistors to ensure accurate voltage settings.
- **Capacitors**: Prefer **low ESR** capacitors (ideally **tantalum**) rated for **50 V** operation.


#### Output Voltage Control

- `Toggle_13V` is a **0 V / 5 V** logic signal used to control the **13 V output**.
- When `Toggle_13V` is set to **V<sub>CC</sub>**, the **13 V output is disabled** and will drop to V<sub>CC</sub>, or remain there if the circuit is powered.

This design ensures accurate voltage detection and reliable control over the high-voltage output.

### DC to DC converter version 1.0 PCB

![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/DC2DC_converter_board.png?raw=true)


![image](https://github.com/yoyoberenguer/EPROM/blob/main/LT1073/DC2DC_converter.png?raw=true)

---

## EPROM Programming module

According to the SMT27C256 datasheet, the **Chip Enable (CE) program pulse width** required to write a single byte is defined as **95–100 µs**.
This value can vary depending on the **EPROM type** and **device operation mode**. To support a wider range of compatible devices, the programming
pulse width in this design will be **variable**, allowing it to meet the specific timing requirements of each memory component.


**Minimum Pulse Width and Maximum Frequency**

Since the minimum allowed programming pulse is **95 µs**, this value sets an upper limit on the frequency at which write pulses can be issued to the EPROM. The theoretical **maximum programming frequency** is calculated as:

$$
f_{\text{max}} = \frac{1}{95\,\mu\text{s}} \approx 10.5\,\text{kHz}
$$

> This estimate does **not** account for signal **propagation delays**, nor the **rise and fall times** of control signals.

**Estimated Programming Time**

Assuming a fixed **programming time of 100 µs per byte**, the time required to fully program the SMT27C256 (32,768 bytes) is:

$$
T_{\text{program}} = 32{,}768 \times 100\,\mu\text{s} \approx 3.28\,\text{seconds}
$$

This is a **rough estimate**. For **reliable operation**, the actual programming frequency will be set **lower**, to account for:

- Rise and fall times of address, data, and control signals
- Access times and delays within the EPROM itself
- Variations across devices and environmental conditions

**Considerations:**

- The **programming pulse width** must be **adjustable** and remain within the EPROM's specified range of **95–100 µs**.
- The **programming pulse** is an **active-low signal** — a short-duration pulse transitioning from **5 V to 0 V**.
- This pulse is applied to the $\overline{E}$ **(Chip Enable)** pin, which corresponds to **pin 20** on the EPROM.
  
#### EPROM programmer pulse generator KICAD 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/M27C256/PulseGenerator_version1.0.PNG?raw=true)

---

## Frequence comparator 

The **maximum programming frequency** tolerated by the EPROM is primarily defined by the **NE555 timer circuit**, which, in this design, 
is configured to operate around **10 kHz**. This reference frequency is implemented in the KiCad schematic shown below.

This frequency corresponds to the **programming pulse period** of approximately **95–105 µs**, applied to **pin 20  $\overline{E}$** of the **SMT27C256B EPROM**.
The actual programming frequency is made **adjustable via a variable resistor**, allowing compatibility with a broad range of EPROM types.

The frequency comparator circuit compares:

- **Reference frequency (B)** — generated by the NE555 circuit
- **User-defined frequency (A)** — produced by the output of a **data selector/multiplexer** (SN74LS153), labeled **CLK** in the diagram

The comparator outputs **3 bits of real-time status information**:

- **A < B**
- **A > B**
- **A = B**

This allows dynamic monitoring and adjustment of the programming clock to ensure it remains within safe limits for the target EPROM.
 
**SN74LS85 Pinout**
 
 ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/74ls85_pinout.PNG?raw=true)
 
**SN74LS85 function table**
 
 ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/74LS85_function_table.PNG?raw=true)


#### Frequency Comparison Display Logic

When the selected frequency **(A)** exceeds the maximum allowed **programming pulse frequency**, a **red LED** will light up.  
If the selected frequency falls **below** the threshold, a **green LED** will illuminate.  
When both frequencies are **equal**, a **yellow LED** will be turned on.

> **Note:** Multiple LEDs may briefly light up simultaneously when the two frequencies are closely matched.  
> This behavior provides a visual indication of near-synchronization and helps avoid programming errors such as checksum mismatches or incorrect copying of the source EPROM during programming mode.

#### How It Works

Both frequencies — the **user-selected frequency (A)** and the **reference frequency (B)** — are fed into separate **synchronous 4-bit binary counters** (**SN74LS193**).
Each counter divides its input frequency by powers of two at each stage (QA through QD), effectively producing a set of sub-frequencies.

The counter outputs are then compared **bit by bit** using a **4-bit magnitude comparator** (**SN74LS85**). The comparator outputs three signals indicating:

- **A < B** → Green LED ON  
- **A > B** → Red LED ON  
- **A = B** → Yellow LED ON

#### Counter Overflow Detection

When either counter reaches a count of **16**, it generates a **carry-out** signal on **pin 12 ($\overline{CO}$)**. 
This signal is used to determine which counter reached the limit first — and thus which input frequency is higher.

The two **$\overline{CO}$ signals** (from both counters) are monitored using a dual-input **AND gate** (**SN74LS08**). 
The output of this AND gate is then inverted using a **NOT gate** to create a **reset pulse**, which resets both counters simultaneously.

> Resetting both counters **at the same time** is crucial for accurate and stable frequency comparison.

#### False Positive Suppression (A = B)

At every reset (T = 0), both counters start from zero, causing all outputs (QA–QD) to be LOW. 
This briefly satisfies the condition **A = B**, incorrectly lighting the yellow LED for a fraction of a second.

To eliminate this **false positive**, the circuit only enables the LED display **after at least one most significant bit (MSB)** is HIGH. This is done by:

- Monitoring **QC and QD** of the A-counter using an **OR gate** (**SN74LS32**)
- The OR gate output enables a **2N2222 transistor**, which switches **22 mA** to the LED display via a **220 Ω resistor network**

This gating ensures that a valid frequency comparison is only displayed when there is enough time resolution to reliably differentiate between A and B.

### Summary

- Frequencies are divided and compared using **SN74LS193** and **SN74LS85**
- Reset logic ensures synchronized measurement cycles
- Output accuracy is enhanced by gating the display with MSBs
- Red, green, and yellow LEDs provide intuitive visual feedback
- This logic ensures robust and accurate EPROM programming, minimizing the risk of data corruption

**SN74LS193 pinout**

 ![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/74LS193_pinout.PNG?raw=true)

#### Additional Notes

- The **SN74LS85** (16-pin magnitude comparator) does **not** include an enable pin. It continuously compares both input values in real time.
- The **SN74LS08** (AND gate) and **SN74LS04** (inverter) can optionally be replaced with a single **NAND gate** for logic simplification, depending on the logic level inversion required.
- **Pin 11 (LOAD)** on the SN74LS85 is **unused** in this design and should be connected to **V<sub>CC</sub>** to ensure proper logic level.
- The **ABCD parallel data inputs** of the SN74LS85 are also **not used** and must be tied to **ground** to avoid floating inputs and potential erratic behavior.


## Frequence comparator KICAD

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/Frequency_comparator_version1.0.PNG?raw=true)

## Frequence comparator PCB
![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/EPROM_FrequencyCompare.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/EPROM_FrequencyCompare_components.png?raw=true)

---

### Frequency Generator 

The frequency generator will provide the main frequency needed by the 17-bit counter (A0 - A16) to sequence every possible 
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



The below diagram represent the 17-bits binary counter (A0-A16) made up from different chips 

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


![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/Kicad_CounterSchematic.PNG?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/counter.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/counter_composants.png?raw=true)

---

### Display multiplexer 7-segments


7 Segements multiplexing. 

Display the memory content `COPIED OVER` to the target EPROM (16 bits, 2 bytes) from the address lines (A0-A14). 
The byte is present on the 8 bits data lines (first 4 LSB bits D0-D3 represent the first byte (char)
and last 4 bits MSB (D4-D7) for the last char.

For example, if the source EPROM address $0000 contains the value $0A (#00001010 in binary), 
D0-D3 (#1010) will represent the hex value A and D4-D7 the hex value 0 (#0000).

The chip 74LS157 is a quad 2 inputs multiplexer allowing to split the data present on the data bus D0-D7 
into two nibbles of data, the less significant bits (D0-D3) and the most significant bit (D4-D7).

74LS157 outputs (Za, Zb, Zc, Zd) are connected to the EPROM decoder via (A0-A3) to convert the decimal value into 
an hexadecimal charactere (0-F). The chip 27C256 act as an BCD to 7-segment Display Decoders and map a decimal value to 
its equivalent in hexadecimal.The rest of the addresse lines on the EPROM are set to zero e.g (A4-A14, not used and forced to zero). 

The first 32 bytes of the EPROM (addresses $00000000 - $00000010) are encoded to represent the hexadecimal 
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
As the transistors Q2 and Q4 are connected to Q and NOT Q they will be operational at different time.
CLK must be > 60hz to avoid flickering between the 7-segs HDSP-7501

The IC 74LS112 (JK flip flop) is producing a clock signal (half of the CLK frequency) and provide
a signal to turn on and off the 7 segments displays alternatively.

**Diagram**
![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/Multiplexing_diagram.PNG?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/EPROMDisplay.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/EPROMDisplay_components.png?raw=true)

---

### EPROM flashing 

The right EPROM is the source EPROM containing all the data that we want to transfer to 
the target EPROM on the left. 
The address bus A0 - A16 is common for both EPROM but it is also connected to the 15-bit counter used 
for incrementing the current addresses where to read the data. 
The data bus D0 - D7 is also common to both EPROM and connected to the Multiplexing stage in order to display the 
value loaded on the bus from the source EPROM. 

During the rising edge and the first demi period of the clock signal (CLK), 
the source EPROM is in READ mode and transfer the data to the data bus D0-D7. 
At the same time, the destination EPROM is in PROGRAM mode. Please check the table below for the operation modes that 
explain how to set this modes.
We can see that the EPROM in READ mode will output the data on the data bus **Data out** and the EPROM in 
PROGRAM mode will receive the data **Data in**

These data D0-D7 are recorder into a 8 bits register made with 2 x 74LS173 IC when a pulse 
signal (**NOT pulse**) is sent to the common clock of the IC 74LS173 (pin 7) during the READ/PROGRAM MODE sequences

The NOT pulse signal (100us) will also push the values present on the data bus D0 - D7 to 
the input Q0 - Q7 of the comparator.

The 74LS173 registers will keep the Byte read values from the SOURCE EPROM until the next NOT pulse (next writing period),
that byte can then be compared with the TARGET EPROM D0-D7 values during the VERIFY MODE (TARGET EPROM data bus being in **data out** mode)

On the falling edge of the main clock cycle, the TARGET EPROM is shifting into the VERIGFY MODE and 
Q0 - Q7 present the data on the DATA bus (recorded byte), While the source EPROM shift into the STANDBY MODE.
The Byte from the target EPROM is then compared with the 8-bit register.

If both byte values are identical the output (pin 19) of the comparator 74LS688 is low otherwise the 
output +5V 


![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/27C256_operating_modes.PNG?raw=true)


**Source EPROM and Target EPROM (left)**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/Flashing_diagram.PNG?raw=true)

**Data validation**


![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/EPROM_ParityCheck_diagram.PNG?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/EPROM_ParityCheck.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/EPROM_ParityCheck_components.png?raw=true)





The data validation is the circuit checking if the byte copied over to the target EPROM is valid or erroneous. 
It is build with two 74LS173 (forming a 8-bit D-type register) and a 8-bits comparator 74LS688. The concept is very simple, 
during the **PROGRAM MODE** (first half of the CLK period, when the signal is high level) a pulse approximatively 100us 
is sent to the target EPROM to write the data from the data bus D0-D7 and 
to the registers to memorize the original byte.
In the second half of the CLK signal (when the level is low), the target EPROM is in **VERIFY MODE** (data out) and shows the 
recorded byte to the data bus D0-D7. The data present on the data bus is compared using the 74LS688 comparator (input Q) to determine 
if the byte recorder within the EPROM is identical to the byte recorded in the 8-bit register (input P).


The NE555 is used in bistable mode to stop the 15-bit counter when a mismatch error is detected. 
A mismatch can occure if the byte copied in the target EPROM differs from the original Byte 
fron the source EPROM during the validation process. If the a mismatch occure, the main counter needs to 
stop incrementing the address values (A0-A14) in order to perform 3 sequetial retries. 
The main counter can resume when the mismatch is no longer present or when 3 retries have been done. 

To resume: 

When a mismatch occure, a +5v signal is sent to the NE555 pin 2 (treshold) to set the output Q (pin 3) 
to a high level +5v (**CounterLock signal**).
The **CounterLock signal** will stop the main 15-bit counter and stop incrementing the address bus A0 - A16.

As a byte mismatch has occured, the comparator 74ls688 output NOT P=Q will goes high. The AND gate will then 
change state when NOT CLK goes HIGH (second half of the clock signal period also known as the **VERIFY MODE**). 
When the AND gate 74ls08 change state, a raising edge signal is sent to the input 1 of the 74LS112 flip flop initiating the retry count.

Both 74ls112 will count from 0 to 2 and will be stopped by a NAND gate (74LS00) connected to both output pin NOT Q0 and Q1 to valid 3 retries). 
When the flip flop counter reaches 3 retries, the NAND gate 74LS00 will trigger a low level signal **loopCount** 
(see table below) to reset the NE555 oscillator resulting in pin 3 (Q) to change state (signal **CounterLock**).
**CounterLock** signal being now low, the main counter A0-A14 will resume incrementing the address bus

Q0  |  NOT Q0 |  Q1   | LoopCount NOT Q0 & Q1
----|---------|-------|--------------------------------
0   |    1    |   0   |   1
1   |    0    |   0   |   1 
0   |    1    |   1   |   0  ==> Third iteration LoopCount = 0V, the low level will trigger a reset.
  

The 360R resistor and the 220pF capacitor connected at the NE555 output pin 3 form a timer with 
a time constant RC. When the voltage accross the Capacitor goes below the low level 
threshold (VIL) on the reset pin 15 (NOT R) chip 74LS112 both JK flip flops are reset. 

Note that the NAND output (**LoopCount**) can be connected directly to the flip flip counters (74LS112) to reset it instantaneously via the reset pin 15 (NOT R) of both chips.However I did not opt for that scenario due to the fact that this will trigger a short low pulse via **LoopCount** at the NAND output with a maximum width of 10-20ns and this signal duration will not be tolerated by the NE555 on pin 4 (reset). The reset event will be ignored by the NE555 due to the signal hold time delay not being sufficient.
```
**RC time constant**
VIL 0.8v (LOW Level Input Voltage)  74LS112 
VOH 3.3v (High-level output voltage) NE555 
VIL = VOH * exp(-t/RC)
t = -RCln(0.8/3.3) and RC = -t/ln(0.8/3.3)
```
We are using a 220pF capacitor value and this gives us 112ns delay after the count of 3 by the flip flop. 
**LoopCount** signal will remain at a low level during at least 112ns before triggering the reset
 
The **Mismatch** signal(output of the comparator 74LS688) is generating a clock signal for the loop counting 
circuit(2 JK flip flop).

**Output of the comparator 74LS688 & AND gate**
  NOT CLK | P=Q! | Mismatch value
  --------|------|--------------
  0       |  0   |   0    ==> CLK = 0V & Byte OK 
  0       |  1   |   0    ==> CLK = 0V & Byte MISMATCH (happen during the PROGRAMMING sequence )
  1       |  0   |   0    ==> CLK = 5V & Byte OK (VERIFY MODE OK)
  1       |  1   |   1    ==> CLK = 5V & Byte MISMATCH (VERIFY MODE NOT OK) loop 3 times

74LS688 Output is high (+5v) when P!=Q
P=Q output is (0V)
If the Byte mismatch the output of 74LS688 will remains high and the led will be lit. 
This scenario can also happen during the PROGRAMMING MODE, this is why
we need to add an AND gate 74LS08 at the output of the comparator 
to lit the led only during the VERIFY MODE when the byte mismatch.
