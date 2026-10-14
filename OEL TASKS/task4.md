# Task 4 — Operation Identification and Operation Schemas

## 4.1 List of Operations

| Operation ID | Operation Name | Purpose |
|---|---|---|
| OP-01 | Validate Operator and Command | Checks operator identity, role, and command permission. |
| OP-02 | Validate Command Payload | Checks command format, target robot, and allowed action. |
| OP-03 | Start Route Navigation | Loads the approved route and starts movement. |
| OP-04 | Detect and Handle Obstacle | Detects obstacles and plans safe avoidance. |
| OP-05 | Authorise Batch Delivery | Checks destination, chamber, door, and protection conditions. |
| OP-06 | Evaluate Critical Battery and Enter Safe Mode | Detects critical conditions and activates protection. |
| OP-07 | Run Chamber Self-Check | Checks essential chamber sensors at power-on. |
| OP-08 | Correct Temperature | Adjusts temperature toward the permitted range. |
| OP-09 | Correct Humidity | Adjusts humidity toward the permitted range. |
| OP-10 | Verify Chamber Conditions | Checks actual sensor readings against the batch profile. |
| OP-11 | Handle Door Opening | Suspends normal conservation when the door opens. |
| OP-12 | Record Event | Stores timestamped commands, alarms, faults, and state changes. |
| OP-13 | Manage Robot Connections | Manages simultaneous robot connections. |
| OP-14 | Queue and Synchronise Events | Stores events during link loss and synchronises them later. |
| OP-15 | Evaluate Crossing Permission | Checks whether railway crossing entry is safe. |
| OP-16 | Handle Crossing Fault | Stops the robot before the crossing and reports faults. |
| OP-17 | Manage Vibration Response | Suspends risky activities during significant vibration. |
| OP-18 | Switch to Emergency Power | Uses backup power if main power fails and backup is available. |
| OP-19 | Activate Protection Response | Reduces light exposure, alerts the operator, and blocks unsafe removal. |
| OP-20 | Authorise Batch Removal | Allows removal only when the chamber is safe. |

## 4.2 Operation Schemas

### OP-07 — Run Chamber Self-Check
- **Description:** Checks essential sensors before monitoring begins.
- **Inputs:** Power-on signal and sensor status values.
- **Outputs:** PASS or FAIL result.
- **Reads:** Temperature, humidity, vibration, door, and power sensor diagnostics.
- **Changes:** Sets the chamber readiness flag.
- **Sends/Alerts:** Reports failed sensors to Mission Control.
- **Assumes (precondition):** The chamber controller has power and can query sensors.
- **Result (postcondition):** Monitoring starts only if all essential sensors pass. Otherwise, the chamber is not declared ready and a fault is recorded.

### OP-08 — Correct Temperature
- **Description:** Adjusts chamber temperature to the permitted batch range.
- **Inputs:** Current temperature, minimum and maximum limits, recovery timeout.
- **Outputs:** Heating/cooling command and correction result.
- **Reads:** Temperature sensor, batch profile, and controller status.
- **Changes:** Heating/cooling control setting and correction timer.
- **Sends/Alerts:** Alerts the operator if correction fails.
- **Assumes (precondition):** The batch profile is loaded and the sensor is valid.
- **Result (postcondition):** A fresh sensor reading confirms temperature is in range, or protection mode is activated after failure or timeout.

### OP-09 — Correct Humidity
- **Description:** Adjusts humidity to the permitted batch range.
- **Inputs:** Current humidity, allowed humidity range, recovery timeout.
- **Outputs:** Humidity-control command and correction result.
- **Reads:** Humidity sensor, batch profile, and actuator status.
- **Changes:** Humidity-control setting and correction timer.
- **Sends/Alerts:** Alerts the operator if humidity remains outside the permitted range.
- **Assumes (precondition):** The batch profile is loaded and the humidity sensor is valid.
- **Result (postcondition):** Fresh sensor readings confirm humidity is in range, or protection mode is activated after failure.

