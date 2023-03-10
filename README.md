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
### Lab-1

After invoking OpenLANE first import the package of openLANE version, in our case the required version is 0.9.
```
package require openlane 0.9
```
##### Design Preparation
Prepare the design, picorv32a for the OpenLANE flow. 
```
prep -design <design-name>
````
Design preparation will merge the technology lef and standard cell lef information and create merged.lef file located in <design_folder>/runs/tag/tmp/ directory. This file contains the layer information, design rules and information about each standard cell.

Environment variables or the parameters required for the run are stored in config.tcl file.

##### Synthesis

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

### Lab-1
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

Placement is carried out as an iterative process till the value of overflow converges to 0.

### Lab-2
To run placement following command is used,
```
%run_placement
```
Same magic commnad is used with the modifed def generated after placement, post place ment layout looks like this,

![image](https://user-images.githubusercontent.com/125293287/224415113-55580e59-14ed-46e9-b59b-b76cb46c3304.png)

## Day 3 - Design library cell using Magic Layout and ngspice characterization


