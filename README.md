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
<summary>DAY-3</summary>


### IO placer revision

![equi_pin_placement](./Images/equi_pin_placement.png)

Previously, we used equidistant value for IO pins in layout. Now we want to change to some other format, we edit environment variable and run floorplan flow again as follows:

```
set ::env(FP_IO_MODE) 2
```

![different_io](./Images/different_io.png)

### Spice deck

A SPICE deck includes information about the following:

Model description

Netlist description

Component connectivity

Component values

Capacitance load

Nodes

Simulation type and parameters

Libraries included

Following is Spice netlist for inverter.

![inverter](./Images/inverter.png)

```
.title CMOS inverter
.include 
M1 out in vdd vdd pmos w=0.375u l=0.25u
M2 out in 0 0 nmos w=0.375u l=0.25u
cload out 0 10f
Vdd vdd 0 2.5
Vin in 0 2.5

**control cmds
.op
.dc Vin 0 2.5 0.05
.plot v(out) vs v(in)
.end
```

We change width of pmos to 0.9375 then we get following characteristics with shifted threshold.



We compare as follows:

| Inverter wp=0.375 | Inverter wp=0.9375 |
| --- | --- |
| ![inverter_dc](./Images/inverter_dc.png) | ![inverter2_dc](./Images/inverter2_dc.png) |



### Switching characteristics

In this section, we try to understand switching characteristics ie rise delay & fall delay of inverter.

We use previous netlist and provide a pulse as input to determine rise & fall delay.

| PMOS W/L ratio | NMOS W/L ratio | Rise delay | Fall delay |
| --- | --- | --- | --- |
| Wp/Lp | Wn/Ln | 100.97ps | 49.61ps |
| Wp/Lp | 2Wn/Ln | 109.88ps | 32.03ps |
| Wp/Lp | 3Wn/Ln | 119.15ps | 23.721ps |

### Inverter layout

We git clone vsdstdcelldesign github repository for inverter layout.
```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```
![inverter_layout](./Images/inverter_layout.png)

### 16 Fabrication of Mask CMOS
The following steps comprise the 16-mask CMOS process:

choosing a substrate: separating the substrate/body material.

Making a transistor's active region: SiO2 and Si3N4 etching and deposition, followed by photolithography, are used to isolate between active area pockets.

Ion implanation for the creation of the N- and P-wells: boron for the P-well and phosphorus for the N-well.

Photolithography processes are used to produce the NMOS and PMOS gates at the gate terminal.

LDD formation: LDD developed to counteract the hot electron effect.

In order to prevent channelling during implants, screen oxide is applied before aresenic is implanted, followed by annealing.

Local connection formation: HF etching is used to remove screen oxide. Ti is deposited for low-resistance connections.

Planarization of higher level metals using CMP, followed by TiN and Tungsten deposition. Top SiN layer for chip protection.

### Spice extraction & simulation

In this section, we will verify the logic implemented by layout by extracting spice netlist and performing simulation in ngspice.

In Tckon window of magic, we use following commands
```
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```
We have two files sky130_inv.ext & sky130_inv.spice created.

![ext2spice](./Images/ext2spice.png)


We perform ngspice simulation for extracted netlist.

Netlist:
```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

//.subckt sky130_inv A Y VPWR VGND
M1000 Y A VPWR VPWR pshort_model.0 w=37 l=23
+  ad=1443 pd=152 as=1517 ps=156
M1001 Y A VGND VGND nshort_model.0 w=35 l=23
+  ad=1435 pd=152 as=1365 ps=148

VDD VPWR 0 3.3v
VSS VGND 0 0v
Va A VGND PULSE (0 3.3 0 0.1n 0.1n 2n 4n)

C0 VPWR A 0.07fF
C1 VPWR Y 0.11fF
C2 Y A 0.05fF
C3 Y VGND 2fF
C4 VPWR VGND 0.59fF
.end
```

```
ngspice sky130_inv.spice
tran 1n 20n
plot v(y) v(a)
```
![layout_simulation](./Images/layout_simulation.png)

We calculate Rise time, fall time, rise delay, fall delay & propogation delay.

Rise time: 63.44ps
Fall time: 42.68ps
Rise delay: 60.46ps
Fall delay: 25.58ps

### DRC violations

We try to understand DRC violations through examples.
Download sample magic layout files from following website.

```
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz
tar xfz drc_tests.tgz
```

