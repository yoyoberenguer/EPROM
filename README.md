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

### DC to DC converter PCB (THT)

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

> **Additional Notes**
>
>- The **SN74LS85** (16-pin magnitude comparator) does **not** include an enable pin. It continuously compares both input values in real time.
>- The **SN74LS08** (AND gate) and **SN74LS04** (inverter) can optionally be replaced with a single **NAND gate** for logic simplification, depending on the logic level inversion required.
>- **Pin 11 (LOAD)** on the SN74LS85 is **unused** in this design and should be connected to **V<sub>CC</sub>** to ensure proper logic level.
>- The **ABCD parallel data inputs** of the SN74LS85 are also **not used** and must be tied to **ground** to avoid floating inputs and potential erratic behavior.


## Frequence comparator KICAD

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/Frequency_comparator_version1.0.PNG?raw=true)

## Frequence comparator PCB (THT)
![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/EPROM_FrequencyCompare.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20Comparator/EPROM_FrequencyCompare_components.png?raw=true)

---

### Frequency Generator module

The frequency generator provides the **main clock signal** required by the **17-bit counter** (A0–A16) to sequentially address every possible location on the EPROM’s address bus.
As discussed previously, the theoretical maximum programming frequency for a **27C256 EPROM** is approximately **10 kHz** (determined by pin 20 timing requirements).  
To ensure proper synchronization with the **pulse generator**, the main clock frequency will be set to **twice** the value of the pulse generator frequency. The rationale for this
will be explained in the next stage: **Frequency Multiplexer**.

**Clock Source Flexibility**

To assist in debugging and testing, the prototype includes **multiple customizable frequency sources**. The board is designed to support:

- A **Schmitt trigger oscillator** with an **adjustable resistor** for wide frequency tuning.  
  The frequency is determined by the **time constant**: `(R1 × C1)`
  
- A **32.768 kHz quartz crystal oscillator** built using a **CD40106BE** Schmitt trigger inverter.  
  If a Schmitt trigger is not used, you must:
  - Add **two 47 kΩ resistors** to bias the input between **V<sub>CC</sub>** and **GND**
  - Add an **extra inverter stage** to reshape the output waveform for a clean digital signal

These options provide a flexible and stable timing base for both normal operation and troubleshooting scenarios.

### Frequency generator KICAD 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20generator/Frequency_generator.PNG?raw=true)

And finally a step by step clock generator to troobleshoot on demand, a pulse is generated each time the contact **SW1** is pressed 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Frequency%20generator/Manual_clock_pulse.PNG?raw=true)

---

## Binary Counter module

The diagram below represents the **17-bit binary counter** (A0–A16) constructed from multiple integrated circuits:

- **SN74F163N**: A 4-bit synchronous binary counter with flip-flops triggered on the **rising (positive-going) edge** of the clock (CLK).  
  The **clear (CLR)** function is synchronous: applying a **low logic level** to CLR sets all four flip-flop outputs to low (0000) after
  the next rising clock edge, regardless of the enable inputs (ENP and ENT).  

This synchronous clear feature allows easy modification of the count length by decoding the Q outputs for the desired maximum count.
An active-low gate output from this decoder is fed back to the CLR input to synchronously reset the counter to zero (0000).

**SN74F163N pinout**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/SN74f163_pinout.PNG?raw=true)

- **DM74LS112A**: Dual Negative-Edge-Triggered Master-Slave J-K Flip-Flop with Preset, Clear, and Complementary Outputs.  
  This device contains two independent negative-edge-triggered J-K flip-flops with complementary outputs.

- **HEF4060B**: A 14-stage ripple-carry binary counter/divider and oscillator featuring three oscillator terminals (RS, REXT, CEXT), ten buffered outputs (**Q3–Q9** and **Q11–Q13**), and an asynchronous master reset input (MR).  
  Note that outputs **Q0, Q1, Q2, and Q10** are **not** available internally and must be implemented separately in our design.  
  The internal oscillator (RS, REXT, CEXT) will be bypassed in favor of an **external oscillator** to clock the HEF4060B, along with the remaining flip-flops and the 4-bit counter (SN74F163N).

