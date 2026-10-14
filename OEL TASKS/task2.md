# Task 2 — Change Requests and Traceability Matrix

## 2.1 Version-Controlled Change Log

| Change ID | Requirement ID | Version | Old Requirement | New Requirement | Reason | Status |
|---|---|---|---|---|---|---|
| CR-01 | SAF-02 / NFR-02 | v1.1 | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | Adds measurable timing and clear triggering conditions. | Modified |
| CR-02 | NFR-01 | v1.1 | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots. | Replaces the unclear word "multiple" with a measurable number. | Modified |
| CR-03 | FR-01 | v1.1 | Only authenticated operators shall be permitted to issue robot commands. | The system shall require authenticated and role-authorised operators before accepting any robot command. | Adds role-based permission checking. | Modified |

## 2.2 Why the Changes Are More Testable

### CR-01
The tester can trigger a critical temperature or battery condition and measure the time until Safe Mode starts. The test passes if Safe Mode starts within 3 seconds.

### CR-02
The tester can connect 20 robots simultaneously and check whether the system maintains connections and handles commands and telemetry correctly.

### CR-03
The tester can try commands from an unauthenticated user, an authenticated user without permission, and an authorised user. The first two commands must be rejected.

## 2.3 Test Cases

| Test ID | Test Scenario | Expected Result | Requirement Link | Risk |
|---|---|---|---|---|
| TC-01 | Submit a command without logging in. | Command is rejected and the attempt is logged. | FR-01, CR-03 | Critical |
| TC-02 | Log in with an account that lacks permission for the command. | Command is rejected and an authorisation event is logged. | FR-01, CR-03 | Critical |
| TC-03 | Submit a valid command using an authorised account. | Command is validated, executed, reported, and logged. | FR-01, FR-02 | High |
| TC-04 | Connect 20 robots simultaneously. | All 20 robots remain connected during the agreed test period without losing required records. | NFR-01, CR-02 | High |
| TC-05 | Trigger a critical battery or temperature condition and measure Safe Mode entry time. | Safe Mode starts within 3 seconds. | SAF-02, CR-01 | Critical |
| TC-06 | Lower battery capacity below the emergency level during navigation. | Delivery is abandoned and the robot returns only if movement is safe. | FR-07, SAF-02 | Critical |
| TC-07 | Detect an obstacle during navigation and then clear it. | Navigation pauses, obstacle handling occurs, and navigation resumes after clearance. | FR-04, FR-05 | Critical |
| TC-08 | Cause chamber temperature to leave its permitted range. | The system corrects temperature and verifies the result through sensor readings. | FR-08, FR-09 | Critical |
| TC-09 | Disconnect the communication link and generate a safety event. | The event is stored locally and synchronised after reconnection. | NFR-05, FR-10 | High |
| TC-10 | Simulate an approaching train while the robot waits before the crossing. | The robot does not enter the crossing. The event is logged. | SAF-01 | Critical |
| TC-11 | Fail a crossing sensor or provide stale sensor data. | Crossing permission is denied and the operator is alerted. | SAF-04 | Critical |
| TC-12 | Open the chamber door during conservation. | Normal conservation and delivery are suspended until safety checks pass. | SAF-03 | Critical |

## 2.4 Multi-Layer Requirements Traceability Matrix (RTM)

| Requirement | State/Transition | Operation | Constraint | Test Case | Risk |
|---|---|---|---|---|---|
| FR-01, CR-03 | TR-01: Command received and checked | OP-01 | C-01 | TC-01, TC-02, TC-03 | Critical |
| FR-02 | TR-01: Command validated | OP-02 | C-01 | TC-03 | High |
| FR-03 | TR-01: IDLE to NAVIGATING | OP-03 | C-02 | TC-03 | High |
| FR-04, FR-05 | TR-02, TR-03: Obstacle handling | OP-04 | C-03 | TC-07 | Critical |
| FR-06 | TR-04: NAVIGATING to DELIVERING | OP-05 | C-04, C-09 | TC-12 | Critical |
| FR-07, CR-01 | TR-07 to TR-10: Critical fault response | OP-06 | C-06 | TC-05, TC-06 | Critical |
| FR-08, FR-09 | Chamber monitoring and correction | OP-08, OP-09, OP-10 | C-10 | TC-08 | Critical |
| FR-10 | Event recording in any state | OP-12 | C-11 | TC-01, TC-05, TC-09 | High |
| NFR-01, CR-02 | 20 robot sessions | OP-13 | C-12 | TC-04 | High |
| NFR-05 | LINK_LOST and recovery | OP-14 | C-08, C-11 | TC-09 | High |
| SAF-01 | Crossing permission check | OP-15 | C-05 | TC-10 | Critical |
| SAF-04 | Crossing sensor/link fault | OP-16 | C-08 | TC-11 | Critical |
| SAF-03 | Delivery inhibited when chamber is unsafe | OP-05, OP-11 | C-04, C-09 | TC-12 | Critical |

## 2.5 Tests to Repeat After Changes

- **CR-01:** Repeat TC-05 and TC-06. Also test critical conditions during NAVIGATING, AVOIDING_OBSTACLE, and DELIVERING.
- **CR-02:** Repeat TC-04 and related connection, command-routing, and telemetry load tests.
- **CR-03:** Repeat TC-01, TC-02, and TC-03. Test every relevant command type and operator role.