Now, open sample file as shown
```
magic -d XR met3.mag
```

Below is rules for me3 layer.

![rules](./Images/rules.png)

We use 'drc_why' command errors in layout as shown.

![m3_drc](./Images/m3_drc.png)

We use following commands to see metal cut as shown.

```
cif see VIA2
```

![m3_metal_cut](./Images/m3_metal_cut.png)

### Fixing poly.9 error in sky130A.tech file - lab

Open the poly.mag file in magic tool.

```
magic -d XR poly.mag
```

![poly9_before](./Images/poly9_before.png)

We find that distance between regular polysilicon & poly resistor should be 22um but it is showing 17um and still no errors . We should go to sky130A.tech file and modify as follows to detect this error.

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

Again we load poly.mag file we find that it is detecting this erroe with DRC errors increased to 35 from 32.

![poly9_after](./Images/poly9_after.png)

</details>


<details>
<summary>DAY-4</summary>

### Converting grid info to track info

The requirement of ports is mentioned in track.info as shown below.

```
li1 X 0.23 0.46
li1 Y 0.17 0.34
met1 X 0.17 0.34
met1 Y 0.17 0.34
met2 X 0.23 0.46
met2 Y 0.23 0.46
met3 X 0.34 0.68
met3 Y 0.34 0.68
met4 X 0.46 0.92
met4 Y 0.46 0.92
met5 X 1.70 3.40
met5 Y 1.70 3.40
```
Before convergence, we grid as follows as:

![before_grid](./Images/before_grid.png)

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. After providing the command, we have following:
```
grid 0.46um 0.34um 0.23um 0.17um
```

![after_grid](./Images/after_grid.png)

### Conversion of magic layout to standard cell LEF file

Extraction of the LEF file for the cell comes next when the layout is completed. To help the placer and router tool, specific characteristics and definitions must be defined for the cell's pins. Ports are the macro's declared PINs, and in LEF files, a cell containing ports is written as a macro cell. Our goal is to extract LEF in a predetermined format from a configuration (in this case, a straightforward CMOS inverter). The first step is to define each port and assign the appropriate class and use characteristics to each port.

Below are steps to define a port :

First, open the.mag file for the design in the Magic Layout window. Next, select Edit >> Text to bring up a dialogue window. Use locali for port y & a, use metal 1 for vdd & gnd as shown in figures below.

![port_a](./Images/port_a.png)

![port_y](./Images/port_y.png)

![port_vdd](./Images/port_vdd.png)

![port_gnd](./Images/port_gnd.png)


Define the purpose of ports as follows in tkcon window:

```
port A class input
port A use signal

port Y class output
port Y use signal

port VPWR class inout
port VPWR use power

port VGND class inout
port VPWR use ground
```

We generate lef file by command:
```
lef write <name>
```
This generates sky130_vsdinv.lef file.

### Steps to include custom cell in ASIC design

We have created a custom standard cell in previous steps of an inverter. Copy lef file, sky130_fd_sc_hd_typical.lib, sky130_fd_sc_hd_slow.lib & sky130_fd_sc_hd_fast.lib to src folder of picorv32a from libs folder vsdstdcelldesign. Then modify the condif.tcl as follows.

```

# Design
set ::env(DESIGN_NAME) "picorv32a"

set ::env(VERILOG_FILES) "$::env(DESIGN_DIR)/src/picorv32a.v"

set ::env(CLOCK_PORT) "clk"
set ::env(CLOCK_NET) $::env(CLOCK_PORT)

set ::env(GLB_RESIZER_TIMING_OPTIMIZATIONS) {1}

set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]

set filename $::env(DESIGN_DIR)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1} {
	source $filename
}
```

To integrate standard cell in openlane flow, perform following commands:

```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```

### Delay tables

We observe that the buffer we insert to maintain signal integrity has some constraints. We observe that size of buffer in every level should have same size and have different delays depending on load driven by them. So VLSI engineers came up with concept of delay tables which consists of 2D array of values input slew & load capacitance defined for cell for different sizes. These delay tables became timing models. The algorithm takes these values and computes delay values. If delay is not available directly, it takes nearest data and determines through extrapolation.

![delay_table](./Images/delay_table.png)

### Openlane steps with custom standard cell

We perform synthesis and found that it has negative slack and met timing constraints.

We perform floorplan and find out custom cell included as follows.