> **Important Notes**
>
>- The **SN74F163N** flip-flops trigger on the **rising edge** of the clock, whereas both the **DM74LS112A** and **HEF4060B** are triggered on the **falling (negative) edge**.  
>  This timing difference is critical to prevent desynchronization of the address bus bits **Q0–Q2, Q10, Q14, and Q15**.
>
>- The extra JK flip-flop outputs for **Q10, Q14, and Q15** are implemented asynchronously, in contrast to the synchronous operation of the HEF4060B and SN74F163N.  
>  This minor asynchronous mode difference does **not** affect the accuracy of the binary counting or the address bus frequencies.
>
>- **Q15** goes high when the count reaches the last address **0x7FFF** (corresponding to the 256K EPROM boundary).  
>  The **Q16** bit is reserved for addressing the 512K EPROM in **version 2.0** of the design.
 
 
**HEF4060 pinout**
  
![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/HEF4060B_pinout.PNG?raw=true)

**HEF4060 functional diagram** 

> Note that the outputs Q0 - Q2, Q10 are missing from the functional diagram

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/HEF4060_Functional_diagram.PNG?raw=true)

## Binary Counter KICAD 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/Kicad_CounterSchematic.PNG?raw=true)

## Binary Counter PCB (THT)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/counter.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Counter/counter_composants.png?raw=true)

---

## Display Multiplexer – 7-Segment Module

This module handles **7-segment display multiplexing** to visualize the memory content copied to the target EPROM. It presents **16 bits (2 bytes)** of data from the address lines (A0–A14) in hexadecimal format using two 7-segment digits.

**Functionality**

Each byte of memory on the data bus (D0–D7) is split into two **nibbles**:

- **Lower nibble (D0–D3)**: Represents the least significant 4 bits of the byte.
- **Upper nibble (D4–D7)**: Represents the most significant 4 bits.

For example, if the source EPROM at address `$0000` contains the value `$0A` (binary `00001010`):

- D0–D3 = `1010` → hexadecimal digit **A**
- D4–D7 = `0000` → hexadecimal digit **0**

**Components**

- **74LS157 (Quad 2-to-1 Multiplexer)**  
  This chip splits the data byte (D0–D7) into two separate nibbles. It selects either the LSB (D0–D3) or the MSB (D4–D7) for output via pins **Za, Zb, Zc, Zd**.
  
- **EPROM 27C256**  
  Acts as a **BCD to 7-segment decoder**. The nibble from the 74LS157 is sent to the EPROM via address inputs **A0–A3**, which select the corresponding segment pattern stored in the EPROM's memory.  
  The higher address lines (**A4–A14**) are forced to logic low (`0`) to target the lower address range of the EPROM.

**Memory Mapping for Display**

- EPROM addresses **$0000 to $0010** store the 7-segment display codes for hexadecimal digits `0–F`, assuming a **common cathode** display.
- Optionally, addresses **$0010 to $0020** can store codes for a **common anode** version.

A **toggle switch** can be connected to address line **A4** to switch between display types:

- A4 = 0 → Common Cathode
- A4 = 1 → Common Anode

This allows selecting the display type at runtime by applying **GND or +5V** to A4.

> **Note**
>
>The EPROM must be pre-programmed with appropriate 7-segment patterns corresponding to each hexadecimal digit.
>  Each entry should also account for optional **decimal point** activation, depending on your segment wiring.


**Example**

If address `$0000` contains the code for digit **0**, and address `$000A` contains the code for digit **A**, the two-digit 7-segment display will show "A0" for the byte `$0A`.

> 📌 Ensure that the 7-segment display type (common anode or cathode) matches the logic stored in the EPROM. Mismatched logic may invert segments and display incorrect values.

**comon cathode representing the address system (see table below)**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/BCD%207-seg%20decoder.PNG?raw=true)

**comon anode representing the address system (see table below)** 

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/BCD%207-seg%20decoder_comon_anode.PNG?raw=true)

## 7-Segment Decoding Table – Common Cathode vs Common Anode

The table below represents the **address lines A0–A4** and their corresponding **7-segment display encodings** stored in the EPROM. The 7-segment codes are used to visually display hexadecimal values (`0`–`F`) on a display module.

- **Addresses $0000 to $000F** contain the segment codes for **common cathode** displays.
- **Addresses $0010 to $001F** contain the segment codes for **common anode** displays.

A toggle switch connected to **address line A4** can be used to select between the two types of display:
- A4 = `0` → Common Cathode
- A4 = `1` → Common Anode

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

