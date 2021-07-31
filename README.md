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
