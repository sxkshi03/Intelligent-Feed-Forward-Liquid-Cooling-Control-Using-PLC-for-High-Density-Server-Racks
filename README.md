# Intelligent-Feed-Forward-Liquid-Cooling-Control-Using-PLC-for-High-Density-Server-Racks

## Objective
To develop a PLC-based feed-forward liquid cooling control system that optimizes rack cooling and CDU pump operation using a Cooling Demand Index (CDI).

## Project Files

### Ladder Logic Screenshots
- [Ladder_Logic_1.png](Ladder_Logic_1.png)
- [Ladder_Logic_2.png](Ladder_Logic_2.png)
- [Ladder_Logic_3.png](Ladder_Logic_3.png)
- [Ladder_Logic_4.png](Ladder_Logic_4.png)
- [Ladder_Logic_5.png](Ladder_Logic_5.png)
- [Ladder_Logic_6.png](Ladder_Logic_6.png)


### Demonstration Video
- [Ladder_Logic_Simulation.mp4](Ladder_Logic_Simulation.mp4)

### Control Phillosphy 
- [Working.md](Working.md)

## Features

- Feed-forward cooling demand calculation
- Cooling Demand Index (CDI) based control
- Multi-rack thermal monitoring (6 server racks)
- Rack-wise Flow Control Valve (FCV) control
- Highest-demand (Max CDI) pump control
- Staged cooling response (30%, 60%, 100%)
- Temperature rate-of-rise prediction
- Safety permissives and fault interlocks
- Continuous supervisory cooling control
- CDU pump speed optimization

## PLC Instructions Used

- XIC (Examine If Closed)
- XIO (Examine If Open)
- OTE (Output Energize)
- MOV (Move)
- ADD
- SUB
- MUL
- DIV
- LIM (Limit)
- GEQ / LEQ / GRT (Compare Instructions)

### Note

A PID controller was not implemented because the PLC development environment did not provide a built-in PID instruction. A staged feed-forward control strategy using CDI and compare logic was implemented instead.