**Common Cathode**
| Addr (hex) | A4 | A3–A0 | Hex Digit | Segments (a–h) | Binary   | Hex Code |
|------------|----|--------|-----------|----------------|----------|----------|
| 0x00       | 0  | 0000   | 0         | abcdef         | 11111100 | 0xF6     |
| 0x01       | 0  | 0001   | 1         | bc             | 01100000 | 0x60     |
| 0x02       | 0  | 0010   | 2         | abdeg          | 11011010 | 0xDA     |
| 0x03       | 0  | 0011   | 3         | abcdg          | 11110010 | 0xF2     |
| 0x04       | 0  | 0100   | 4         | bcfg           | 01100110 | 0x66     |
| 0x05       | 0  | 0101   | 5         | acdfg          | 10110110 | 0xB6     |
| 0x06       | 0  | 0110   | 6         | acdefg         | 10111110 | 0xBE     |
| 0x07       | 0  | 0111   | 7         | abc            | 11100000 | 0xE0     |
| 0x08       | 0  | 1000   | 8         | abcdefg        | 11111110 | 0xFE     |
| 0x09       | 0  | 1001   | 9         | abcdfg         | 11110110 | 0xF6     |
| 0x0A       | 0  | 1010   | A         | abcefg         | 11101110 | 0xEE     |
| 0x0B       | 0  | 1011   | B         | cdefg          | 00111110 | 0x3E     |
| 0x0C       | 0  | 1100   | C         | adef           | 10011100 | 0x9C     |
| 0x0D       | 0  | 1101   | D         | bcdeg          | 01111010 | 0x7A     |
| 0x0E       | 0  | 1110   | E         | adefg          | 10011110 | 0x9E     |
| 0x0F       | 0  | 1111   | F         | aefg           | 10001110 | 0x8E     |

**Common Anode**
| Addr (hex) | A4 | A3–A0 | Hex Digit | Segments (a–h) | Binary   | Hex Code |
|------------|----|--------|-----------|----------------|----------|----------|
| 0x10       | 1  | 0000   | 0         | abcdef         | 00000011 | 0x03     |
| 0x11       | 1  | 0001   | 1         | bc             | 10011111 | 0x9F     |
| 0x12       | 1  | 0010   | 2         | abdeg          | 00100101 | 0x25     |
| 0x13       | 1  | 0011   | 3         | abcdg          | 00001101 | 0x0D     |
| 0x14       | 1  | 0100   | 4         | bcfg           | 10011001 | 0x99     |
| 0x15       | 1  | 0101   | 5         | acdfg          | 01001001 | 0x49     |
| 0x16       | 1  | 0110   | 6         | acdefg         | 01000001 | 0x41     |
| 0x17       | 1  | 0111   | 7         | abc            | 00011111 | 0x1F     |
| 0x18       | 1  | 1000   | 8         | abcdefg        | 00000001 | 0x01     |
| 0x19       | 1  | 1001   | 9         | abcdfg         | 00001001 | 0x09     |
| 0x1A       | 1  | 1010   | A         | abcefg         | 00010001 | 0x11     |
| 0x1B       | 1  | 1011   | B         | cdefg          | 11000001 | 0xC1     |
| 0x1C       | 1  | 1100   | C         | adef           | 01100011 | 0x63     |
| 0x1D       | 1  | 1101   | D         | bcdeg          | 10000101 | 0x85     |
| 0x1E       | 1  | 1110   | E         | adefg          | 01100001 | 0x61     |
| 0x1F       | 1  | 1111   | F         | aefg           | 01110001 | 0x71     |


> 🔧 **Note**: The 7-segment codes are provided in hexadecimal and may vary depending on the display wiring (a–g segments + DP). Adjust accordingly if decimal points are used.

This design provides full compatibility with both **common cathode** and **common anode** displays by using a simple logic-level switch and preloaded EPROM codes.

**7-Segment Display Multiplexing**

Both 7-segment displays used in this design are common anode type. Each display is activated briefly by supplying a **V<sub>CC</sub>** +5V control signal to its corresponding transistor switch, 
enabling the current flow through the display.The transistors **Q2** and**Q4** are connected respectively to the **Q** and **$\overline{Q}$** outputs of a JK flip-flop **74LS112**.
This configuration ensures that the two displays are activated alternately, never simultaneously. As a result, each display lights up only during its assigned clock phase.

To avoid visible flickering, the clock frequency driving the multiplexing should be greater than **60 Hz**. This ensures that the displays are refreshed quickly enough for human perception, creating a smooth visual appearance.

