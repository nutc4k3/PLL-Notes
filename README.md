# Ｐｈａｓｅ－Ｌｏｃｋｅｄ　Ｌｏｏｐ　（ＰＬＬ）　ＩＣ　Ｄｅｓｉｇｎ

> 🗒️ Notes on PLL IC Design for the [VLSI Hardware Design Program]( https://www.vlsisystemdesign.com/).

# *Contents*
------------
- [Introduction to Phase-Locked Loop](#introduction-to-phase-locked-loop)
  * [Phase Frequency Detector](#phase-frequency-detector)
  * [Filter](#filter)
  * [Frequency Divider](#frequency-divider)
  * [Voltage Controlled Oscillator](#voltage-controlled-oscillator)
- [Design Flow](#design-flow)
- [Tools and supporting files](#tools-and-supporting-files)
    + [Tools](#tools)
    + [Supporting Files](#supporting-files)
- [Circuit Design and Simulation](#circuit-design-and-simulation)
  * [Specifications](#specifications)
  * [Individual Components](#individual-components)
- [Control System](#control-system)
- [Acknowledgment](#acknowledgment)

## Introduction to Phase-Locked Loop

🔬 A phase-locked loop (PLL) is a control system that generates output signal related to the input signal phase. This happens by synchronizing itself to the input clock from comparing the phase difference between the input signal and the output signal of a voltage-controlled oscillator (VCO). 

🖍️ A PLL can be implemented in many ways and for many different goals or means, ranging from simple home-made applications to complex modern systems. You can configure PLLs as frequency multipliers, dividers, demodulators, tracking generators, or clock recovery circuits. You can use PLLs to generate stable frequencies, recover signals from a noisy communication channel, or distribute clock signals throughout your design. With that in mind, it would be very hard to dig in its specifics in just some notes, so the content presented here is just the scratch surface of this profound topic.

⚙️ The main blocks of a phase-locked loop consists of a Phase Frequency Detector (PFD), a Filter (Loop Filter/Charge Pump), a Voltage Controlled Oscillator (VCO) and a Frequency Divider (FD) or counter. The purpose of working with PLLs is to provide minimal phase and frequency noise, flexibility to control the signal frequency and adaptability, since it can be implemented directly on SoC.

![image](https://user-images.githubusercontent.com/78468534/127774506-b254b925-d629-4f40-8440-e0f332b1e57c.jpeg)

As shown in the block diagram above, the PLL has two routines:
1. *Compare the frequency of reference and ouput (PFD)*
2. *Adjust the frequency to match reference signal (CP and VCO)*


### Phase Frequency Detector
> The PFD compares the outputs of the N and R counters in order to generate a correction voltage, which is converted to a current by the charge pump.

As its name suggests, a **Phase Frequency Detector** responds both to phase and frequency differences, checks if the output is leading or lagging behind the reference, and outputs the digital translation of this comparison. One approach is to use XOR gates and flip-flops to measure signal difference and falling/rising edge signals behavior.

### Filter
> The CP converts the digital output from PFD to an analog signal to control the VCO. A low pass filter can be added before the VCO to smoothen the signal and stabilize the feedback loop.

To control the oscillator, the filter analog signal is passed through the low pass filter before connecting to the VCO. This prevents a known problem in filters that happens when no UP/Down signal is identified, resulting in a leakage current flowing to the output and charging the capacitor, therefore affecting the control voltage. By using this *current steering circuit*, the VCO receives a much more stable and clean signal, playing a major role in the stability of the system.

### Frequency Divider
> The frequency divider circuit is used to multiply the output frequency.

A PLL with a frequency divider on its feedback loop is called a clock multiplier PLL meaning it can make clock signals which are multiples of the reference signals. One way to implement this is using a toggle flip-flop. In toggle flip-flops, the output is half the frequency of the input. This can be designed using a D-Flipflop with an inverter feeding the output back to the input, resulting in a frequency division by 2. Hence, if the a signal with 10 ns period is supplied at the input, the ouput would be a signal of 20 ns period. In the same way, for obtaining a 8x multiplier 3 such toggling flip flop would be used and so on.

### Voltage Controlled Oscillator
> Voltage controlled oscillators are the actual parts which produces alternating digital clock signal, their frequency can be controlled by the input voltage. 

One way of implemententing VCOs is using simple odd number inverters in series and with the same delay (current starving ring oscillator). This causes the output signal to have a fixed frequency. By making this output flip after a certain delay, driving half of a period which is given twice the delay of the inverter and the inverter count `p = 2*d(i)*i`. To have control over the frequency, the current starving mechanism is used over the ring oscillator circuit, where two supply of currect sources are provided at the top and bottom of the ring oscillator.

## Design Flow

The design flow of the PLL IC could take the following steps:

1. **SPICE level circuit**
> Program used in integrated circuit and board-level design to check the integrity of circuit designs and to predict circuit behavior.
2. **Pre-layout simulation**
> Pre-layout simulations take place before completing the PCB layout. For pre-layout simulation, we must build up a circuit schematic to include all elements of the simulation. For signal-integrity purposes, this includes IC buffer models, package models, trace models, vias, discrete components, and any connectors and cables. In terms of power integrity, this includes plane shapes, stitching vias, capacitors, power sources, and loads (ICs).

4. **Layout development**
> Layout combines a huge number of circuits into a larger integrated circuit. This design methodology starts with building fundamental circuit blocks and integrating them into a larger system. 
5. **Parastic Extraction**
> Parasitic extraction is the process of extracting the capacitance effects(parasitics) of the circuit realised in the layout. This can be done using the magic tool. The magic tool can do this using extract all command. This can further be converted to a spice file using command ext2spice. 
The parasitic extraction and post-layout simulation are steps for ensuring fuctionality. The GDS file generation can be created with magic tool.
For extracting the parasitic capacitance and resistance after layout design some useful commands:
``` 
 extract all
 ext2spice cthresh 0 rthresh 0
 extracts the parasitics from the 0th value itself.
 ext2spice
 write gds
 ```
> After extraction of the spice file, all the parasitics are included in it along with other parameters (ad,as,pd,ps) and some changes occurs in the spice program. The first thing which changes is the scale for the transistor sizing and hence it should be changed to the required scale. Now the parasitic extraction of the PFD file is done and the spice file is checked and it is found that a total of 43 parasitics capacitance is extracted nad the highest capacitive value is of capacitor C35 = 3.76fF, across the VDD and GND pin which is observe as a part of lab exercise. The snapshot of the same is included below.

6. **Post-layout simulation**
> Post-layout simulations use the completed PCB layout as their basis. It involves extraction of physical information from the routed board. Items like traces, planes, and vias with defined geometries are automatically modeled, as are simpler components such as discrete devices. We add models for the ICs, connectors, and other connected components to run simulations. In each case, these user inputs are translated into a simulation schematic used by the simulator. Moreover, both types of simulations have the same simulation results. For signal-integrity simulations, the results appear as time/voltage waveforms that identify signal quality and timing information, and they can be displayed as waveforms or tables of data
7. **Tapeout deploy**
> GDS file can be created using the option File > Write GDS. Although the GDSII file is considered the final format which is sent for fabrication, the IC designed cannot be send as such. The design need to be "prepared" for fabrication process. This preparation is called the tapeout. Any addition which helps connect the wafer to outside world would come under this preparation. This may include adding I-O ports, UART and other peripherals. We will be using Efabless free "shuttle", Caravel SoC.
Within the design for caravel, the IP can be added using Place Instance option within the magic tool. After placing, the inputs and outputs need to be connected to the carvel template.

## Tools and supporting files

#### Tools

- [ngspice](https://sourceforge.net/projects/ngspice/) -> **Open-source electronic circuit simulator widely used in ciruit design and analog VLSI area** 
```
ngspice <spice_file_name>
```
- [magic](http://opencircuitdesign.com/magic/) -> **Open-source VLSI layout tool.**
```
magic -T <technology_file_name> <layout_file_name>
```

#### Supporting Files
- [Resources and scripts](https://github.com/lakshmi-sathi/avsdpll_1v8/)  -> **Ready to use 130nm PLL Clock Multiplier IP and base files.**
- [Sky130 PDK](https://github.com/google/skywater-pdk) -> **Google-Skywater SKY130 is a set of 180nm-130nm technology based process nodes and Process Design Kits provided for free.**

---------

## Circuit Design and Simulation
### Specifications
> 8x PLL Clock Multipler IP on Google-skywater 130nm node

| Parameter | Description | min | typ | max | Unit | Conditions |
| --- | --- | --- | --- | --- | --- | --- |
| VDD | Digital Supply | - | 1.8 | - | V | T = 27C |
| F<sub>CLKREF</sub> | Reference | 5 | - | 12.5 | MHz | T = 27C |
| F<sub>CLKOUT</sub> | Output Clock | 40 | - | 100 | MHz | PLL Mode, T = 27C |
| F<sub>CLKOUT</sub> | Output Clock | - | - | - | MHz | VCO Mode, T = 27C |
| J<sub>RMS</sub> | Jitter (rms) | - | - | 20 | ns | PLL Mode |
| DC | Duty Cycle | 52.7 | - | 50 | % | T = 27C | 
| T<sub>SET</sub> | Settling Time | ~37 | - | ~22 | ns | T = 27C |
| C<sub>L</sub> | Load Capacitance | - | - | - | fF | T = 27C |
| IDD | Supply Current | - | - | - | fF | T = 27C |

### Individual Components

| Stage | PFD | CP | VCO | FD |
| --- | --- | --- | --- | --- |
| SPICE level circuit | <img src="https://user-images.githubusercontent.com/78468534/127782010-b21f76ed-6bed-4406-bfd0-2c5fca9838ac.jpeg" width="40%"> | <img src="https://user-images.githubusercontent.com/78468534/127782045-6a5b2df2-13fd-4456-9337-6b2af6604d05.jpeg" width="40%"> | <img src="https://user-images.githubusercontent.com/78468534/127782614-ed6b8289-cf29-4cd9-bfe7-078176fe6c26.jpeg" width="40%">  | <img src="https://user-images.githubusercontent.com/78468534/127781480-b09756aa-a4ce-48e4-8164-4fad67cf1f7d.jpeg" width="40%">  |
| Pre-layout simulation | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Prelayout_sim/pd_prelayout_sim.png" width="100%"> | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Prelayout_sim/cp_leak_prelayout_sim.png" width="60%"> <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Prelayout_sim/cp_up_prelayout_sim.png" width="60%">| <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Prelayout_sim/vco_prelayout_sim.png" width="100%"> | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Prelayout_sim/fd_prelayout_sim.png" width="100%"> |
| Layout development | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Layout/pfd_layout.png" width="100%"> | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Layout/cp_layout.png" width="100%"> | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Layout/vco_layout.png" width="100%"> | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Layout/fd_layout.png" width="100%"> |
| Parasitic Extraction | <img src="https://github.com/nutc4k3/PLL-Notes/blob/bd0126b0f799eaf40b8484877a2cea9a30a73b2b/images/Layout/parasitic_extraction_pfd.png" width="100%"> | - | - | - |
| Post-layout simulation | <img src="https://github.com/nutc4k3/PLL-Notes/blob/bd0126b0f799eaf40b8484877a2cea9a30a73b2b/images/Postlayout_sim/pfd_postlayout_sim.png" width="40%"> <img src="https://github.com/nutc4k3/PLL-Notes/blob/bd0126b0f799eaf40b8484877a2cea9a30a73b2b/images/Postlayout_sim/pfd2_postlayout_sim.png" width="40%"> <img src="https://github.com/nutc4k3/PLL-Notes/blob/bd0126b0f799eaf40b8484877a2cea9a30a73b2b/images/Postlayout_sim/pfd3_postlayout_sim.png" width="10%"> | - | - | - |

## Control System
| Stage | PLL |
| --- | --- |
| Pre-layout simulation | <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Prelayout_sim/pll_prelayout_sim.png" width="40%"> <img src="https://github.com/nutc4k3/PLL-Notes/blob/2f0dcc57996da59c5646d0e96a157ce1534c42d8/images/Prelayout_sim/pll2_prelayout_sim.png" width="40%">|
| Layout development | <img src="https://github.com/nutc4k3/PLL-Notes/blob/bd0126b0f799eaf40b8484877a2cea9a30a73b2b/images/Layout/pll_layout.png" width="60%"> |
| Parasitic Extraction | - |
| Post-layout simulation | - |
| Tapeout | - |

## Acknowledgment

- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
- [Lakshmi Sathi,8x PLL Clock Multipler IP on Google-skywater 130nm node](https://github.com/lakshmi-sathi/avsdpll_1v8).
