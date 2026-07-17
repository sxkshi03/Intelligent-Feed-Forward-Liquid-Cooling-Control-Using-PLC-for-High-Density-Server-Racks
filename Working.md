# CONTROL PHILOSOPHY

## 1. Objective

This PLC program implements a **feed-forward feedback based liquid cooling control strategy** for a high-density data center comprising six liquid-cooled server racks (R1–R6). The controller continuously evaluates the thermal demand of each rack using real-time IT load and temperature measurements, calculates a Cooling Demand Index (CDI), and coordinates individual Flow Control Valve (FCV) positions together with the Cooling Distribution Unit (CDU) pump speed to maintain safe operating temperatures while minimizing unnecessary cooling energy.

---

## 2. Safety and Permissive Logic

Before cooling control is enabled, the PLC verifies all operating permissives.

The system checks:

- Emergency Stop healthy
- Start command received
- No CDU fault
- Coolant flow above minimum allowable limit
- Temperature setpoints available

Only when all conditions are satisfied is the internal **Safety_OK** bit energized.

This bit acts as the master permissive enabling all subsequent cooling calculations and actuator commands.

---

## 3. Feed-Forward Cooling Demand Calculation

Unlike conventional temperature-only control, this program predicts future cooling requirements using a feed-forward strategy.

Each rack continuously measures:

- Rack IT Load
- Rack Temperature
- Temperature Rate of Rise

These values are normalized and combined to calculate an individual Cooling Demand Index (CDI) for every rack.

The CDI increases immediately when:

- server load increases,
- rack temperature exceeds its setpoint,
- or the rack temperature begins increasing rapidly.

Consequently, additional cooling is requested before excessive temperatures occur.

---

## 4. Rack-Level Cooling Control

Each rack is independently controlled.

The calculated CDI is translated into a staged Flow Control Valve opening.

| Cooling Demand Index | FCV Opening |
| --- | --- |
| Low Demand | 30% |
| Medium Demand | 60% |
| High Demand | 100% |

This staged control philosophy provides:

- stable valve operation,
- reduced oscillation,
- simple commissioning,
- predictable behaviour,
- reduced computational complexity.

---

## 5. System Cooling Demand

After calculating the individual rack CDIs, the controller determines the **highest cooling demand** among all six racks.

The PLC sequentially compares

- CDI_R1
- CDI_R2
- CDI_R3
- CDI_R4
- CDI_R5
- CDI_R6

to determine **Max_CDI**.

The maximum demand is selected to ensure the most thermally stressed rack always receives sufficient coolant flow.

---

## 6. CDU Pump Control

The Cooling Distribution Unit (CDU) pump speed is controlled using the calculated **Max_CDI**.

| Max CDI | Pump Speed |
| --- | --- |
| Low | 40% |
| Medium | 70% |
| High | 100% |

This supervisory strategy automatically increases coolant circulation as overall cooling demand increases.

---

## 7. Continuous Closed-Loop Operation

The PLC executes continuously during every scan cycle.

Each scan performs the following sequence:

1. Read sensor inputs.
2. Recalculate rack temperatures and load conditions.
3. Update Cooling Demand Index.
4. Update FCV positions.
5. Update CDU pump speed.
6. Repeat.

This continuous execution forms a supervisory closed-loop control system in which every new sensor reading influences the next cooling command.

---

# INPUTS

## Digital Inputs

| Tag | Description |
| --- | --- |
| Start_PB | System Start Pushbutton |
| Stop_PB | System Stop Pushbutton |
| Emergency_Stop | Emergency Stop |
| CDU_Fault | CDU Fault Status |
| CRAH_Fault *(optional)* | CRAH Fault Status |

---

## Analog Inputs

### Rack Process Variables

| Tag | Description |
| --- | --- |
| R1_Load – R6_Load | Rack IT Load (%) |
| R1_Temp – R6_Temp | Rack Temperature (°C) |

### Previous Temperature Memory (used for Rate of Rise)

| Tag | Description |
| --- | --- |
| Pr_R1_Temp – Pr_R6_Temp | Previous Scan Temperature |

### Cooling System

| Tag | Description |
| --- | --- |
| Coolant_Flow | Coolant Flow |
| Coolant_Supply_Temp | Supply Coolant Temperature |
| Coolant_Return_Temp | Return Coolant Temperature |

### Room Conditions

| Tag | Description |
| --- | --- |
| Hot_Aisle_Temp | Hot Aisle Temperature |
| Cold_Aisle_Temp | Cold Aisle Temperature |

---

# OUTPUTS

## Analog Outputs

| Tag | Description |
| --- | --- |
| FCV_Position_Cmd_R1 | Rack 1 FCV Opening (%) |
| FCV_Position_Cmd_R2 | Rack 2 FCV Opening (%) |
| FCV_Position_Cmd_R3 | Rack 3 FCV Opening (%) |
| FCV_Position_Cmd_R4 | Rack 4 FCV Opening (%) |
| FCV_Position_Cmd_R5 | Rack 5 FCV Opening (%) |
| FCV_Position_Cmd_R6 | Rack 6 FCV Opening (%) |
| CDU_Pump_OUT | CDU Pump Speed Command (%) |

---

## Digital Outputs

| Tag | Description |
| --- | --- |
| Feedback_Loop | Supervisory Feedback Status (optional) |

---

# Internal Memory Tags

## System Status

