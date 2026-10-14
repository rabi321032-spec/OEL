# Task 5 — Safety Constraints, Formalisation, and Violations

## 5.1 Safety Constraints in Simple English

| Constraint ID | Constraint | Why It Is Necessary |
|---|---|---|
| C-01 | A robot command shall execute only when the operator is authenticated and authorised. | Prevents unauthorised control. |
| C-02 | The robot shall not deliver directly from IDLE. | Prevents skipping navigation and destination checks. |
| C-03 | The robot shall not deliver while handling an obstacle. | Prevents unsafe delivery behaviour. |
| C-04 | The robot shall not deliver when the chamber door is open or protection mode is active. | Protects the medicine. |
| C-05 | The robot shall enter a railway crossing only when the barrier is confirmed open, no train is approaching or present, and all safety inputs are valid. | Prevents railway collisions. |
| C-06 | Safe Mode shall be entered within 3 seconds of detecting a specified critical battery or temperature condition. | Reduces exposure to critical conditions. |
| C-07 | The crossing barrier shall not open while a train is approaching or present. | Prevents unsafe crossing access. |
| C-08 | Failed, stale, or unavailable crossing sensors or communication shall prevent crossing permission. | Unknown conditions must not be treated as safe. |
| C-09 | If the chamber door opens, normal conservation shall remain suspended until the door is closed and conditions are verified. | Protects temperature-sensitive medicines. |
| C-10 | Temperature or humidity correction shall not be declared successful until fresh sensor readings confirm the permitted range. | Prevents false confirmation of successful correction. |
| C-11 | Safety events shall be stored locally during communication loss and synchronised after recovery. | Preserves evidence of incidents. |
| C-12 | The system shall support at least 20 simultaneously connected robots under agreed load-test conditions. | Meets the pilot scalability target. |

## 5.2 Formalisation of Constraints

**Symbols:**
- `∧` means AND.
- `∨` means OR.
- `¬` means NOT.
- `→` means IMPLIES.

### C-01: Command Authorisation

Command execution requires authentication and role permission.

`Command_Execute → (Authenticated ∧ Role_Authorised)`

### C-02: No Delivery Directly from IDLE

`State_IDLE → ¬Delivery_Active`

### C-03: No Delivery During Obstacle Handling

`State_AVOIDING → ¬Delivery_Active`

### C-04: Chamber Door and Protection Interlock

`(Door_Open ∨ Protection_Active) → ¬Delivery_Active`

### C-05: Safe Railway Crossing

`Robot_Enters_Crossing → (Barrier_Open ∧ ¬Train_Approaching ∧ ¬Train_Present ∧ Inputs_Valid)`

### C-06: Safe Mode Timing

`Critical_Condition → (SafeMode_EntryTime - DetectionTime ≤ 3 seconds)`

### C-07: Barrier Safety

`(Train_Approaching ∨ Train_Present) → ¬Barrier_Open`

### C-08: Sensor and Communication Failure

`¬Inputs_Valid → (¬Crossing_Clear_Confirmed ∧ ¬Robot_Entry_Permitted)`

### C-09: Door-Open Conservation Rule

`Door_Open → ¬Conservation_Active`

### C-10: Verified Environmental Correction

`Correction_Successful → (Temperature_In_Range ∧ Humidity_In_Range)`

### C-11: Local Event Storage

`(Link_Lost ∧ Safety_Event) → Event_Stored_Locally`

### C-12: Robot Capacity

`Connected_Robots ≥ 20`

This capacity condition must hold during the defined load test while required system services remain functional.

## 5.3 Violation Scenarios and Expected Responses

| Constraint | Violation Scenario | Detection Method | Expected Response |
|---|---|---|---|
| C-01 | `Command_Execute=TRUE`, `Authenticated=FALSE`, `Role_Authorised=FALSE` | Command gateway checks authentication and role. | Reject command, log attempt, and prevent robot action. |
| C-02 | `State_IDLE=TRUE`, `Delivery_Active=TRUE` | State monitor detects delivery during IDLE. | Block delivery and log a state violation. |
| C-03 | `State_AVOIDING=TRUE`, `Delivery_Active=TRUE` | Robot state monitor checks obstacle and delivery flags. | Inhibit delivery and continue safe obstacle handling. |
| C-04 | `Door_Open=TRUE`, `Protection_Active=FALSE`, `Delivery_Active=TRUE` | Door sensor and delivery interlock detect the conflict. | Suspend delivery and alert the operator. |
| C-05 | `Robot_Enters_Crossing=TRUE`, `Barrier_Open=FALSE`, `Train_Approaching=TRUE`, `Inputs_Valid=TRUE` | Crossing monitor checks entry permission. | Deny entry, stop robot before crossing, alert operator, and log the event. |
| C-06 | `Critical_Condition=TRUE`, `DetectionTime=10.0s`, `SafeMode_EntryTime=13.8s` | Timer calculates 3.8 seconds elapsed. | Mark the timing test failed, raise an alarm, and log the incident. |
| C-07 | `Train_Present=TRUE`, `Barrier_Open=TRUE` | Railway monitor compares train detection with barrier status. | Trigger alarm, command safe barrier closure where possible, alert operator, and log fault. |
| C-08 | `Inputs_Valid=FALSE`, `Crossing_Clear_Confirmed=TRUE`, `Robot_Entry_Permitted=TRUE` | Sensor health and communication monitor detect invalid data. | Revoke crossing permission, stop robot before entry, and alert operator. |
| C-09 | `Door_Open=TRUE`, `Conservation_Active=TRUE` | Door interlock checks door status and conservation mode. | Suspend conservation, verify conditions before resuming, and log event. |
| C-10 | `Correction_Successful=TRUE`, `Temperature_In_Range=FALSE`, `Humidity_In_Range=TRUE` | Fresh sensor readings reveal temperature is still out of range. | Reject success status, continue recovery if safe, or activate protection and alert. |
| C-11 | `Link_Lost=TRUE`, `Safety_Event=TRUE`, `Event_Stored_Locally=FALSE` | Local event-storage monitor detects a missing record. | Raise a logging fault, preserve available diagnostic data, and alert after recovery. |
| C-12 | `Connected_Robots=18` during the required 20-robot test | Load test measures active connections. | Mark capacity test failed and record performance results. |

## 5.4 Additional Safety Notes

- A violation scenario describes an unsafe condition that the system must detect, not a condition that should be allowed.
- Failed, stale, unknown, or contradictory railway sensor values must be treated as unsafe.
- Immediate safety actions should not depend only on delayed remote commands.
- The system owner must define the exact critical temperature, emergency battery level, crossing communication timeout, and sensor data freshness limits.
- Safety incidents should be timestamped and recorded for later investigation.
