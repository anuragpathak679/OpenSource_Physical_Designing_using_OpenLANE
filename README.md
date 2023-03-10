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

![image](https://user-images.githubusercontent.com/125293287/224706396-d7680022-d12e-4921-9a3e-58c933bfbf1f.png)

## Day 3 - Design library cell using Magic Layout and ngspice characterization

### Spice netlist
Spice netlist of any layout is required to simulate and verify the functionality. A spice deck contains following information,
- Component connectivity
- Component value
- Identify nodes
- name nodes

![image](https://user-images.githubusercontent.com/125293287/224905465-5e5aebe4-9abf-4ec4-949f-434207388224.png)

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
![image](https://user-images.githubusercontent.com/125293287/224896774-ec7df8a2-eb6e-4af9-8d9d-5acd39c69ce8.png)

## 16-Mask CMOS Process

- Selecting a substrate

![image](https://user-images.githubusercontent.com/125293287/224897820-471c2904-c00f-4faa-838b-aaef120a96a1.png)

- Creating active region for transistors

![image](https://user-images.githubusercontent.com/125293287/224898466-b9f4dc61-85f9-4850-991a-ba0d02744b84.png)

- Nwell & Pwell formation (Twin Tub Process)

![image](https://user-images.githubusercontent.com/125293287/224899038-e87534ae-fb03-44bd-a77c-d80424f22379.png)

- Formation of Gate terminal

![image](https://user-images.githubusercontent.com/125293287/224899616-5e3d1e53-722b-4087-9b16-c97a101e3047.png)

- LDD (Lightly Doped Drain) Formation it will avoid hot electron effect and short channel effect 

![image](https://user-images.githubusercontent.com/125293287/224900204-cd4ab625-2031-4ca5-a79c-7a196a96c477.png)

- Source and Drain Formation (High temperature annealing)

![image](https://user-images.githubusercontent.com/125293287/224900972-0bdbc8de-abbf-4082-9886-d626ee6c2fec.png)

- Contact and local Interconnects formation (Sputtering, RCA cleaning)

![image](https://user-images.githubusercontent.com/125293287/224901585-aed31133-aa12-40ef-b19e-82c48ecf1d1a.png)

- Higher Level metal layer formation

![image](https://user-images.githubusercontent.com/125293287/224902629-c3baeaaa-bb1c-497b-a2ba-4eaddc693791.png)

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
Take a snippet of spice netlist

## Transient Analysis using ngspice

- For running transient simulation using ngspice, need to include the relevent library data, source information and command in the spice netlist to run the simulation.

```
M0 Y A VGND VGND nshort_model.0 ad=0 pd=0 as=0 ps=0 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0  ad=0 pd=0 as=0 ps=0 w=37 l=23
VDD VPWR 0 3.3V
VSS VGND 0 0V

```
- To invoke ngspice we just have to use following commands

```
ngspice <path_to_modifed_spice_netlist>
```

![image](https://user-images.githubusercontent.com/125293287/224974450-24428061-730d-47f3-9333-ebcfd04cf1ff.png)

- After that in the ngspice console we can use plot function to get the Inverter characterstic, input and output voltage wrt time.

```
ngspice 1 -> plot Y vs time A
```
<p align="left" width="100%">
    <img src="https://user-images.githubusercontent.com/125293287/224974245-55fcd939-3251-4af0-ab36-8d9d662aff6a.png"> 
</p>

- This waveform is used to do the timing characerization.
- Parameters like rise time delay, fall time delay, propagation delay are calculated.