| Tag | Description |
| --- | --- |
| Start_Running | System Running Latch |
| Safety_OK | Master Permissive |

---

## Setpoints

| Tag | Description |
| --- | --- |
| R_Temp_Setpoint | Rack Temperature Setpoint |
| Max_Temp_TR | Maximum Rack Temperature |
| Max_Temp_A | Maximum Allowable Temperature |
| Min_Coolant_Flow | Minimum Coolant Flow |

---

## Normalized Variables

| Tag | Description |
| --- | --- |
| Norm_R1_Temp – Norm_R6_Temp | Normalized Temperature Error |
| Norm_R1_Rate – Norm_R6_Rate | Normalized Temperature Rate |

---

## Cooling Demand

| Tag | Description |
| --- | --- |
| CDI_R1 – CDI_R6 | Cooling Demand Index for each rack |
| Max_CDI | Highest Cooling Demand Index |

---

## Intermediate Variables

| Tag | Description |
| --- | --- |
| Temp_Error_R1 – Temp_Error_R6 | Temperature Error |
| Rate_R1 – Rate_R6 | Temperature Rate of Rise |
| FCV_Open_% | Intermediate FCV command before output |

## Values Assigned

| Parameter | Value | Purpose |
| --- | --- | --- |
| Rack Temperature Setpoint | **27°C** | Normal operating target |
| Cooling Increase Band | **28–32°C** | Increase cooling proportionally |
| High Temperature Alarm | **35°C** | Warn operator, maximize cooling |
| High Temperature Trip | **40°C** | Safety trip / emergency shutdown |

| PLC Tag | Description | Unit | Typical Range | Purpose |
| --- | --- | --- | --- | --- |
| AI_R1_Load | Rack 1 IT Load | % | 0–100% | Feedforward |
| AI_R2_Load | Rack 2 IT Load | % | 0–100% | Feedforward |
| AI_R3_Load | Rack 3 IT Load | % | 0–100% | Feedforward |
| AI_R4_Load | Rack 4 IT Load | % | 0–100% | Feedforward |
| AI_R5_Load | Rack 5 IT Load | % | 0–100% | Feedforward |
| AI_R6_Load | Rack 6 IT Load | % | 0–100% | Feedforward |
| AI_R1_Temp | Rack 1 Outlet Temperature | °C | 20–45°C | Feedback |
| AI_R2_Temp | Rack 2 Outlet Temperature | °C | 20–45°C | Feedback |
| AI_R3_Temp | Rack 3 Outlet Temperature | °C | 20–45°C | Feedback |
| AI_R4_Temp | Rack 4 Outlet Temperature | °C | 20–45°C | Feedback |
| AI_R5_Temp | Rack 5 Outlet Temperature | °C | 20–45°C | Feedback |
| AI_R6_Temp | Rack 6 Outlet Temperature | °C | 20–45°C | Feedback |
| AI_Hot_Aisle_Temp | Hot Aisle Temperature | °C | 25–45°C | CRAH Control |
| AI_Cold_Aisle_Temp | Cold Aisle Temperature | °C | 18–30°C | CRAH Control |
| AI_Coolant_Flow | CDU Coolant Flow Rate | L/min | 20–200 L/min | CDU Control |
| AI_Coolant_Pressure | CDU Supply Pressure | bar | 1–5 bar | Pump Monitoring |
| AI_Coolant_Supply_Temp | Coolant Supply Temperature | °C | 18–30°C | Cooling Performance |
| AI_Coolant_Return_Temp | Coolant Return Temperature | °C | 22–40°C | Heat Load Calculation |

| Parameter | Setpoint |
| --- | --- |
| Rack Temperature | **27°C** |
| Hot Aisle Temperature | **35°C** |
| Cold Aisle Temperature | **22°C** |
| Coolant Supply Temperature | **20°C** |
| Minimum Coolant Flow | **40 L/min** |
| Coolant Pressure | **2.5 bar** |

| Analog Input | Initial Value | Unit |
| --- | --- | --- |
| Rack 1 Load | 35 | % |
| Rack 2 Load | 60 | % |
| Rack 3 Load | 80 | % |
| Rack 4 Load | 45 | % |
| Rack 5 Load | 95 | % |
| Rack 6 Load | 25 | % |
| Rack 1 Temperature | 27 | °C |
| Rack 2 Temperature | 29 | °C |
| Rack 3 Temperature | 32 | °C |
| Rack 4 Temperature | 28 | °C |
| Rack 5 Temperature | 35 | °C |
| Rack 6 Temperature | 26 | °C |
| Cold Aisle Temperature | 22 | °C |
| Hot Aisle Temperature | 35 | °C |
| Coolant Flow | 100 | L/min |
| Coolant Pressure | 2.5 | bar |
| Coolant Supply Temperature | 20 | °C |
| Coolant Return Temperature | 28 | °C |

| Rack | Current Temp Input (AI) | Previous Temp Memory | Calculated Rate |
| --- | --- | --- | --- |
| Rack 1 | 27°C | 26.5°C | +0.5°C/s |
| Rack 2 | 29°C | 28.2°C | +0.8°C/s |
| Rack 3 | 32°C | 31°C | +1°C/s |
| Rack 4 | 28°C | 27.7°C | +0.3°C/s |
| Rack 5 | 35°C | 33.5°C | +1.5°C/s |
| Rack 6 | 26°C | 25.8°C | +0.2°C/s |
