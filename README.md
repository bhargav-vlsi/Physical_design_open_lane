# Physical_design_open_lane


<details>
<summary>DAY-1</summary>

### Introduction to package, chip, pads, core, die and IPs
This section explains about various terminology used in ASIC chip design. 

Let us consider Arduino board which is basic embedded toolkit used for embedded programming. This arduino board has a processor chip which contains multiple interfaces for various applications. 
Package refers to housing where integrated circuit is placed.
Chip is placed usually at centre of package where leads of package are connected  through thin wires.
Pads are placed to send or received signals from or to leads of package and core.
Core refers to actual circuit designed with particular components and technology process which handles the logic.
Die is base of chip on which entire integrated circuit is built and cut out off wafer.
IPs are kind of blackbox where functionality of circuit is known and not design. We usually use IPs where we reuse of existing code in the form of IPs.

### Introduction to open source ASIC design flow
At every level, from the transistor level to the architectural level, computer programmes are used to build both analog and digital electronics. These tools support chip designers from RTL to GDS.
We have tools like Openlane, Openroad as EDA tool. Process design kit is collection of files that is used to model a fabrication process for EDA tools for designing a IC. It contains design rules like DRC & LVS, device models, standard cell libraries and I/O libraries. Google joined hands with skywater to produce open source PDK in 130nm technology node.

We have following ASIC flow:

![Asic_flow](./Images/Asic_flow.png)

1. Synthesis: Converts RTL code to gate level netlist from standard cell libraries.

2. Floor & power planning: Decides partition between different system blocks and places I/O pads. We place power rails to provide power to various components of system.

3. PLacement: We place standard cells from netlist on decided floor plan.

4. Clock tree synthesis: We create a clock distribution network to deliver clock signals sequential part of system.

5. Routing: Interconnection of blocks using metal layers.

6. Final verification: We perform DRC-design rule check, Layout vs schematic check, Static timing analysis.

### Introduction to Open-lane

OpenLane is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, CVC, SPEF-Extractor, KLayout and a number of custom scripts for design exploration and optimization. The flow performs all ASIC implementation steps from RTL all the way down to GDSII. 

![Openlane_flow](./Images/Openlane_flow.png)

### Various Open source tools in ASIC flow

RTL simulation: Iverilog & gtkwave

RTL synthesis & mapping: yosys

Floor planning : ioplacer

PLacement: OpenPD

STA: OpenSTA

clock tree synthesis: Triton CTS

Routing: TritonRoute

SPEF extraction: SPEF-extractor

DRC, GDS-II: Magic

LVS: Netgen

Circuit simulation: ngspice

### Openlane tool

We follow below steps to invoke the tool.
Go to openlane folder created in home folder.
```
make mount 
```

```
OpenLane Container (2264b12):/openlane$ ./flow.tcl -interactive
% package require openlane 0.9
```

![openlane_invoke](./Images/openlane_invoke.png)


Then prepare the design for RTl-GDS flow and run a synthesis command for picorv32a design as a sample. The picorv32a is present in design folder of openlane along with few other sample designs.
```
prep -design picorv32a
run_synthesis
```

Then we review result of synthesis flow. A folder by the name runs is created in picorv32a folder which contains folders related to ASIC flow like placement, synthesis, routing etc. We access report folder os synthesis and analyze the result as follows.

![picorv32a_dff_count](./Images/picorv32a_dff_count.png)


</details>

<details>
<summary>DAY-2</summary>

### Utilization factor and aspect ratio

Core is where actual circuit netlist is placed and die just encapsulates the core. We are interested to understand area, utilization factor and aspect ratio of core.

If we have any logical circuit, we assume it be a square based area, we try to determine the area of core where we can fit in out circuit. 

Area is simpliy the sum of product of width and height of standard cells and flip flops. 


Utilization factor is ratio of area occupied by netlist to total area of core. From this we find that, area of netlist and core is not always same. If this ratio is 1, then it is 100% utilization of core and no wastage of area.

```
Utilization factor = Area occupied by circuit / Area of core
```

Aspect ratio is ratio of height to width of core. If this ratio is 1, then is is means that core is square in shape.
```
Aspect ratio = Height of core / Width of core
```

### Concept of pre-placed cells