The 74LS112 JK flip-flop is used to divide the main clock frequency by two, generating a toggling signal that alternately enables one display and then the other. 
This alternating activation is essential for multiplexed display control, reducing the number of required data lines while maintaining visual clarity.


### 7-Segment Module KICAD
![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/Multiplexing_diagram.PNG?raw=true)

### 7-Segment Module PCB (THT) 
![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/EPROMDisplay.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/Multiplexing/EPROMDisplay_components.png?raw=true)

---

### EPROM Flashing Module

In this stage, the **right-hand EPROM** serves as the **source**, containing the data to be copied. The **left-hand EPROM** is the **target**, where the data is written.

**Shared Buses**

- The **address bus (A0–A16)** is shared between both EPROMs and is driven by a **15-bit binary counter**, which increments to point to each successive memory address.
- The **data bus (D0–D7)** is also shared and connects both EPROMs to the **multiplexing stage**, allowing real-time display of the data being transferred.

**Operation Sequence**

During the **rising edge and first half** of the clock (CLK) cycle:
- The **source EPROM** is in **READ mode** and outputs data onto the **data bus (D0–D7)**.
- Simultaneously, the **target EPROM** is in **PROGRAM mode**, ready to receive this data for writing.

Refer to the mode selection table (not shown here) for specific control pin configurations required to switch between **READ**, **PROGRAM**, and **VERIFY** modes.

**Data Transfer and Register Latching**

The data from the **source EPROM** is latched into an **8-bit register** made of two **74LS173 ICs** when a **100 µs** active-low pulse **$\overline{Pulse}$** is applied to pin 7 (common clock input).

This same **$\overline{Pulse}$** signal also loads the data into the **Q0–Q7 inputs of the comparator** (e.g., 74LS688), preparing it for the subsequent verification phase.

The **74LS173 registers** retain the byte read from the source until the next write pulse. This byte is then compared to the output of the **target EPROM** during **VERIFY mode**, where both data values are expected to match.

**Verify Mode (Falling Edge)**

On the **falling edge** of the clock:
- The **target EPROM** switches to **VERIFY mode** and places its output onto the **data bus (D0–D7)**.
- The **source EPROM** enters **STANDBY mode**.

The value now on the data bus is compared against the latched byte using the **74LS688 comparator**:
- If the bytes **match**, the comparator output (**pin 19**) goes **LOW** (logical 0).
- If there is a **mismatch**, the output remains **HIGH** (+5V), indicating a failed programming attempt.


![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/27C256_operating_modes.PNG?raw=true)


**Source EPROM and Target EPROM (left) KICAD schematic**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/Flashing_diagram.PNG?raw=true)

---

## Data validation module KICAD**

![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/EPROM_ParityCheck_diagram.PNG?raw=true)


**Data Validation and Retry Logic**

The **data validation circuit** ensures that each byte written to the target EPROM matches the original byte from the source EPROM. 
It is implemented using:
- **Two 74LS173 ICs** (forming an 8-bit D-type register)
- **One 74LS688** 8-bit comparator
- **One NE555 timer** in bistable mode
- **Two 74LS112** JK flip-flops (for retry counting)
- Logic gates: **74LS08** (AND) and **74LS00** (NAND)

**Write and Verify Phases**

- During the **PROGRAM MODE** (first half of the clock signal, i.e., when **CLK is high**), a ~100 µs **pulse** is sent to:
  - **Target EPROM** → writes the byte from the data bus (D0–D7)
  - **74LS173 registers** → latches and stores the original byte

- During the **VERIFY MODE** (second half of the clock, i.e., when **CLK is low**):
  - The **target EPROM** outputs the recently written byte to the data bus
  - The **74LS688 comparator** compares this value (input Q) with the stored byte in the 74LS173 registers (input P)

If **P = Q**, the data is valid. If not, a **mismatch** has occurred.

**Error Detection and Retry Mechanism**

To handle possible write errors, the system includes a **retry mechanism with up to 3 attempts** before resuming the flash operation.

**NE555 Bistable Circuit**

- When a **mismatch is detected** (P ≠ Q), the comparator’s output $\overline{P=Q}$ goes **HIGH**
- This triggers a **+5 V pulse** to pin 2 (**threshold**) of the NE555
- The NE555 output (pin 3, `Q`) goes **HIGH**, asserting the **`CounterLock`** signal
- `CounterLock` halts the 15-bit address counter, freezing the increment on the address bus (**A0–A16**)

