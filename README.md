# PLL_ICDesign

This workshop conducted by VSD.

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

### Introduction to Phase Frequency Detector 

This blocks identifies the difference in phase of the reference and the output signal.

![image](https://user-images.githubusercontent.com/86144443/127751334-881b399e-e059-4451-a7eb-ecd48e37e4ff.png)

When falling edge of OUT signal is detected, DOWN stays active till the falling edge of REF signal.
When the falling edge of REF is detected, UP stays high till falling edge of OUT signal.

Now this functionality is applied to output signal with different frequencies.

![image](https://user-images.githubusercontent.com/86144443/127751509-02503a92-57b0-4368-b764-6e2827194003.png)

This functionality also captures the frequency difference between the signals. When the OUT frequency is higher, DOWN signal gets activated which says to slow down the output. When the OUT frequency is lower, the UP signal gets activated which suggests to speed up the input.

This functionality can be represented in FSM format:

![image](https://user-images.githubusercontent.com/86144443/127751534-18329504-f3d0-4c71-8ac5-890b13469f39.png)

The falling and rising edges can be identified using flip flops. We will need two flip flops to detect the falling edges of two signals REF and OUT. When both the falling edges are detected, the ouput of the AND gate goes high which clears the output of two flip flops.

![image](https://user-images.githubusercontent.com/86144443/127751606-ae835816-23af-4e85-aa91-4cd4bae3e172.png)

There is one small issue in the circuit which is the DEAD ZONE. When the difference in phase is too small, the output is clipped as the capacitor does the gets enough time to charge. In this dead zone, the PFD is insensitive to phase difference.

![image](https://user-images.githubusercontent.com/86144443/127751669-35936f40-0e0c-4233-a0e1-2f107e912a83.png)

### Introduction to Charge Pump




![image](https://user-images.githubusercontent.com/86144443/127749514-22d2d557-cfb5-4dc7-8601-ef1f77a7f007.png)

![image](https://user-images.githubusercontent.com/86144443/127749572-ed6216af-bbe7-4b07-b991-8f8e1af1ba94.png)

We can see that the current leakage is very small in the charge pump. The slope of the signal is very small ( 40 * 10^-6)
Now let us give an actual pulse signal and see the response of the charge pump.

![image](https://user-images.githubusercontent.com/86144443/127749678-69180ad6-e231-470e-93b4-2fdd6a6651a9.png)

The UP and DOWN in the signal is charging and discharging of the capacitor.
The output of the VCO:

![image](https://user-images.githubusercontent.com/86144443/127749761-fa1c51dd-405e-46b8-878f-a0063757844f.png)

![image](https://user-images.githubusercontent.com/86144443/127750034-e1043661-11db-46b9-a0e2-8869058a7923.png)

Now we will combine all the files to make a PLL circuit.
