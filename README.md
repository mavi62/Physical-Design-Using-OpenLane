# Physical-Design-Using-OpenLane

This project is done in the course ["Advanced Physical Design using OpenLANE/Sky130"]

## Table of Contents

- [Day1-Introduction](#Day1-introduction)
- [Day2-Good Floorplan vs bad Floorplan](#Day2-Good-Floorplan-vs-bad-Floorplan)
- [Day3-Design Library Cell using ngspice simulations](#Day3-Design-Library-Cell-using-ngspice-simulations)
- [Day4-Pre-layout Timing analysis and CTS](#Day-4-Pre-layout-Timing-analysis-and-CTS)
- [Day5-Final steps in RTL2GDS](#Day5-Final-steps-in-RTL2GDS)

- [Reference](#reference)

# Day1

<details>
  <summary>Introduction</summary>
With the introduction of open-source technology for chip creation, many RTL designs and EDA Tools were made available for free. The [SKY130 PDK] fills the gap in a whole Open source chip development.(https://skywater-pdk.readthedocs.io/en/latest/rules.html) from Skywater Technologies and Google. There were a number of EDA Tools with distinct functions throughout the design cycle. The design flow was not clear, and the Skywater pdk was only compatible with industrial equipment.  These problems were addressed by [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane), which offered a fully automated and tidy RTL to GDSII flow. OpenLane is not a product; rather, it is a flow made up of a number of EDA tools, automation scripts, and Skywater-pdks that have been optimized for use with open-source EDA tools.    
</details>

<details>
 <summary> Overall Design Flow</summary>
An RTL design is created for a design specification using HDLs like Verilog or VHDL, or it can be created using high-level synthesis tools like SystemC, MATLAB HDL Coder, Bluespec, etc. 
The process of converting the RTL Netlist into a manufactured IC then starts, and is known as the Physical Design Flow.
Floor planning, which entails placing preplaced cells, power planning, etc., comes first in the physical design process. The placement of logical synthesis comes next. So that the clock's skew is at a minimal or under the necessary threshold, we now perform CTS (Clock Tree Synthesis). Following CTS, all of the assembled components are routed. A process known as "Static Timing Analysis" is used between each and every step in the physical design flow, from logic synthesis through routing, to analyze the design at each stage and confirm that it is actually right.  Magic is an open source application to view the layouts for every stage. You can extract a tiny netlist, run a SPICE simulation, and compare the results with the post-layout Simulation using ngspice.

Physical Design begins with Floor planning - placing the preplaced cells, power planning etc., secondly Placement 

![1](https://github.com/mavi62/IIITB_VLSI/assets/57127783/a5567e8a-fe1e-435a-a040-b6beb3423209)

  
</details>
<details>
  <summary>OpenLane Flow</summary>

### 1.  Synthesis 
The RTL Level Design is then synthesized using a Logic Synthesizer. We use Yosys which is an Open Source Logic Synthesizer. The RTL Netlist is then  converted into a synthesised netlist where there are details about the standard cells and its implementations. Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts  into  a  RTL Netlist. abc does the tehnology mapping to the required skywater-pdk variants 

### 1.1 Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area etc.,

### 1.2 Deign Exploration Utility 
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing.

### 1.3 Design For Test - DFT Insertion
This is an optional step carried out by Fault. It is used to test the design. 

###  2. Floor Planning and Power Planning
This is done by OpenROAD flow. The macros and IPs are placed in the core before proceding further. This is called as pre-placement. Floor planning is done separately for the macros and it is called macro floor planning. They are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present. Then to prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin. 

When several blocks tap power from a single source, there is a problem of Voltage Droop at the Vdd and Ground Bounce at the Vss which can again push the logic out of the required noise margin into the undefined state. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source. 

### 3. Placement
There are two types of placement.  The other required logic is placed optimally.
Placement is of two steps
- Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
- Detialed Placement - After Global placement is done minimal alterations are done to correct the issues

### 4. Clock Tree Synthesis 
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow. This is done by TritonCTS.

### 5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges
- OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
- OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode
OpenLane allows the user to chose either of the above approaches

### 6. Routing
This step is used to implement the interconnect using the different metal layers specified in the PDK. There are two steps

 - Global Routing - This is done inside the OpenROAD flow (FastRoute)
 - Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the **Logic Equivalence Check** is performed by Yosys, since OpenROAD does some optimisations the circuit.  

### 7. RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.,

### 8. STA
At this stage again OpenSTA is used to perform the Static Timing Analysis.  

### 9. Sign-off Steps
- Design Rule Check (DRC) is performed by Magic
- Layout Versus Schematic (LVS) is performed by Netgen

### 10. GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file 

## OpenLane Installation and Environment Setup

Prior to the installation of the OpenLane install the dependencies and packages using the command shown below :</br>
``` 
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
```

**Steps to install OpenLane, PDKs and Tools**</br>
```
cd $HOME
git clone https://github.com/The-OpenROAD-Project/OpenLane --recurse-submodules 
cd OpenLane
make
make test
cd /C:/mavi/OpenLane/designs/ci
cp -r * ../
```

### Steps to run synthesis in OpenLane:


```
cd ~/OpenLane
make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```

OpenLane invokes the following

- `Yosys` - RTL Synthesis and maps to yosys generic cells
- `abc` - Technology mapping with the Skywater130 PDK. Here `sky130_fd_sc_hd` Skywater Foundry produced High density standard cells are used.
- `OpenSTA` - This does the Static Timing Analysis on the netlist generated after synthesis and generated the timing reports 

View the synthesis statistics

![2](https://github.com/mavi62/IIITB_VLSI/assets/57127783/ff6b9321-2dc8-483e-900e-0ca131f048a3)


### Key concepts

#### Flops ratio 

- The flop ratio is defined as the ratio of the number of flops to the total number of cells
- Here flop ratio is **1596/10104 = 0.1579** (i.e: 15.8%) [From the synthesis statistics]


</details>

# Day2
<details>
<summary>Chip Floor Planning Consideration</summary>
  
#### Utilisation Factor

- The ratio of area occupied by the cells in the netlist to the total area of the core
- Best practice is to set the utilisation factor less than 50% so that there will be space for optimisations, routing, inserting buffers etc.,

### Aspect Ratio

- Aspect ratio is the ratio of height to the width of the die.
- Aspect Ratio of 1 indicates that the die is a square die

## Floorplanning

Floorplanning involves the following stages

### Pre-Placed cells

- Whenever there is a complex logic which is repeated multiple times or a design given by a third-party it can be perceived as abstract black box with input and output ports, clocks etc ., 
- These modules can be either macros or IP
    - Macro  - It is a module such as CPU Core which are developed by the entity fabicating the chip
    - IP - It is an "Intellectual Propertly" which the entity fabricating the chip gets as a package from a third party or even packaged Hard IPs developed by the same entity. Common examples of IPs are SRAM, PLL, Protocol Converters etc.,

- These Macros and IPs are placed in the core at first before placing the standard cells  and power planning
- These are optimally such that the cells which are more connected to each other are placed nearby and oriented for input and ouputs

### Decoupling Capacitors to the pre placed cells
- The power lines can have some RLC component causing the voltage to drop at the node where it enters the Blocks or the ground of the cell can be at a higher potential than ideally 0V
- When this happens, there is a chance such that the logic transitions are not to the upper or lower noise margins but to the forbidden state causing the circuit to misbehave
- This is prevented by adding a capacitor in parallel with the power and ground node of the block such that the capacitor decouples the block from the power source whenever there is a logic transition

### Power Planning

- When there are several cells or blocks drawing power from the same power rail and sinking power to the same ground pin the following effects are observed
    - Whenever there is alogic transition from 1 to 0 in a large number of cells then there is a Voltage Droop in the power lines as Voltage Drops from Vdd
    - Whener there is a logic transition from 0 to 1 in a large number of cells simultaneously causes the ground potential to raise above 0V calles as Ground Bump
    - These effects pose a risk of driving the logic state out of the specified noise margin.
    - To avoid this the Vdd and Gnd are placed as a grid of horizontal and vertical tracks and the cell nearer to an intersection can tap power or sink power to the Vdd or Gnd intersection respectively

### Pin Placement
 - The input, output and Clock pins are placed optimally such that there is less complication in routing or optimised delay
 - There are different styles of pin placement in openlane like `random pin placement` , `uniformly spaced` etc.,

  </details>

  <details>

<summary>Floorplan run on OpenLANE & review layout in Magic</summary>

**Floorplan envrionment variables or switches:**
1. ```FP_CORE_UTIL``` - core utilization percentage
2. ```FP_ASPECT_RATIO``` - the cores aspect ratio
3. ```FP_CORE_MARGIN``` - The length of the margin surrounding the core area
4. ```FP_IO_MODE``` - defines pin configurations around the core(1 = randomly equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - vertical metal layer where I/O pins are placed
6. ```FP_CORE_HMETAL``` - horizontal metal layer where I/O pins are placed
 
***Note: Usually, the parameter values for vertical metal layer and horizontal metal layer will be 1 more than that specified in the files***

**Importance files in increasing priority order:**
1. ```floorplan.tcl``` - System default settings
2. ```conifg.tcl```
3. ```sky130A_sky130_fd_sc_hd_config.tcl```
 
 To run the picorv32a floorplan in openLANE:
 
 ```
 run_floorplan
 
 ```

![3](https://github.com/mavi62/IIITB_VLSI/assets/57127783/b4ea6fe0-b6e9-44ab-8734-075991faa9e0)


Post the floorplan run, a .def file will have been created within the ```results/floorplan``` directory. We may review floorplan files by checking the ```floorplan.tcl.``` The system defaults will have been overriden by switches set in conifg.tcl and further overriden by switches set in ```sky130A_sky130_fd_sc_hd_config.tcl.```

To view the floorplan, Magic is invoked after moving to the results/floorplan directory:

```
magic -T /.volare/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read picorv32.def &

```

![4](https://github.com/mavi62/IIITB_VLSI/assets/57127783/7a81b5bd-b384-4aed-a3eb-5e899ad8bc18)


One can zoom into Magic layout by selecting an area with left and right mouse click followed by pressing "z" key.

Various components can be identified by using the what command in tkcon window after making a selection on the component.

Zooming in also provides a view of decaps present in picorv32a chip.

The standard cell can be found at the bottom left corner.

You can clearly see I/O pins, Decap cells and Tap cells. Tap cells are placed in a zig zag manner or you can say diagonally
  
</details>

<details>
  <summary>
    Library Binding and Placement
  </summary>
  
  ## Netlist Binding and initial place design

First we need to bind the netlist with physical cells. We have shapes for OR, AND and every cell for pratice purpose. But in reality we dont have such shapes, we have give an physical dimensions like rectangles or squares weight and width. This information is given in libs and lefs. Now we place these cells in our design by initilaising it. 

## Optimize Placement

The next step is placement. Once we initial the design, the logic cells in netlist in its physical dimisoins is placed on the floorplan. Placement is perfomed in 2 stages:

Global Placement: Cells will be placed randomly in optimal positions which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length.
Detailed Placement: It alters the position of cells post global placement so as to legalise them.
Legalisation of cells is important from timing point of view.

Optimization is stage where we estimate the lenght and capictance, based on that we add buffers. Ideally, Optimization is done for better timing.

![5](https://github.com/mavi62/IIITB_VLSI/assets/57127783/0dc7d080-e7d3-4a2b-926d-31913b4e02b8)


## Congestion aware Placement 

Post placement, the design can be viewed on magic within results/placement directory:

```
magic -T /home/parallels/OpenLane/vsdstdcelldesign/libs/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &

```

![7](https://github.com/mavi62/IIITB_VLSI/assets/57127783/51edb889-f395-4c8b-96e6-7df507ce1036)


**Note: Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN.  It is created after post CTS. The steps are - floorplan, placement, CTS, Post CTS and then PDN**

## Need for libraries and characterization

As we know, From logic synthesis to routing and STA, each and evry stage has one thing in common i.e., logic gates/ logic cells. In order for the tool understand these gates are and their timing, we need to characterize these cells. 

# CELL DESIGN AND CHARACETRIZATION FLOWS

Library is a place where we get information about every cell. It has differents cells with different size, functionality,threshold voltages. There is a typical cell design flow steps.
1. Inputs : PDKS(process design kit) : DRC & LVS, SPICE Models, library & user-defined specs.
2. Design Steps :Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

## Standard Cell Characterization Flow

A typical standard cell characterization flow that is followed in the industry includes the following steps:

1. Read in the models and tech files
2. Read extracted spice Netlist
3. Recognise behavior of the cells
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide neccesary output capacitance loads
8. Provide neccesary simulation commands

Now all these 8 steps are fed in together as a configuration file to a characterization software called GUNA. This software generates timing, noise, power models.
These .libs are classified as Timing characterization, power characterization and noise characterization.

![8](https://github.com/mavi62/IIITB_VLSI/assets/57127783/f8035e73-58bc-458b-af37-4a9efede27b7)


# TIMING CHARACTERIZATION

In standard cell characterisation, One of the classification of libs is timing characterisation.

## Timing threshold definitions 
Timing defintion |	Value
-------------- | --------------
slew_low_rise_thr	| 20% value
slew_high_rise_thr | 80% value
slew_low_fall_thr |	20% value
slew_high_fall_thr |	80% value
in_rise_thr	| 50% value
in_fall_thr |	50% value
out_rise_thr |	50% value
out_fall_thr | 50% value

## Propagation Delay and Transition Time 

**Propagation Delay** 
The time difference between when the transitional input reaches 50% of its final value and when the output reaches 50% of its final value. Poor choice of threshold values lead to negative delay values. Even thought you have taken good threshold values, sometimes depending upon how good or bad the slew, the dealy might be still +ve or -ve.

```
Propagation delay = time(out_thr) - time(in_thr)
```
**Transition Time**

The time it takes the signal to move between states is the transition time , where the time is measured between 10% and 90% or 20% to 80% of the signal levels.

```
Rise transition time = time(slew_high_rise_thr) - time (slew_low_rise_thr)

Low transition time = time(slew_high_fall_thr) - time (slew_low_fall_thr)
```

</details>

# DAY3 Design Library Cell using ngspice simulations

<details>
  <summary>CMOS inverter ngspice simulations </summary>
  ``ngspice``  is opesoure engine where simulations are done.

  ### IO Placer revision

 - PnR is a iterative flow and hence, we can make changes to the environment variables in the fly to observe the changes in our design. 
 - Let us say If I want to change my pin configuration along the core from equvi distance randomly placed to someother placement, we just set that IO mode variable on command prompt as shown below
 ```
 set ::env(FP_IO_MODE) 2
```
## SPICE Deck Creation and Simulation for CMOS inverter

- Before performing a SPICE simulation we need to create SPICE Deck
SPICE Deck provides information about the following:
- Component connectivity - Connectivity of the Vdd, Vss,Vin, substrate. Substrate tunes the threshold voltage of the MOS.
- component values - values of PMOS and NMOS, Output load, Input Gate Voltage, supply voltage.
- Node Identification and naming - Nodes are required to define the SPICE Netlist
     For example ```M1 out in vdd vdd pmos w = 0.375u L = 0.25u``` , ```cload out 0 10f```
- Simulation commands
- Model file - information of parameters related to transistors
Simulation of CMOS using different width and lengths. From the waveform, irrespective of switching the shape of it are almost same.

![9](https://github.com/mavi62/IIITB_VLSI/assets/57127783/391cac8e-8f49-4b37-adbe-0e1a7dab20ed)


From the waveform we can see the characteristics are maintained  across all sizes of CMOS. So CMOS as a circuit is a robust device hence use in designing of logic gates. Parameters that define the robustness of the CMOS are

## Switching Threshold Vm

- The Switching Threshold of a CMOS inverter is the point where the Vin = Vout on the DC Transfer characreristics. 
- At this point, both the transistors are in saturation region, means both are turned on and have high chances of current flowing driectly from VDD to Ground called Leakage current.
 
![10](https://github.com/mavi62/IIITB_VLSI/assets/57127783/d1c46221-8cef-4ab6-a8d2-0ab4fab524b7)


Through transient analysis, we calculate the rise and fall delays of the CMOS by SPICE Simulation. As we know delays are calculated at 50% of the final values.


## Lab steps to git clone vsdstdcelldesign

- First, clone the required mag files and spicemodels of inverter,pmos and nmos sky130. The command to clone files from github link is:
```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```
once I run this command, it will create ``vsdstdcelldesign`` folder in openlane directory.

Inorder to open the mag file and run magic go to the directory

For layout we run magic command

``` magic -T sky130A.tech sky130_inv.mag & ```

Ampersand at the end makes the next prompt line free, otherwise magic keeps the prompt line busy. Once we run the magic command we get the layout of the inverter in the magic window

![11](https://github.com/mavi62/IIITB_VLSI/assets/57127783/8f824761-6912-4314-af0e-3015a03b1f97)


</details>

<details>
  <summary>Inception of Layout and CMOS Fabrication Process
</summary>
  
## Mask CMOS Fabrication

The 16-mask CMOS (Complementary Metal-Oxide-Semiconductor) fabrication process involves several crucial steps for creating integrated circuits. Let's break it down with some jargon:

1. **Substrate Selection**:
   - In the initial phase, the appropriate semiconductor substrate is chosen.

2. **Active Region Creation**:
   - To isolate the active regions for transistors, the process begins with the deposition of SiO2 and Si3N4 layers, followed by photolithography and silicon nitride etching.
   - This is known as LOCOS (Local Oxidation of Silicon), where oxide is grown in certain regions.
   - Subsequently, Si3N4 is removed using hot phosphoric acid.

3. **N-Well and P-Well Formation**:
   - The N-well and P-well regions are created separately.
   - P-well formation involves photolithography and ion implantation of p-type Boron material into the p-substrate.
   - N-well is formed similarly with n-type Phosphorus material.
   - High-temperature furnace processes drive-in diffusion to establish well depths, known as the tub process.

4. **Gate Formation**:
   - The gate is a pivotal CMOS transistor terminal that controls threshold voltages for transistor switching.
   - A polysilicon layer is deposited and photolithography techniques are applied to create NMOS and PMOS gates.
   - Important parameters for gate formation include oxide capacitance and doping concentration.

5. **Lightly Doped Drain (LDD) Formation**:
   - LDD is created to mitigate hot electron and short channel effects.

6. **Source & Drain Formation**:
   - Thin oxide layers are added to avoid channel effects during ion implantation.
   - N+ and P+ implants are performed using Arsenic implantation and high-temperature annealing.

7. **Local Interconnect Formation**:
   - Thin screen oxide is removed through etching in HF solution.
   - Titanium deposition through sputtering is initiated.
   - Heat treatment results in chemical reactions, producing low-resistant titanium silicon dioxide for interconnect contacts and titanium nitride for top-level connections, enabling local communication.

8. **Higher Level Metal Formation**:
   - To achieve suitable metal interconnects, non-planar surface topography is addressed.
   - Chemical Mechanical Polishing (CMP) is utilized by doping silicon oxide with Boron or Phosphorus to achieve surface planarization.
   - TiN and blanket Tungsten layers are deposited and subjected to CMP.
   - An aluminum (Al) layer is added and subjected to photolithography and CMP.
   - This constitutes the first level of interconnects, and additional interconnect layers are added to reach higher-level metal layers.

9. **Dielectric Layer Addition**:
   - Finally, a dielectric layer, typically Si3N4, is applied to safeguard the chip.

This complex process results in the creation of advanced integrated circuits with multiple layers of interconnects, essential for modern electronic devices.

<img width="1175" alt="12" src="https://github.com/mavi62/IIITB_VLSI/assets/57127783/be1107ab-a442-4f53-bd92-3429207e32ae">


## SKY130 basic layer layout and LEF using inverter

- From Layout, we see the layers which are required for CMOS inverter. Inverter is, PMOS and NMOS connected together.
- Gates of both PMOS and NMOS are connected together and fed to input(here ,A), NMOS source connected to ground(here, VGND), PMOS source is connected to VDD(here, VPWR), Drains of PMOS and NMOS are connected together and fed to output(here, Y). 
The First layer in skywater130 is ``localinterconnect layer(locali)`` , above that metal 1 is purple color and metal 2 is pink color.
If you want to see connections between two different parts, place the cursor over that area and press S one times. The tkson window gives the component name.

![13](https://github.com/mavi62/IIITB_VLSI/assets/57127783/667e8a7d-8194-47a6-86b3-c25547e82cd6)


### Library exchange format (.lef)

- The layout of a design is defined in a specific file called LEF.
-  It includes design rules (tech LEF) and abstract information about the cells. 
    -  ```Tech LEF``` -  Technology LEF file contains information about the Metal layer, Via Definition and DRCs.
    -  ```Macro LEF``` -  Contains physical information of the cell such as its Size, Pin, their direction.
 
## Designing standard cell and SPICE extraction in MAGIC 

-  First we need to provide bounding box width and height in tkson window. lets say that width of BBOX is 1.38u and height is 2.72u. The command to give these values to magic is
   ``` property Fixed BBOX (0 0 1.32 2.72)  ```
- After this, Vdd, GND segments which are in metal 1 layer, their respective contacts and atlast logic gates layout is defined
Inorder to know the logical functioning of the inverter, we extract the spice and then we do simulation on the spice. To extract it on spice we open TKCON window, the steps are
- Know the present directory - ``pwd ``
- create an extration file -  the command is  `` extract all `` and  ``sky130_inv.ext`` files has been created
          
- create spice file using .ext file to be used with our ngspice tool  - the commands are  
      ``` ext2spice cthresh 0 rthresh 0 ``` - extracts parasatic capcitances also since these are actual layers - nothing is created in the folder
      ``` ext2spice ``` - a file ```sky130_inv.spice``` has been created.
  
![14](https://github.com/mavi62/IIITB_VLSI/assets/57127783/6e834382-c0d5-4f3c-9638-86e9f7eff228)


</details>

<details>
  <summary> SKY130 Tech File Labs </summary>
  
## Create Final SPICE Deck

let us see what is inside the spice Deck
In the spice file subcircuit(subckt), pmos and nmos node connections are defined
   
For NMOS  ``` XO Y A VGND VGND sky130_fd_pr_nfet_01v8 ``` . The order is  ``` Cell_name Drain Gate Source Substrate model_name ``` .
For PMOS  ``` X1 Y A VPWR VPWR sky130_fd_pr_pfet_01v8 ``` . The order is   ``` cell_name Drain Gate Source Substrate model_name ```.
   
For transient anaylsis, we would like to define these following connections and extra nodes for these in spice file
  - VGND to VSS
  - Supply voltage from VPWR to Ground - extra nodes here will be 0 and VDD with a value of 3.3v 
  - sweep in/pulse between A pin and VGND (0)
Before, editing the file, make sure scaling is proper, we measure the value of the gride size from the magic layout and define using `` .option scale=0.01u`` in the Deck file.
  
Now keeping the connection in mind, define the required commands in the file. Along with this we need to include libs for nmos ``nshort.lib`` and pmos ``pshort.lib`` and define transient analysis commands too. We comment the subckt since we are trying to input the controls and transient analysis also. Model names are changed to ``nshort_model.0`` and ``pshort_model.0`` according to the libs of nmos and pmos.
  
These voltage sources and simulation commands are defined in the Deck file.

   ``
.include ./libs/pshort.lib
.include ./libs/nshort.lib
   VDD VPWR 0 3.3V
   VSS VGND 0 0V
   Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)
   .tran 1n 20n
   .control
   run
   .endc
   .end
   ``
   
![15](https://github.com/mavi62/IIITB_VLSI/assets/57127783/7e7783e5-3074-4d44-8227-5325e1f76e52)


## Using ngspice for spice simulation
  
Spice Deck is done and now to run spice simulation invoke ngspice in the tool and pass the source file. 
 
  ``` ngspice sky130_inv.spice ```
  
On the prompt you can see the values the ngspice has taken. To see the plot, use
   
   ``` plot y vs time a ``` 
   
![16](https://github.com/mavi62/IIITB_VLSI/assets/57127783/17ee3f4a-b635-4bd2-8fc2-ff2d6ef60060)


![17](https://github.com/mavi62/IIITB_VLSI/assets/57127783/a2389c4a-96b8-4179-92e5-e01ac8a13cdb)

## Standard cell characterization of CMOS Iinverter 
 
characterization of the inverter standard cell depends on Four timing parameters
 
 **Rise Transition**: Time taken for the output to rise from 20% to 80% of max value
 **Fall Transition**: Time taken for the output to fall from 80% to 20% of max value
 **Cell Rise delay**: difference in time(50% output rise) to time(50% input fall)
 **Cell Fall delay**: difference in time(50% output fall) to time(50% input rise)
 
 The above timing parameters can be computed by noting down various values from the ngspice waveform.
 
 ``` Rise Transition : 2.25421 - 2.18636 = 0.006785 ns / 67.85ps ```
 ``` Fall Transitio : 4.09605 - 4.05554 = 0.04051ns/40.51ps ```
 ```Cell Rise Delay : 2.21701 - 2.14989 = 0.06689ns/66.89ps ```
 ```Cell Fall Delay : 4.07816 - 4.05011 = 0.02805ns/28.05ps ```

 ## LAB exercise and DRC Challenges

## Intrdocution of Magic and Skywater DRC's

  - In-depth overview of Magic's DRC engine
  - Introduction to Google/Skywater DRC rules
  - Lab : Warm-up exercise : Fixing a simple rule error
  - Lab : Main exercie : Fixing or create a complex error

 # Sky130s pdk intro and Steps to download labs
  
  - setup to view the layouts
  - For extracting and generating views, Google/skywater repo files were built with Magic
  - Technology file dependency is more for any layout. hence, this file is created first.
  - Since, Pdk is still under development, there are some unfinished tech files and these are packaged for magic along with lab exercise layout and bunch of stuff into the tar ball
 
We can download the packaged files from web using ``wget `` command. wget stands for web get, a non-interactive file downloader command.
  
  ``` wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz```
  
once extraction is done, drc_tests file is created and you will have all the information about magic layout for this lab exercise

Now run MAGIC

For better graphics use command ``magic -d XR ``

Now, lets see an example of simple failing set of rules of metal 1 layer.  you can either run this by magic command line `` magic -d XR met1.mag `` or from the magic console window, `` menu - file - open -load file9here, met1.mag) ``

![18](https://github.com/mavi62/IIITB_VLSI/assets/57127783/5bc08494-f545-4f56-9f87-397617d6c1c5)


## Load Sky130 tech rules for drc challenges 

First load the poly file by ``load poly.mag`` on tkcon window.

Finding the error by mouse cursor and find the box area, Poly.9 is violated due to spacing between polyres and poly.

![19](https://github.com/mavi62/IIITB_VLSI/assets/57127783/d1a73869-0e80-4939-87eb-28b508345700)


In line

```
spacing npres *nsd 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```
change to

```
spacing npres allpolynonres 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```
Also,
```
spacing xhrpoly,uhrpoly,xpc alldiff 480 touching_illegal \

	"xhrpoly/uhrpoly resistor spacing to diffusion < %d (poly.9)"
```

change to 

```
spacing xhrpoly,uhrpoly,xpc allpolynonres 480 touching_illegal \

	"xhrpoly/uhrpoly resistor spacing to diffusion < %d (poly.9)"

```
![20](https://github.com/mavi62/IIITB_VLSI/assets/57127783/2e957d26-b148-4031-9941-12241ad63a77)


</details>
