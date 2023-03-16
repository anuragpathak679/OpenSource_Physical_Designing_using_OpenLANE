# OpenSource_Advanced_Physical_Design_using_OpenLANE

This repository consists of all the work done during the Advanced Physical Design workshop conducted by VSD. Here the main objective was to get a brief overview of complete RTL2GDS flow using picorv32a design.

## Details on RTL2GDS flow 

OpenLANE furnishes a complete automated RTL2GDSII flow which consists of several open source tools like OpenROAD, Yosys, ABC, Magic, Fault, TristonRoute, Netgen etc. Here, user gets the flexibility to add customize and optimize multiple designs. Following is the detailed flowchart of the OpenLANE architecture:

![image](https://user-images.githubusercontent.com/125293287/224393812-b4e36bd0-8c65-4cd9-94b0-b7b856f4bb78.png)

As mentioned in the above flowchart the flow looks something like this, (input) RTL -> Synthesis -> floorplan + powerplan -> placement -> CTS -> Routing -> GDSII (final output)

## Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK

First of all move to the openlane folder and enable docker.
```
docker
```
To invoke OpenLANE in interactive mode, run the ```./flow.tcl``` script and use -interactive flag.
``` 
./flow.tcl -interactive 
```
### Lab-1 : Design preparation and synthesis

After invoking OpenLANE first import the package of openLANE version, in our case the required version is 0.9.
```
package require openlane 0.9
```
#### Design Preparation
Prepare the design, picorv32a for the OpenLANE flow. 
```
prep -design <design-name>
````
Design preparation will merge the technology lef and standard cell lef information and create merged.lef file located in <design_folder>/runs/tag/tmp/ directory. This file contains the layer information, design rules and information about each standard cell.

Environment variables or the parameters required for the run are stored in config.tcl file.

#### Synthesis
The next step is to run_synthesis using the yosys and abc tools.

```
run_synthesis  
```
This will take some time and then a success message will come up.

![image](https://user-images.githubusercontent.com/125293287/224402145-bb1a5473-7487-45d5-8280-4ee16d7e2472.png)

## Day 2 - Good floorplan vs bad floorplan and introduction to library cells

### Floorplanning
Simple arrangement of logical block, library cells, pins on die. it ensures that every module has been assigned an appropriate area and aspect ratio, every pin of the module has connection with other modules or periphery of the chip and modules are arranged to consume lesser area.

 - Core and Die : 
Core is the section of the chip where the design's fundamental logic is placed whereas die which consists of core is a small semiconductor material specimen on which the fundamental circuit is fabricated. 
 - Utilization Factor : 
Ratio of the area of core used by standard cells to the total core area, generally kept in the range of 0.5-0.7 i.e. 50% - 60%. Maintaining a proper utilization factor is necessary for placement and routing optimization.
 - Aspect Ratio : 
The height of the core area divided by the width of the core area.
 - Preplaced Cells : 
IP's such as Memory Cells, Clock-gating Cells etc. are desiged only once and then used repeatedly in the design. These IPs are placed in chip before the placemnet and routing, so these are called preplaced cells. 
 - Decoupling Caps : 
Capacitors placed close to preplaced cells to ensure smooth transfer of logic between them. These capacitors will charge up to the power supply voltage over time and behave like the power supply which faces drop due to the interconnect wires.

### Powerplanning
Power grid network is created to distribute power to each part of the design symmetrically. This will help reduce the IR drop and charge accumulation at a particular ground, ground nounce which causes high noise margins.

 - Pin Placement : 
Decide the timing delays and number of buffers required in the whole core, the input pins of a particular block are placed near the block. Clock nets are thicker than the normal routing wire as these wires mainly drive the whole design continuously hence a low resistance path is required.

### Lab-1 : Run floorplan
To run floorplan simply use the following command.
```
%run_floorplan
```

![image](https://user-images.githubusercontent.com/125293287/224410652-fc1cb274-9084-4564-9806-80e01ab747af.png)

After successfull run DEF file will be generated which contains the design-specific information of the circuit and is a representation of the design at any point during the physical design process.

Magic tool is used to view the floorplan, It requires tech file, lef file and def file as an input.

```
magic -T <path_to_tech_file> lef read <path_to_lef_file> def read <path_to_def_file> &
```
Layout in the magic will look like this,

![image](https://user-images.githubusercontent.com/125293287/224412372-df6c5e21-7311-4e18-928a-e7b5935d6068.png)

We can check if the horizontal pins are in third layer,

![image](https://user-images.githubusercontent.com/125293287/224412191-e9db1096-8669-4154-857e-dfbec3a66b36.png)

### Placement

- Placement step determines the location of each component on the die using def and binds the netlist to the standard cell library given by foundary which contains details of cell like height, width and delay.
- Standard cells will be placed on the floorplan according to the given netlist without disturbing the preplaced cells in floorplan.
- Optimizations also take place to remove any timing violations created due to the relative placement on die.

The DEF file created during floorplan forms the input to placement step. Placement in OpenLANE occurs in two stages:
   - Global Placement : Corase Placement 
   - Detailed Placement : Fine placement of cells into standard cell rows while adhering to global placement

Process of placement is iterative till the value of overflow converges to 0.

### Lab-2 : Run placement 
To run placement following command is used,
```
%run_placement
```
Same magic commnad is used with the modifed def generated after placement, post place ment layout looks like this,

![image](https://user-images.githubusercontent.com/125293287/224415113-55580e59-14ed-46e9-b59b-b76cb46c3304.png)

### Cell design and Characterization

- Standard cell libraries contains information about each cell like area, delay, threshold voltage & power consumption along with their drive strength.
- All the stages and steps involved in the entire design of a standard cell are cumulatively called Cell Design Flow. The inputs, outputs and steps involved in the process of cell design are, 

<p width="100%">
    <img width="60%" src="https://user-images.githubusercontent.com/125293287/224706396-d7680022-d12e-4921-9a3e-58c933bfbf1f.png"> 
</p>

## Day 3 - Design library cell using Magic Layout and ngspice characterization

### Spice netlist
Spice netlist of any layout is required to simulate and verify the functionality. A spice deck contains following information,
- Component connectivity
- Component value
- Identify nodes
- name nodes

<p width="100%">
    <img width="30%" src="https://user-images.githubusercontent.com/125293287/224905465-5e5aebe4-9abf-4ec4-949f-434207388224.png"> 
</p>

### Threshold Voltage of CMOS

- The voltage which input voltage to an CMOS inverter is equal to the output voltage, i.e. Vin = Vout. 
- It is a function of the W/L ratio of the device and varying the W/L ratio will vary the output and the transfer charecteristic of the device.
- A perfectly symmetrical device will have a switching threshold such that Vin = Vout = VDD/2 which is achieved when (W/L) ratio of PMOS is approximatly 2.5 times the (W/L) ratio of NMOS.

### Lab-1 : CMOS Inverter Design using Magic
- Magic tool is being used to design the CMOS inverter. It also has inbuilt DRC check feature
- It takes the mag file and the technology file as an input (sky130A.tech in this case). 
- Clone the repo from https://github.com/nickson-jose/vsdstdcelldesign.git. 
- Run the magic software using the following command
```
magic -T sky130A.tech sky130_inv.mag
```
<p width="100%">
    <img width="80%" src="https://user-images.githubusercontent.com/125293287/224896774-ec7df8a2-eb6e-4af9-8d9d-5acd39c69ce8.png"> 
</p>

## 16-Mask CMOS Process

- Selecting a substrate

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224897820-471c2904-c00f-4faa-838b-aaef120a96a1.png"> 
</p>

- Creating active region for transistors

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224898466-b9f4dc61-85f9-4850-991a-ba0d02744b84.png"> 
</p>

- Nwell & Pwell formation (Twin Tub Process)

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224899038-e87534ae-fb03-44bd-a77c-d80424f22379.png"> 
</p>

- Formation of Gate terminal

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224899616-5e3d1e53-722b-4087-9b16-c97a101e3047.png"> 
</p>

- LDD (Lightly Doped Drain) Formation it will avoid hot electron effect and short channel effect 

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224900204-cd4ab625-2031-4ca5-a79c-7a196a96c477.png"> 
</p>

- Source and Drain Formation (High temperature annealing)

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224900972-0bdbc8de-abbf-4082-9886-d626ee6c2fec.png"> 
</p>

- Contact and local Interconnects formation (Sputtering, RCA cleaning)

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224901585-aed31133-aa12-40ef-b19e-82c48ecf1d1a.png"> 
</p>

- Higher Level metal layer formation

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224902629-c3baeaaa-bb1c-497b-a2ba-4eaddc693791.png"> 
</p>

### Lab-2 : Extract Spice Netlist from layout of the standard cell

Extraction of spice netlist from the standard cell layout is done using the magic software in 3 steps,
- Create the extraction file 
```
extract all
```
- Above step will create a .ext file which is used to generate the spice file.
```
ext2spice cthresh 0 rthresh 0
ext2spice
```
- The generated spice file will be used for simulation using ngspice.

![image](https://user-images.githubusercontent.com/125293287/225527647-571c0a4d-d1c1-4254-8ebf-49621501f629.png)

## Transient Analysis using ngspice

- For running transient simulation using ngspice, need to include the relevent library data, source information and command in the spice netlist to run the simulation.

```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib
//.subckt sky130_inv A Y VPWR VGND
M0 Y A VPWR VPWR pshort_model.0 ad=1443 pd=152 as=1517 ps=156 w=37 l=23
M1 Y A VGND VGND nshort_model.0 ad=1435 pd=152 as=1365 ps=148 w=35 l=23
C0 VPWR A 0.07fF
VDD VPWR 0 3.3V
VSS VGND 0 0V
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)
C1 Y A 0.05fF
C2 Y VPWR 0.11fF
C3 Y VGND 2fF
C4 VPWR VGND 0.59fF
.tran 1n 20n
.control
run
.endc
.end