Let's consider that we have a circuit which performs a certain function in top level module. But, we will separate them into multiple blocks where interconnect each of them again through wires. The importance of this concept lies in the fact that we may have a functionality being implemented in multiple plcaes, we need not separately implement. We implement this block once and have multiple copies used for better & faster implementation. Some of these blocks found in market are memory, multiplexers, comparators and many more. These are called as pre-placed blocks.

![pre_placed_cells](./Images/pre_placed_cells.png)

### Decoupling capacitor

Decoupling capacitors are used to maintain stable supply to internal digita circuits. Without these, due to presence of wire resistance & inductance, the voltage represented by logic 1  or 0 might not be achieved due to noise margin of circuit. We want the voltage levels to lie within noise margin to able to distinguish between logic 1 & 0.

### Power planning

Power planning in chip design is an important aspect. Let us consider that we have circuit with on power supply. We have used decipuling capacitors for input ports to avoid destrcution of voltage levels. But is not possible to add these capacitors everywhere as it increases the size & feasible solution. Instead we increase the power supply given to chip so that particular logical part of circuit receives power from nearest power rail. Without this power planning, ground bounce where many points are discharging to single ground and voltage level of ground increases beyond noise marging causes ambiguity in logic level. Same concept applies to voltage drop where power voltage drops if many points in circuit draw power at same time.

### Pin placement
Pin placement refers to deciding input and output ports location on core. It decides delay and amount of wire requried to connect blocks. So it decides size of pins to provide power signal strength. We place these pins between space die and core border. This space does not contain any other cells of circuit.


### Floorplan of picorv32a

We perform floorplanning we use following command.
```
run_floorplan
```

![run_floorplan](./Images/run_floorplan.png)

Then we go to results folder of floorplan and open floorplan file .def in magic tool as shown below.
```
magic -T <techfile> lef read <lef_file> def read <def-file>
```

![picorv32a_floorplan](./Images/picorv32a_floorplan.png)

### Netlist binding and initial place design

After we design the system with netlist consisting of various cells, we consider these cells. These cells are taken from library where size & delay and other details associated with each cell. We take the floorplan performed in previous step for placement & routing. We place standard cells in a way similar to netlist like placing a cell closer to input port and placing another cell closer to output to have lesser delay. This is known as initial placement.


### Optimized placement using estimated wire-length and capacitance

In this stage, we estimate length & capacitance of wires to determine the optimized placement of cells. So if we have not maintained signal integrity, then we use buffers to reduce wire length and capacitance and have optimized placement.

### Placement step in openlane

Placement occurs in two stages: GLobal & detailed placement.
Global Placement: It finds optimal position for all cells which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length.

Detailed Placement: It alters the position of cells post global placement so as to legalise them.

We peform placement in openlane as follows:
```
run_placement
```

![run_placement](./Images/run_placement.png)

![picorv32a_placement](./Images/picorv32a_placement.png)

### Cell design

Standard cell design flow involves the following:

-Inputs: PDKs, DRC & LVS rules, SPICE models, libraries, user-defined specifications.

-Design steps: Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).

-Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib file

### Standard cell characterisation

Standard cell characterization follows below step:

Logic (Boolean function)

Schematic (Connection pins only)

Netlist (Internal circuit made of transistors)

Netlist with parasitics

Physical Layout

Timing (Delays, hold and setup times, ...)

Power

Noise

We use software called GUNA to perform above characterization steps.

### Timing characterization parameters

We have timing threshold definitions as follows:

slew_low_rise_thr	20% value

slew_high_rise_thr	80% value

slew_low_fall_thr	20% value

slew_high_fall_thr	80% value

in_rise_thr	        50% value

in_fall_thr	        50% value

out_rise_thr	        50% value

out_fall_thr	        50% value


Propogation delay: Time difference between input waveform & output waveform crossing 50% of reference value.
Poor choice of threshold value can lead to negative delays.



</details>



<details>
<summary>References</summary>

https://github.com/kunalg123/

https://openlane.readthedocs.io/en/latest/getting_started/installation/installation_ubuntu.html#installation-of-required-packages

https://github.com/The-OpenROAD-Project/OpenLane

https://vsdiat.com/

https://github.com/Devipriya1921/Physical_Design_Using_OpenLANE_Sky130

</details>
