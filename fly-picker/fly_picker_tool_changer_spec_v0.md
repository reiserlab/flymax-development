
# Objective

Develop a quick-release vacuum tool that can be used by the FlyMAX system to pick up a fly using vacuum, and transfer the vacuum-tethered fly to an experimental station.

This project will also develop a tool holder for the receiving station, and a tool rack to hold empty pickers.

# Concept of Operations

1. Tools are placed by hand in a rack within reach of the gantry system.
2. FlyMAX system moves over the rack (known position where the next tool to be picked up is).
3. Gantry moves delta robot down to standoff position about 10mm above the tool.
4. Delta robot does vertical move to pick up the tool.
5. Tool engages with delta robot tool holder.
6. Delta robot retracts.
7. Gantry retracts.
8. Needle tip location in robot frame is auto-calibrated with TBD method.
9. Delta robot is moved to fly pick station.
10. Delta robot finds and picks a fly (VACUUM ON).
11. Pick quality is verified.
12. Fly is inspected.
13. IF Good pick AND Good fly:
	1. Move delta robot to fly transfer station.
	2. Gantry moves down to standoff height.
	3. Delta robot moves down to transfer position.
	4. Tool is secured by receiving station.
	5. Delta robot retracts to standoff height, tool separates from robot.
	6. VACUUM OFF
	7. Gantry retracts and return to step 2 in the flow.

The receiving station can be equipped with its own vacuum source to keep the fly tethered, or it can be fully passive if the tool is able to maintain vacuum for an extended duration.

# Requirements

|Requirement ID|Requirement Name|Description|Notes|
|---|---|---|---|---|
|FLYMAX-TOOL-001|Vacuum Function|The picker tool shall pick up a fly using vacuum.||
|FLYMAX-TOOL-002|Ejector Pulse Function|The picker tool shall enable fly release via a puff of air (ejector pulse).||
|FLYMAX-TOOL-003|Quick Engagement Function|The tool shall be pickable-uppable by the robot using only linear motions.|One axis at a time.|
|FLYMAX-TOOL-004|Quick Release Function|The tool shall be releasable by the robot using only linear motions.|One axis at a time.|
|FLYMAX-TOOL-005|Sustain Vacuum Function|The tool shall maintain vacuum long enough to hold the fly while transferring to the receiving station.|Receiving station vacuum is OK, or the tool can sustain vacuum for extended durations.|
|FLYMAX-TOOL-006|Vacuum Pressure|The tool shall meet its functions with a vacuum level of -.80bar to -0.92bar gauge (80-92% vacuum).||
|FLYMAX-TOOL-007|Ejector Pulse Pressure|The tool shall stay in place when air is passed through it continuously at a maximum pressure of 4.5 bar.||
|FLYMAX-TOOL-008|Maximum Engagement Force|The tool shall require a maximum applied force of 0.245N (25gf) to be secured to the robot.|Due to delta robot motor torque limits.|
|FLYMAX-TOOL-009|Maximum Detachment Force|The tool shall require a maximum applied force of 0.245N (25gf) to be disengaged from the robot.|Due to delta robot motor torque limits.|
|FLYMAX-TOOL-010|Applied Forces During Operation|The tool shall withstand a maximum axial compressive force of 1N (102gf) without damage or buckling.|This may be applied during station height calibration.|
|FLYMAX-TOOL-011|Acceleration Envelope|The tool shall remain in place with simultaneous accelerations of 2g in all 3 axes.|In-plane motions or 3D motions with a 2g acceleration/deceleration profile.|
|FLYMAX-TOOL-012|Mechanical Interface with the Robot|The tool shall attach to a 440C shaft with outer diameter of 6mm -0.012mm/-0.004 mm diameter, and a 2mm thru hole.|An adapter can be accommodated if required.|
|FLYMAX-TOOL-013|Needle Diameter|The tool shall be configurable with blunt end needles between 24 and 26 gauge.|The needles need not be interchangeable, different tools can be made for each gauge.|
|FLYMAX-TOOL-014|Total Length|The total length shall not exceed 50mm.||
|FLYMAX-TOOL-015|Needle Length|The needle length shall be at least 20mm.|To keep bulky things away from the fly for computer vision.|
|FLYMAX-TOOL-016|Needle Tip Runout|The circular runout at the tip of the needle shall be minimized. The desired target is $\leq0.025mm$.|If small enough we may not need to calibrate it out.|
|FLYMAX-TOOL-017|Allowable Play|No measurable play shall exist when fully seated on the robot interface.|A conical interface with preload may be considered.|


# Initial Concept

The image below shows an initial concept of a tool design. This can and should be modified, or completely redone as needed.

Tool Cross Section:
![FlyMAX tool concept](<./FlyMAX Tool Changer Concept.png>)

Tool Attached to Robot:
![Tool attached to the robot](<./FlyMAX Tool Changer on Robot.png>)
