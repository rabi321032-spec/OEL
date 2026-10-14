# Task 1 — Requirement Extraction

## 1.1 Functional Requirements

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| FR-01 | The system shall authenticate the operator and verify role permission before accepting every robot command. | FR | Critical | Security testing |
| FR-02 | The system shall validate command format, target robot, and permitted action before dispatching a command. | FR | High | Functional testing |
| FR-03 | The robot shall start navigation after receiving a valid delivery request. | FR | High | Simulation testing |
| FR-04 | The robot shall suspend navigation and handle an obstacle when one is detected. | FR | Critical | Scenario testing |
| FR-05 | The robot shall resume navigation only after the obstacle is avoided and the route is confirmed safe. | FR | High | State-transition testing |
| FR-06 | The robot shall deliver a batch only after reaching the assigned destination and verifying delivery conditions. | FR | Critical | Functional testing |
| FR-07 | The robot shall abandon delivery and return toward the warehouse when the battery reaches the emergency level, if movement remains safe. | FR | Critical | Battery fault simulation |
| FR-08 | The chamber shall monitor temperature, humidity, vibration, door status, and power for each batch. | FR | Critical | Sensor integration testing |
| FR-09 | The chamber shall verify through fresh sensor readings that temperature or humidity has returned to the permitted range after correction. | FR | Critical | Sensor testing |
| FR-10 | The system shall record commands, status changes, alarms, faults, and delivery events with timestamps. | FR | High | Log inspection |

## 1.2 Non-Functional Requirements

*The following targets are proposed where the scenario does not specify exact values. They should be confirmed with the instructor or system owner.*

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| NFR-01 | The system shall support at least 20 simultaneously connected robots without losing command or telemetry records. | NFR - Scalability | High | Load testing |
| NFR-02 | The robot shall enter Safe Mode within 3 seconds of detecting a critical battery or specified critical temperature condition. | NFR - Performance | Critical | Timed fault-injection testing |
| NFR-03 | Under normal link conditions, the control centre shall display a received status update within 5 seconds of receiving the telemetry message. | NFR - Performance | Medium | Timestamp testing |
| NFR-04 | The system shall reject all tested commands from unauthenticated operators and authenticated operators without the required role permission. | NFR - Security | Critical | Security testing |
| NFR-05 | The system shall retain locally generated safety events during a 30-minute communication outage and synchronise them after the link returns. | NFR - Reliability | High | Link-loss recovery testing |

## 1.3 High-Risk Requirements

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| SAF-01 | The robot shall not enter a railway crossing unless the barrier is confirmed open, no train is approaching or present, and all required safety inputs are valid and current. | Safety | Critical | Railway simulation |
| SAF-02 | The robot shall enter Safe Mode within 3 seconds of detecting a specified critical battery or temperature condition. | Safety | Critical | Timed fault-injection testing |
| SAF-03 | The robot shall not deliver while handling an obstacle, while the chamber door is open, or while chamber protection is active. | Safety | Critical | Integrated safety testing |
| SAF-04 | If railway safety sensors fail or crossing communication is lost or stale, the robot shall stop before the crossing until safety is confirmed. | Safety | Critical | Sensor and communication fault testing |

## 1.4 Verification Methods

- **Inspection:** Review requirements, settings, and logs.
- **Functional testing:** Check actual results against expected results.
- **Simulation:** Reproduce robot and railway scenarios safely.
- **Load testing:** Test the system with 20 connected robots.
- **Fault injection:** Simulate battery, sensor, and communication failures.