### OP-10 — Verify Chamber Conditions
- **Description:** Checks whether chamber conditions are safe for the medicine.
- **Inputs:** Batch profile and current sensor samples.
- **Outputs:** CONDITIONS_SAFE or CONDITIONS_UNSAFE.
- **Reads:** Temperature, humidity, vibration, door status, and sensor-health flags.
- **Changes:** Updates the chamber safety-verification flag.
- **Sends/Alerts:** Alerts the operator if readings are unsafe or invalid.
- **Assumes (precondition):** The batch profile exists and sensor readings have timestamps.
- **Result (postcondition):** Conditions are marked safe only when all required readings are valid, current, and within permitted limits.

### OP-11 — Handle Door Opening
- **Description:** Suspends normal conservation if the chamber door opens.
- **Inputs:** Door-open event and current chamber mode.
- **Outputs:** Door-event result and suspension status.
- **Reads:** Door sensor, batch profile, environmental readings, and sensor health.
- **Changes:** Suspends normal conservation and clears previous safety verification.
- **Sends/Alerts:** Records the opening and alerts the operator when required.
- **Assumes (precondition):** The door sensor is operational.
- **Result (postcondition):** Normal conservation and delivery remain suspended until the door is closed and chamber conditions are re-verified.

### OP-19 — Activate Protection Response
- **Description:** Protects the medicine when correction fails or a critical chamber fault occurs.
- **Inputs:** Failed correction, critical fault, or recovery timeout.
- **Outputs:** Protection status.
- **Reads:** Fault code, environmental readings, and actuator status.
- **Changes:** Activates protection mode, reduces light exposure, and blocks batch removal.
- **Sends/Alerts:** Sends an urgent operator alert and records the fault.
- **Assumes (precondition):** The protection controller is available or a defined fail-safe response exists.
- **Result (postcondition):** Protection is active, the incident is logged, and batch removal remains blocked until safe recovery.

### OP-14 — Queue and Synchronise Events
- **Description:** Stores events during communication loss and sends them after recovery.
- **Inputs:** Local event records and link status.
- **Outputs:** Synchronisation result.
- **Reads:** Link health, event queue, and sequence numbers.
- **Changes:** Marks events as pending or synchronised.
- **Sends/Alerts:** Reports synchronisation success or failure.
- **Assumes (precondition):** Local event storage is available.
- **Result (postcondition):** Events are synchronised without silent loss. Unsent events remain queued and generate an alert.

### OP-15 — Evaluate Crossing Permission
- **Description:** Determines whether the robot may enter the railway crossing.
- **Inputs:** Barrier state, train detection, sensor health, data freshness, and link status.
- **Outputs:** PERMIT_ENTRY or DENY_ENTRY.
- **Reads:** Required crossing signals and their timestamps.
- **Changes:** Updates the crossing-permission decision.
- **Sends/Alerts:** Reports denied entry or faults to Mission Control.
- **Assumes (precondition):** Crossing signals are available for evaluation.
- **Result (postcondition):** Entry is permitted only if the barrier is confirmed open, no train is approaching or present, and all required inputs are valid and current.

## 4.3 Operations That Change the State Model

1. **OP-06 — Evaluate Critical Battery and Enter Safe Mode:** A critical condition can move NAVIGATING, AVOIDING_OBSTACLE, or DELIVERING to SAFE_MODE. See TR-09, TR-10, and TR-11 in Task 3.
2. **OP-11 — Handle Door Opening:** Opening the door suspends normal conservation and delivery until the door is closed and the conditions are verified. See TR-19.
3. **OP-14 — Queue and Synchronise Events:** A communication timeout moves the system to LINK_LOST, while verified recovery permits a controlled transition. See TR-12 to TR-14.

## 4.4 Operations That Require Sensor Verification

- **OP-08 Correct Temperature:** Sending a cooling or heating command does not prove that the temperature reached the required range. A new sensor reading is necessary.
- **OP-09 Correct Humidity:** Sending a humidity-control command does not prove that humidity is safe. The system must read the sensor again.

If correction fails within the recovery period, the chamber must activate protection, reduce light exposure, alert the operator, and prevent unsafe batch removal.