**Retry Counter Logic**

- The **retry counter** is implemented with **two 74LS112 JK flip-flops**, counting from **0 to 2**
- The **clock input** to the first JK flip-flop is triggered by an **AND gate (74LS08)** when:
  - $\overline{P=Q}$ is HIGH **and**
  - $\overline{CLK}$ is HIGH (second half of the cycle, i.e., **VERIFY MODE**)

- Each mismatch increments the retry counter
- When **retry count = 3**, both $\overline{Q0}$ and `Q1` outputs go LOW
- This triggers a **NAND gate (74LS00)**, generating a `loopCount` signal (LOW) that:
  - **Resets the NE555**, setting its output (Q) **LOW**
  - Deasserts the `CounterLock` signal
  - Resumes normal operation of the address counter

**Summary of Signals**

| Signal           | Source         | Description                                                 |
|------------------|----------------|-------------------------------------------------------------|
| $\overline{P=Q}$ | 74LS688        | HIGH when mismatch is detected                              |
| `CounterLock`    | NE555 (Q)      | HIGH to stop address counter; LOW to resume                 |
| `loopCount`      | 74LS00 NAND    | LOW when retry count = 3, used to reset NE555               |
| $\overline{CLK}$ | Clock Inverter | High during VERIFY MODE                                     |
| Retry Counter    | 2× 74LS112     | Counts retry attempts from 0 to 2                           |


Q0  | $\overline{Q0}$ |  Q1   | LoopCount $\overline{Q0}$ & Q1
----|---------|-------|--------------------------------
0   |    1    |   0   |   1
1   |    0    |   0   |   1 
0   |    1    |   1   |   0  ==> Third iteration LoopCount = 0V, the low level will trigger a reset.

**RC Reset Timing with NE555 and 74LS112**

The **360 Ω resistor** and **220 pF capacitor** connected to **pin 3 of the NE555** form a simple RC timing circuit.  
This creates a time delay that ensures the voltage at the NE555 output decays gradually.

This decay is essential to **reset both JK flip-flops** (74LS112) through their active-low reset pins ($\overline{R}$, pin 15)  
once the voltage across the capacitor drops below the **TTL low-level input threshold**:

- **V<sub>IL</sub>** = 0.8 (74LS112 low input threshold)
- **V<sub>OH</sub>** = 3.3 (typical NE555 high-level output)
- **R** = 360R
- **C** = 220pf


**⚠️ Why Not Direct NAND-to-Reset**

The `LoopCount` signal (output of the NAND gate) can **technically** be wired directly to the $\overline{R}$ inputs of the 74LS112 to reset the JK flip-flops.  
However, this approach is **not ideal** because:

- The `LoopCount` pulse duration is extremely short — typically **10–20 ns**.
- This brief pulse is **not recognized** by the NE555's reset pin (pin 4), which requires a longer **low-hold time**.

As a result, **the NE555 may ignore the reset event**, leading to unpredictable circuit behavior.

## RC Time Constant and Discharge Time Calculation

We want to calculate the time it takes for the voltage across a capacitor to drop from a high level \( V_{\text{OH}} \) to the logic low threshold \( V_{\text{IL}} \) of a 74LS112 JK flip-flop.

### Discharge Equation

The voltage across the capacitor over time is given by:

$$
V(t) = {V_{\text{OH}}} \cdot e^{-t / RC}
$$

Solving for \( t \) when \( V(t) = **V<sub>IL</sub>** \):

$$
\begin{aligned}
V_{\text{IL}} &= V_{\text{OH}} \cdot e^{-t / RC} \\\\
\frac{V_{\text{IL}}}{V_{\text{OH}}} &= e^{-t / RC} \\\\
-\frac{t}{RC} &= \ln\left(\frac{V_{\text{IL}}}{V_{\text{OH}}}\right) \\\\
t &= -RC \cdot \ln\left(\frac{0.8}{3.3}\right)
\end{aligned}
$$

### Substituting Values

- R = 360R  
- C = 220pF
- **V<sub>IL</sub>** = 0.8V  
- **V<sub>OH</sub>** = 3.3V

Compute the RC time constant:

$$
RC = 360 \cdot 220 \times 10^{-12} = 7.92 \times 10^{-8}\ \text{s}
$$