```
- To invoke ngspice we just have to use following commands

```
ngspice <path_to_modifed_spice_netlist>
```

<p align="left" width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/225529053-c4e3d5e8-2b87-4013-b702-75cfbae79b2c.png"> 
</p>

- After that in the ngspice console we can use plot function to get the Inverter characterstic, input and output voltage wrt time.

```
ngspice 1 -> plot Y vs time A
```
<p align="left" width="100%">
    <img width="60%" src="https://user-images.githubusercontent.com/125293287/225529305-bf40284b-6cd2-47ba-bc06-e93a7a64c9b3.png"> 
</p>


- This waveform is used to do the timing characerization.
- Parameters like rise time delay, fall time delay, propagation delay are calculated.

## Day 4 - Pre-layout timing analysis and importance of good clock tree

- PnR requires just the pin placement and metal information, there is no need of providing any logic.
- To incorporate any standard cell layout in OpenLANE RTL2GDS flow, it should be converted to a standard cell LEF. 
- LEF stands for Library Exchange Format. It performs the interconnect routing in conjunction to routing guides generated from the PnR flow. 

### Exract LEF file using magic software

- Before creating the LEF file ensure that the design of the standard cell is honoring the foundry requirments.
- ```tracks.info``` file gives information about the offset and pitch (minimum permissible grid size) of a track in a given layer both in horizontal and vertical direction. The track information is given in below mentioned format.
 
```
<layer-name> <X/Y direction> <track-offset> <track-pitch>
```
![image](https://user-images.githubusercontent.com/125293287/224986096-6e75fe05-6467-4884-800e-bd2f9a2abb19.png)

To convert existing standard cell layout into LEF, following aspects should be taken into account,

1. The input and ouptut of the cell should fall on intersection of the vertical and horizontal grid lines.

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224987187-dcb11765-11a7-43c4-adfe-cef96a958516.png"> 
</p>

2. Height of the cell should be odd multiple of the vertical track pitch, to ensure ```VPWR``` and ```VGND``` properly fall on the PDN.
3. Width of the cell should be an odd multiple of the horizontal track pitch.

<p width="100%">
    <img width="40%" src="https://user-images.githubusercontent.com/125293287/224987673-ed6de716-ef6a-4810-a1af-2b53a85fd266.png"> 
</p>

We can veriy from the above image that the width and height of rectangle are odd multiple of x and y pitch respectively from ```tracks.info``` file.

After this all the ports of the custom inverter cell should be defined and correct class and use attributes for each port should be set.

<p width="100%">
    <img width="70%" src="https://user-images.githubusercontent.com/125293287/224992366-c7f47842-dd43-4b44-8c41-e0f1adbf3431.png"> 
</p>

After setting the properties, next step is to generate the LEF file.

```
lef write sky130_anurag_inv.lef
```
<p width="100%">
    <img width="20%" src="https://user-images.githubusercontent.com/125293287/224993966-64a6f16a-9b58-4dc6-bc21-8589ae5bdb0d.png"> 
</p>

### Include custom cell LEF in picorv32a design

After generating the LEF file do some changes in the ```config.tcl``` file for our project like,

- include the lef file for custom inverter cell
```
set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```
- Also we need to set the library path to include standard cell information
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
```
After modifying the config file run synthesis, and just before ```run_synthesis``` step add following commands in the console.
```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```
<p width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/125293287/225566867-f2e04050-f5f7-43fc-add4-a950f3e7fafe.png"> 
</p>

After running synthesis we can see that ```tns (total negative slack)``` and ```wns (worst negative slack)``` to be negative, it implies there is some violation which we will have to fix.

### Standalone STA using OpenSTA
STA or Static Timing Analysis can be performed outside the OpenLANE flow by directly invoking OpenSTA. 

This requires extra configuration to be done to specific the verilog file, constraints, clcok period and other required parameters.
OpenSTA is invoked using the below mentioned command.

```
set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
read_liberty -min /home/anurag.pathak/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib
read_liberty -max /home/anurag.pathak/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib
read_verilog /home/anurag.pathak/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/10-03_08-44/results/synthesis/picorv32a.synthesis.v
link_design picorv32a
read_sdc /home/anurag.pathak/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc
report_checks -path_delay min_max -fields {slew trans net cap input_pin}
report_tns
report_wns
```