![custom_cell_floorplan](./Images/custom_cell_floorplan.png)

We perform placement step as well.

![custom_cell_layout](./Images/custom_cell_layout.png)

### Setup & hold time concepts

It is here that we introduce SETUP and HOLD time. Setup time is defined as the minimum amount of time before the clock’s active edge that the data must be stable for it to be latched correctly. Any violation may cause incorrect data to be captured, which is known as setup violation.

![setup_time](./Images/setup_time.png)

Hold time is defined as the minimum amount of time after the clock’s active edge during which data must be stable. Violation in this case may cause incorrect data to be latched, which is known as a hold violation. Note that setup and hold time is measured with respect to the active clock edge only.

![hold_time](./Images/hold_time.png)

### Clock jitter concept

Circuitry in the clock generator, noise, changes in the power supply, interference from surrounding circuitry, etc. are the usual causes of clock jitter. The design margin called for in the timing closure specification includes jitter as a factor.


Period jitter is the difference between a clock signal's cycle time and the ideal period over a large number of randomly chosen cycles, such as 10K cycles. The clock period deviation can be supplied as either an average value across the chosen cycles (RMS value) or as the difference between the chosen group's highest and minimum deviations (peak-to-peak period jitter).

The difference between two consecutive clock cycles across a random number of clock cycles is known as cycle to cycle jitter, or C2C. (say 10K). Typically, this is described as the peak value for the random group.By doing so, the high frequency jitter can be calculated.

The effect being measured in the frequency domain is phase noise. In the frequency domain, phase noise is the representation of fast, short-lived, random variations in the phase of the waveform. These fluctuations can be converted into jitter values for digital design.

![clock_jitter](./Images/clock_jitter.png)

![timing](./Images/timing.png)

After putting command
```
set ::env(SYNTH_MAX_FANOUT) 4
```
we got positive slack in sta analysis.




### Clock tree synthesis

The goal of constructing a clock tree is to make sure that the clock input reaches all the elements and that there is no clock skew. The H-tree is one of the most used methods in CTS. If you have ever tried to reduce slack in a previous run, you may have noticed that the netlist has been changed by cell replacement techniques. Before trying to run a CTS in tritoncts tool.

The goal of the Clock Tree Synthesis is to reduce the routing resources of the clock signal, reduce the area of the clock repeaters, while maintaining a reasonable clock skew, reasonable clock latency, reasonable clock transition time, minimum Pulse Width, and duty cycle requirements for all the sequence elements in the design, and reasonable clock power within the spec. Clock Skew refers to the difference in the clock arrival time between two registers

Here is an example of bad tree

![bad_tree](./Images/bad_tree.png)


### Cross talk & cross net shielding

Crosstalk noise is noise generated on the clock network by aggressor nets surrounding the clock signal. This noise can delay or make the clock signal faster or even cause spurious transitions known as glitches. To maintain the signal integrity of the clock signal, physical designers protect the clock wires using a power net. They may also use NDR rules that route the clock signal by leaving one empty track next to the clock route. This helps to reduce the impact of noise on the clock network. The function of the clock signal is to control and synchronize trigger events within a synchronous design. Therefore, maintaining the signal integrity is essential to meet your design functional specification.

![glitch](./Images/glitch.png)


### Clock tree synthesis lab

We following command to run CTS in openlane:
```
run_cts
write_verilog ./designs/picorv32a/picorv32a_cts.v
```

Since clock buffers are added during the CTS run, buffer delays are now a factor, and real clocks will be used for the remainder of our research. Now, setup and hold time slacks may be examined in OpenROAD's post-CTS STA analysis for the openLANE flow:

open openroad tool using following in openlane command prompt.
```
openroad
```

![openroad](./Images/openroad.png)

