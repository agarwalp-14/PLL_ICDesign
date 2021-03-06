# Phase Locked Loop (PLL) Design using SKY130nm Technology

This workshop was conducted by VSD from 31st July 2021 to 1st August 2021.

![image](https://user-images.githubusercontent.com/86144443/127809920-98825f4e-8b41-4286-9493-2296e2fb412f.png)


## Table of Content

- [Day1: PLL Theory and Lab Setup](#day1--pll-theory-and-lab-setup)
  * [Introduction to Phase Locked Loop(PLL)](#introduction-to-phase-locked-loop-pll-)
  * [Introduction to Phase Frequency Detector](#introduction-to-phase-frequency-detector)
  * [Introduction to Charge Pump](#introduction-to-charge-pump)
  * [Introduction to VCO and frequency divider](#introduction-to-vco-and-frequency-divider)
  * [Important terms in PLL](#important-terms-in-pll)
    + [Lock Range](#lock-range)
    + [Capture Range](#capture-range)
    + [Settling time](#settling-time)
  * [Tool Setup and Design flow](#tool-setup-and-design-flow)
    + [Tools setup](#tools-setup)
    + [Development flow](#development-flow)
  * [Introduction to PDK, specifications](#introduction-to-pdk--specifications)
  * [Pre-Layout Circuits](#pre-layout-circuits)
- [Day 2](#day-2)
  * [Pre-Layout Simulations](#pre-layout-simulations)
  * [Layouts](#layouts-)
  * [Parasitics Extraction](#parasitics-extraction)
  * [Post Layout Simulations](#post-layout-simulations)
  * [Tape-Out](#tape-out)
- [Acknowledgements](#acknowledgements)

## Day1: PLL Theory and Lab Setup

### Introduction to Phase Locked Loop(PLL)

PLL is used to get an accurate clock signal without frequency or phase noise and also to get the frequency of our choice.

The two ways by which we can generate the clock signal is:
- Quartz crystal: Superior spectral purity
- Voltage Controlled Oscillator (VCO): It can be implemented on-chip using inverters and have good flexibility but they tend to have fluctuations in their phase.

The entire purpose of the PLL is to design it with the superior spectral purity like that of the Quartz crystal but still maintaining the flexibility Like the VCO.
The intuition of the PLL is to have the same/multiple of the reference frequency and a constant phase difference with it.

![image](https://user-images.githubusercontent.com/86144443/127729816-7de4a2ef-254b-49e4-904b-5961fa562a8f.png)

The control system block tracks the ouput frequency and synchronizes it with respect to the reference signal.

![IMG-0441](https://user-images.githubusercontent.com/86144443/127730122-f2550cea-5e6e-40cb-8dff-89857e8a0dca.jpg)

The VCO is the on chip oscillator. The PFD takes care of the comparison of the output signal with the reference signal. Charge pump converts the digital comparison that comes from the outputt of the PFD to an analog signal. LPF smoothens the output.

Few applications of the PLL are:

a. Clock generation

b. Frequency synthesizer

c. Clock recovery in a serial data link

### Introduction to Phase Frequency Detector 

This blocks identifies the difference in phase of the reference and the output signal.

![image](https://user-images.githubusercontent.com/86144443/127783895-4b166b76-e9c8-47ea-8baa-7fc51ae06e52.png)


When falling edge of OUT signal is detected, DOWN stays active till the falling edge of REF signal.
When the falling edge of REF is detected, UP stays high till falling edge of OUT signal.

Now this functionality is applied to output signal with different frequencies.


This functionality also captures the frequency difference between the signals. When the OUT frequency is higher, DOWN signal gets activated which says to slow down the output. When the OUT frequency is lower, the UP signal gets activated which suggests to speed up the input.

This functionality can be represented in FSM format:

![image](https://user-images.githubusercontent.com/86144443/127751534-18329504-f3d0-4c71-8ac5-890b13469f39.png)

The falling and rising edges can be identified using flip flops. We will need two flip flops to detect the falling edges of two signals REF and OUT. When both the falling edges are detected, the ouput of the AND gate goes high which clears the output of two flip flops.

![image](https://user-images.githubusercontent.com/86144443/127751606-ae835816-23af-4e85-aa91-4cd4bae3e172.png)

There is one small issue in the circuit which is the DEAD ZONE. When the difference in phase is too small, the output is clipped as the capacitor does the gets enough time to charge. In this dead zone, the PFD is insensitive to phase difference.

![image](https://user-images.githubusercontent.com/86144443/127751669-35936f40-0e0c-4233-a0e1-2f107e912a83.png)

### Introduction to Charge Pump

The charge pump converts the digital measure of the frequency/phase difference into an analog control signal to control the oscillator. This can done using the current steering circuit.

![image](https://user-images.githubusercontent.com/86144443/127783949-e0cf3926-2f63-4523-a11c-0922b0103a1e.png)


When UP is active the capacitor gets charged, this increases the voltage at charge pump output. When Down is active, the capacitor gets discharged through ground.

![image](https://user-images.githubusercontent.com/86144443/127751898-975aaa0e-6b22-4216-87ea-2e1ad6d4b499.png)

![image](https://user-images.githubusercontent.com/86144443/127751917-225cd908-41e8-4782-845b-153f06d39dea.png)

This output voltage controls the VCO. An increase in voltage speeds up the oscillator, while a reduction in voltage slows down the oscillator.

The circuit for charge pump: 

![image](https://user-images.githubusercontent.com/86144443/127752240-d13a4302-e06b-493f-be82-8abf06fdf692.png)

If we replace  the output capacitance with a low pass filter, the fluctuations in the output volatge with smoothen out and it will also stabilize the PLL circuit. Without this loop filter, the PLL will not work properly. 

![image](https://user-images.githubusercontent.com/86144443/127752027-86937051-492d-4207-bf42-37df39845afc.png)

For maintaining the stability of PLL, the following considerations must be followed:
- The value of Cx should be roughly around one tenth of C. 
- The loop filter bandwidth must be less than one tenth of the highest output frequency desired for the PLL.

The loop filter Bandwidth is 1/(1+RC1) where C1= (C * Cx)/ (C + Cx)

### Introduction to VCO and frequency divider

The VCO is a combination of odd number of inverters in series. The period of this oscillator is (2 * delay_of_inverter * inverter_count).
To control the output frequency we use a 'current-starving' mechanism. Two current sources are used as current supplies at the top and bottom of the ring oscillator.
Its necessary to design this VCO such that the range of output frequency we want for the PLL is within the range of frequency the VCO can produce properly.

![image](https://user-images.githubusercontent.com/86144443/127752773-53117b53-92dc-4b7d-99b4-af1a0b480ab9.png)

The basic toggle flipflop divides the frequency by 2. 

![image](https://user-images.githubusercontent.com/86144443/127752858-f8f400c7-8f36-4e3a-89ad-e17a2f9d8529.png)

Using three such toggle flip flops we can create a divide by 8 frequency divider.

### Important terms in PLL

#### Lock Range

It is the range of frequency for which the PLL is able to follow the input frequency. If the frequency of the input signal is falling beyond the PLL lock range then PLL will not be able to lock. Under this condition, VCO frequency jumps to its fundamental free running frequency.

#### Capture Range

The range of input frequencies for which the PLL will capture the input signal is called the capture range. As seen in the figure below, it is narrower compared to the lock range. Once input signal is captured, PLL will remain in locked state and will track the changes in the input signal till it remains within lock range.The loop filter bandwidth has an effect on the capture range.

![image](https://user-images.githubusercontent.com/86144443/127784744-76810842-7f2f-4a2b-b148-db93cf5bf263.png)

#### Settling time

The time within which the PLL is able to lock in from an unlocked condition.

### Tool Setup and Design flow

#### Tools setup
Two tools are used in this workshop:
- *ngspice* for transistor level circuit simulation. For ngspice, sky130 primitive library needs to be downloaded which contains the transistor level information which is required for eunning simulations. Command to run ngspice:
   *ngspice <circuit_file_name*
- *magic* for layout design and parasitic extraction. We need technology file for 130nm node. Command:
  *magic -T <technology_file_from_PDK> <the_layout_file_to_open>
  
  #### Development flow
  - SPICE-level circuit development
  - Pre-Layout Simulation
  - Layout Development
  - Parasitics Extraction
  - Post Layout Simulation
  
  ### Introduction to PDK, specifications
  
  The Process Design Kit(PDK) has different variations of the same gate and contains information like area, power and many other characteristics of the gates. This kit is provided by the fabrication center. We can incorporate all these in our design and then perform simulations.
  
  ![image](https://user-images.githubusercontent.com/86144443/127753559-f42a90c1-0623-4f5b-81d7-8a31500364e3.png)

The PDK content:
- io- input-output
- pr- primitives (spice)
- sc- standard cell
- hd- high density
- hs- high speed
- lp- low power
- hdll- high density low leakage

For the analog design, we will be building things from scratch, hence, we will be using the primitive sky130 library, *sky130_fd_pr*

PLL specifications:

- Corner - 'TT' (Here TT means typical typical. Both the NMOS and PMOS are nominal)
- Supply voltage - 1.8V
- Room Temperature
- VCO mode and PLL mode- The control voltage can be directly given to the VCO, the PLL IC will work as a VCO
- Input F_min= 5MHz; F_max= 12.5MHz
- Multiplier- 8x
- Jitter (RMS) < 20nsec (this is the phase noise specification) 
- Duty cycle- 50%

The approximate pin placement of the PLL IC:

![image](https://user-images.githubusercontent.com/86144443/127753775-f8fe5c45-d014-4d4b-8e4e-08e9fb1c9713.png)

### Pre-Layout Circuits

The frequency divider:

![image](https://user-images.githubusercontent.com/86144443/127753809-689adf50-4170-4766-a5f1-b0333b3dd899.png)

The Phase Frequency Detector (PFD):

![image](https://user-images.githubusercontent.com/86144443/127753836-1ea18c55-fcbc-48ee-9762-a7abfb7fbd57.png)

This Charge Pump: 

![IMG-0460](https://user-images.githubusercontent.com/86144443/144002263-56928000-afca-41e2-a317-a0bf57870f2e.jpg)


This cicuit tackles the issue of charge leakage when both UP and DOWN signals are off.

The VCO:

![image](https://user-images.githubusercontent.com/86144443/127753979-937300fe-d276-455d-85de-bb3968d28060.png)

 ## Day 2
 
 ### Pre-Layout Simulations
 Writing the sub-cicuits and simulating them individually. 
 
 Frequency Divider:
 
 ![image](https://user-images.githubusercontent.com/86144443/127749514-22d2d557-cfb5-4dc7-8601-ef1f77a7f007.png)
 
 The output frequency is half that of the input frequency.
 
 Charge Pump:

![image](https://user-images.githubusercontent.com/86144443/127749572-ed6216af-bbe7-4b07-b991-8f8e1af1ba94.png)

This is the output when no input is given.
We can see that the current leakage is very small in the charge pump. The slope of the signal is very small ( 40 * 10^-6)

Now let us give an actual pulse signal and see the response of the charge pump.

![image](https://user-images.githubusercontent.com/86144443/127749678-69180ad6-e231-470e-93b4-2fdd6a6651a9.png)

The UP and DOWN in the signal is charging and discharging of the capacitor.

The output of the VCO:

![image](https://user-images.githubusercontent.com/86144443/127749761-fa1c51dd-405e-46b8-878f-a0063757844f.png)

To get full swing at the ring oscillator VCO output an extra inverter is placed at the output, otherwise these oscillations will be of lower and varying amplitude.

Phase Frequency Detector: 

![image](https://user-images.githubusercontent.com/86144443/127750034-e1043661-11db-46b9-a0e2-8869058a7923.png)

The phase difference between two input signals is 6ns which is detected by the PFD circuit.

Now we will combine all the files to make a PLL circuit.


 ### Layouts
 
 Frequency Divider Layout:
 
 ![image](https://user-images.githubusercontent.com/86144443/127769545-64b9adcc-ede3-4180-b833-137ee55d7cf8.png)

PFD Layout:

![image](https://user-images.githubusercontent.com/86144443/127769690-9adf6752-e054-4a38-a291-4a75d841bfab.png)


VCO Layout:

![image](https://user-images.githubusercontent.com/86144443/127769727-33ac83dd-9361-46f6-8fd2-f2080d0e02eb.png)

Charge Pump Layout:

![image](https://user-images.githubusercontent.com/86144443/127769733-7a07aaae-7ab9-4f09-93a7-c3aed896f13f.png)

MUX Layout:

![image](https://user-images.githubusercontent.com/86144443/127769766-e30ab7af-dc7f-456a-81fd-f29bb0a8128c.png)

Integrated PLL:

![image](https://user-images.githubusercontent.com/86144443/127771231-1d48b10a-e8ba-4e2a-85f7-fc6ae3db068d.png)


### Parasitics Extraction

The wiring interconnects in the layouts creates parasitic capacitances and resistances. The parasitic extraction is the calculation of these parasitics in the designed devices so that an accurate analog model of the circuit is created. With shrinking nodes, these parasitics hve started making a significant impact on the circuit performance, so it becomes important to incorporate them in our design. 

![image](https://user-images.githubusercontent.com/86144443/127772225-a44daaae-626a-4a19-a8b6-509f7ab8051f.png)

  These simple commands are used to extract the parasitics from the layout. We extract the parasitics to a spice file using command *ext2spice*
  Here we mention rthresh and cthresh ass 0 because we want to extract all the resistive and capacitive parasitics that are greater than 0.
  
  ![image](https://user-images.githubusercontent.com/86144443/127772419-ccd716dc-5828-491b-a815-b75be4ede581.png)

  On viewing PFD.spice, we can notice that there are additional capacitances extracted by the capacitor. Total 46 additional capacitances are there for PFD layout.
  
  On extracting parasitics for FD.mag, we observe a total 43 parasitic capacitance with the largest capacitance being C35 with the value of 3.76fF between Vdd and GND.
  
  ![image](https://user-images.githubusercontent.com/86144443/127772577-c593d167-b1a1-4e00-a04c-faa81470ec9e.png)

### Post Layout Simulations

Phase Frequency Detector:

![image](https://user-images.githubusercontent.com/86144443/127776113-e32df8e4-a33c-41f4-bd12-b87a0a60e938.png)


![image](https://user-images.githubusercontent.com/86144443/127776068-08d9d94c-d7b1-4650-b87e-fd1a56934aa1.png)

![image](https://user-images.githubusercontent.com/86144443/127776521-f9d99ff3-dfb8-4e34-8870-f8689cec3229.png)

Red: Clock 1

Blue: Clock 2

Orange: Up Signal

Green: Down Signal

Charge Pump:

Response to *Up* signal:

![image](https://user-images.githubusercontent.com/86144443/127780380-e63bdd35-f13c-40ab-b944-217a46c64a88.png)

Orange: Charge Pump Output Voltage

Red: Up Signal

Blue: Down Signal

Response to *Down* signal:

![image](https://user-images.githubusercontent.com/86144443/127780429-218aafbe-c1bf-4790-9f72-c4a6983fe367.png)

Orange: Charge Pump Output Voltage

Red: Up Signal

Blue: Down Signal
  
Response due to charge leakage: 

![image](https://user-images.githubusercontent.com/86144443/127780584-872f2d45-5a1f-473b-b064-dfb6bd7c6ecf.png)

Orange: Charge Pump Output Voltage
Red: Up Signal
Blue: Down Signal
Leakage: < 0.05V in 100us

We combine all the instances of the layout on magic and extract parasitics and perform simulations for one last time. On getting the desired result we write the entire layout to a GDS(Graphic Data System) file. GDS file contains the information like geometric shapes and text labels about the layout in binary format.

### Tape-Out

Tape-out is the process of sending the final designs to foundry for undergoing the fabrication process. The Silicon wafer needs to be connected to the external world. For this we use a SoC which has other IPs which will help in making our design user-ready. 

![image](https://user-images.githubusercontent.com/86144443/127784513-0dd83271-4608-4792-80f9-c4119cad6b02.png)

This is the Efabless Caravel Soc template. We can add our design in the user area. All other needs like serial connectivity, memory and testing mechanisms will be taken care by the SoC.

## Acknowledgements
 - Kunal Ghosh (co-founder VSD Corp. Pvt Ltd) 
 - Lakshmi S
 - https://github.com/lakshmi-sathi/avsdpll_1v8