Now calculate the time:

$$
t = -7.92 \times 10^{-8} \cdot \ln\left(\frac{0.8}{3.3}\right) \approx 1.12 \times 10^{-7}\ \text{s}
$$

### Final Answer

$$
t \approx 112\ \text{ns}
$$

This delay ensures the reset input of the 74LS112 remains active for a sufficient duration, compensating for the short duration of the `LoopCount` pulse.


**Result**

- The RC network introduces a **112 ns delay** after the `LoopCount` signal goes LOW.
- This ensures the voltage drop across the capacitor is **slow enough to meet NE555 reset timing requirements**.
- The **reset signal will remain LOW** long enough to reliably trigger both **JK flip-flops**.

**Role of the Comparator**

- The `Mismatch` signal (output of the **74LS688 comparator**) acts as a **clock input** to the flip-flop retry counter.
- This ensures **loop counting occurs only on verified mismatches** between the source and target EPROMs.

**Summary**

| Signal                 | Purpose                                | Notes                                 |
|------------------------|----------------------------------------|----------------------------------------|
| `LoopCount`            | Triggers NE555 reset after 3 retries   | Extended via RC to meet hold timing   |
| `Mismatch`             | Clocks retry flip-flops                | Initiates retry logic on data error   |
| NE555 Pin 3            | Drives RC timing circuit               | Delay controls 74LS112 reset logic    |
| 74LS112 $\overline{R}$ | Flip-flop reset input (active LOW)     | Triggered via RC from NE555           |


**Output of the comparator 74LS688 & AND gate**
  $\overline{CLK}$ | P=Q! | Mismatch value
  ----------|------|--------------
  0         |  0   |   0    ==> CLK = 0V & Byte OK 
  0         |  1   |   0    ==> CLK = 0V & Byte MISMATCH (happen during the PROGRAMMING sequence )
  1         |  0   |   0    ==> CLK = 5V & Byte OK (VERIFY MODE OK)
  1         |  1   |   1    ==> CLK = 5V & Byte MISMATCH (VERIFY MODE NOT OK) loop 3 times

**Comparator Logic and LED Mismatch Indication (74LS688 + 74LS08)**

The **74LS688** comparator outputs a logic level based on the equality of its two 8-bit inputs:

- **Output = LOW (0 V)** when \( P = Q \)
- **Output = HIGH (+5 V)** when \( P \ne Q \)

In our application:

- **P** is the byte stored from the source EPROM (held in a 74LS173 register)
- **Q** is the byte read from the target EPROM during **VERIFY MODE**

**⚠️ Problem During PROGRAMMING MODE**

If a mismatch occurs during the **PROGRAMMING MODE**, the comparator output will be **HIGH**.  
However, at this stage, the data in the target EPROM is still being written and **should not** trigger a mismatch LED.

To prevent false positives during programming, we need to ensure the mismatch indication is **only active** during **VERIFY MODE**.

**Solution Using 74LS08 (AND Gate)**

We add a **74LS08 AND gate** after the output of the 74LS688 comparator.

**Inputs to the AND gate:**

1. Output from 74LS688 (HIGH when \( P \ne Q \))
2. **$\overline{CLK}$ signal** (HIGH during the VERIFY phase)

**Output of the AND gate:**

- **HIGH** (LED ON) **only if**:
  - There's a mismatch \( (P \ne Q) \)
  - The circuit is in **VERIFY MODE** ($\overline{CLK}$ = HIGH)

This ensures the mismatch LED is only lit when a valid verification mismatch is detected.

**Summary**

| Condition             | 74LS688 Output |  $\overline{CLK}$ (Verify Mode)  | LED (via 74LS08) |
|-----------------------|----------------|----------------------------------|------------------|
| P = Q (match)         | LOW (0 V)      | Any                              | OFF              |
| P ≠ Q during Program  | HIGH (+5 V)    | LOW                              | OFF              |
| P ≠ Q during Verify   | HIGH (+5 V)    | HIGH                             | **ON**           |

This design prevents false mismatch indications and ensures that the LED only lights when the byte validation truly fails during verification.

## Data validation PCB (THT)
![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/EPROM_ParityCheck.png?raw=true)

![image](https://github.com/yoyoberenguer/EPROM/blob/main/EPROM_flashing/EPROM_ParityCheck_components.png?raw=true)