```
read_lef ./designs/picorv32a/runs/RUN_2023.09.15_09.02.59/tmp/merged.nom.lef 
read_def ./designs/picorv32a/runs/RUN_2023.09.15_09.02.59/results/cts/picorv32a.def
write_db ./designs/picorv32a/picorv32a.db
read_verilog ./designs/picorv32a/picorv32a_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
read_sdc ./designs/picorv32a/runs/RUN_2023.09.15_09.02.59/results/cts/picorv32a.sdc
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

We can see that slack constraints are met.
![cts_slack_met](./Images/cts_slack_met.png)

</details>

<details>
<summary>DAY-5</summary>

### General Lee's Maze routing algorithm

![maze_algorithm](./Images/maze_algorithm.png)

1. Create an empty queue to hold the matrix's coordinates, and initialise it so that the source cell is marked as visited by having a distance from the source of 0 in the queue.

2. Start the BFS procedure by calling the source cell.

3. Set all the values in a boolean array to false and initialise it to have the same size as the input matrix. This is used to keep track of whether a coordinate has been visited.

4. Continue iterating until the queue is empty. Front cell to be released from the queue. If the target cell is reached, return. Otherwise, enqueue the cells and mark them as visited for each of the four cells that are immediately surrounding the current cell and have a cell value of 1

5. Return false if the destination is not reached after all queue elements have been processed.

### Design Rule check

A physical design technique called Design Rule Checking (DRC) is used to check whether a chip layout complies with a number of requirements set out by the semiconductor manufacturer. Each semiconductor manufacturing process will have its own set of guidelines and margins to ensure that normal manufacturing variability won't lead to chip failure.

Few types of DRCs:

Minimum width and spacing for metal

Minimum width and spacing for via

Fat wire Via keep out Enclosure

End of Line spacing

Minimum area

Over Max stack level

Wide metal jog

Misaligned Via wire

Different net spacing

Special notch spacing

Shorts violation

Different net Via cut spacing

Less than min edge length

### Power Distribution Network generation

Power Distribution Network generation is not a component of the floorplan run in OpenLANE, in contrast to the typical ASIC flow. After the CTS and post-CTS STA analyses, PDN must be prepared.

We use following command for power distribution network generation ie power and GND rails.
```
gen_pdn
```

![pdn](./Images/pdn.png)

gen_pdn – Generate a power distribution network
The power distribution network must use design_cts.def as the input def file. This creates a grid and band for Vdd and floor. These are placed around the standard cell. A standard cell is designed so that its height is a multiple of the distance between its Vdd and ground bar. The slope here is 2.72. Power can be supplied to standard cells only if the above conditions are met. The chip is powered via a power connection. There is one for Vdd and one for Gnd
Current flows from the pad to the ring through the through hole.
The strap is connected to the ring. The Vdd band is connected to the Vdd ring and the Gnd band is connected to the Gnd ring. Has horizontal and vertical support
Now we need to supply power from the tape to the standard cell. Straps are connected to standard cell rails
If a macro is present, the strap is attached to the macro's ring via the macro pad and her PDN for the macro is pre-created. Straps and rails have definitions. In this design, the tabs are on metal layers 4 and 5, and the standard cell bars are on metal layer 1. Connect layers with vias as needed.

### Routing

Routing is the process of physically connecting signal pins using metal layers. Following CTS and optimisation, routing is the phase in which precise connections between standard cells, macros, and I/O pins are made. The logical connections provided in the netlist are used to determine the creation of electrical connections in the layout utilising metals and vias. 

![routing](./Images/routing.png)

OpenLANE uses the TritonRoute tool for routing. There are 2 stages of routing:

1. Global routing: Routing region is divided into rectangle grids which are represented as course 3D routes (Fastroute tool).

2. Detailed routing: Finer grids and routing guides used to implement physical wiring (TritonRoute tool).

Running routing step in TritonRoute as part of openLANE flow:
```
run_routing
```

Finally, we can use 
```
./flow.tcl -design picorv32a
```
to run the entire flow without interaction.



</details>

<details>
<summary>References</summary>

https://github.com/kunalg123/

https://openlane.readthedocs.io/en/latest/getting_started/installation/installation_ubuntu.html#installation-of-required-packages

https://github.com/The-OpenROAD-Project/OpenLane

https://vsdiat.com/

https://github.com/Devipriya1921/Physical_Design_Using_OpenLANE_Sky130

https://github.com/nickson-jose/vsdstdcelldesign

https://www.edn.com/understanding-the-basics-of-setup-and-hold-time/

https://vlsi.pro/clock-jitter/

https://anysilicon.com/clock-tree-synthesis/

https://www.codesdope.com/blog/article/lee-algorithm/

https://vlsi-backend-adventure.com/routing.html

https://semiengineering.com/knowledge_centers/eda-design/verification/design-rule-checking-drc/

https://www.design-reuse.com/articles/41504/design-rule-checks-drc-a-practical-view-for-28nm-technology.html

</details>
