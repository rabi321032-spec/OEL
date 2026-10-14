# Task 3 — State Modelling and Transition Verification

## 3.1 Robot States

| State ID | State | Meaning | Entry Condition | Exit Condition |
|---|---|---|---|---|
| ST-01 | IDLE | Robot waits for a delivery request. | Startup checks pass or warehouse arrival is confirmed. | A valid delivery request is accepted. |
| ST-02 | NAVIGATING | Robot travels toward the clinic. | A valid delivery request is accepted or an obstacle is cleared. | Destination reached, obstacle detected, or a critical event occurs. |
| ST-03 | AVOIDING_OBSTACLE | Robot handles an obstacle while navigation is suspended. | An obstacle is detected during navigation. | Obstacle is cleared and the route is safe, or a critical fault occurs. |
| ST-04 | DELIVERING | Robot performs the medicine handover. | Destination and delivery conditions are verified. | Delivery completes or a critical fault requires suspension. |
| ST-05 | RETURNING | Robot travels toward the warehouse. | Delivery completes or delivery is abandoned and safe return is possible. | Warehouse arrival is confirmed or movement becomes unsafe. |
| ST-06 | SAFE_MODE | Protective behaviour is active and unsafe actions are blocked. | Critical battery, temperature, or fault condition is detected. | Fault is cleared, checks pass, and recovery is authorised. |
| ST-07 | LINK_LOST | Remote communication is unavailable or timed out. | Communication timeout is detected. | Link is restored and verified, or Safe Mode is activated. |

**Safety rule:** Safe Mode takes priority over delivery progress. If the communication link is lost during a critical fault, the robot must still apply local protective behaviour.

## 3.2 Events and Conditions

| Event ID | Event | Meaning |
|---|---|---|
| EV-01 | PowerOnSelfCheckPassed | Startup checks are complete. |
| EV-02 | CommandReceived | A command arrives from Mission Control. |
| EV-03 | DeliveryRequest | A valid delivery request is received. |
| EV-04 | ObstacleDetected | An obstacle is detected on the route. |
| EV-05 | ObstacleCleared | The obstacle is avoided and the route is confirmed safe. |
| EV-06 | DestinationReached | The robot reaches the assigned clinic. |
| EV-07 | DeliveryCompleted | Medicine handover is complete. |
| EV-08 | WarehouseReached | The robot reaches the warehouse. |
| EV-09 | CriticalBattery | Battery capacity falls below the emergency level. |
| EV-10 | CriticalTemperature | Battery or chamber temperature exceeds its critical threshold. |
| EV-11 | CriticalFault | A critical system fault is detected. |
| EV-12 | LinkTimeout | No valid communication arrives within the configured timeout. |
| EV-13 | LinkRestored | Communication returns and is verified. |
| EV-14 | RecoveryAuthorised | Fault checks pass and recovery is approved. |
| EV-15 | DoorOpen | Chamber door opens during conservation or delivery. |

A **state** describes what the robot is doing. An **event** is something that happens and may cause a state change.

## 3.3 State Transition Table

| Transition ID | Current State | Event / Condition | Next State | Requirement | Valid? | Verification Evidence |
|---|---|---|---|---|---|---|
| TR-01 | Any command-ready state | CommandReceived; authentication and role permission pass | Command may proceed to validation/execution | FR-01 | Yes, if authorised | Security test and command log |
| TR-02 | IDLE | Valid DeliveryRequest | NAVIGATING | FR-03 | Yes | Robot starts approved route |
| TR-03 | NAVIGATING | ObstacleDetected | AVOIDING_OBSTACLE | FR-04 | Yes | Navigation pauses |
| TR-04 | AVOIDING_OBSTACLE | ObstacleCleared and route safe | NAVIGATING | FR-05 | Yes | Route-clear evidence and state log |
| TR-05 | NAVIGATING | DestinationReached and delivery conditions safe | DELIVERING | FR-06 | Yes | Destination and chamber checks pass |
| TR-06 | DELIVERING | DeliveryCompleted | RETURNING | FR-06 | Yes | Handover is logged |
| TR-07 | RETURNING | WarehouseReached | IDLE | FR-03 | Yes | Arrival is logged |
| TR-08 | NAVIGATING | CriticalBattery and safe return possible | RETURNING | FR-07 | Yes, with safety guard | Battery event and return decision logged |
| TR-09 | NAVIGATING | CriticalTemperature or critical fault | SAFE_MODE | SAF-02 | Yes | Timed fault test |
| TR-10 | AVOIDING_OBSTACLE | CriticalBattery, CriticalTemperature, or CriticalFault | SAFE_MODE | SAF-02 | Yes | Fault-injection evidence |
| TR-11 | DELIVERING | CriticalBattery, CriticalTemperature, or CriticalFault | SAFE_MODE | SAF-02, SAF-03 | Yes | Delivery is inhibited |
| TR-12 | IDLE | LinkTimeout | LINK_LOST | NFR-05 | Yes | Link-loss log |
| TR-13 | NAVIGATING | LinkTimeout | LINK_LOST | NFR-05 | Yes, with local safety policy | Robot follows safe link-loss policy |
| TR-14 | LINK_LOST | LinkRestored and checks pass | Previous safe state or IDLE | NFR-05 | Conditional | Link and event synchronisation logs |
| TR-15 | SAFE_MODE | RecoveryAuthorised and fault cleared | Safe recovery state | SAF-02 | Conditional | Sensor checks and recovery record |
| TR-16 | IDLE | Direct request to start delivery without navigation | DELIVERING | FR-06 | No | Negative transition test |
| TR-17 | AVOIDING_OBSTACLE | Direct request to deliver | DELIVERING | FR-05, FR-06 | No | Negative transition test |
| TR-18 | Any active state | Critical event while link is lost | SAFE_MODE while retaining link-loss information | SAF-02, NFR-05 | Yes | Local fault-injection test |
| TR-19 | DELIVERING | DoorOpen or chamber protection active | Protected delivery suspension / SAFE_MODE | SAF-03 | Yes | Integrated chamber/robot test |

## 3.4 Verification Investigations

### (a) Can IDLE → DELIVERING happen?
No. The robot must receive a valid request, navigate to the destination, and verify delivery conditions first. A direct transition violates FR-06.

### (b) What if AVOIDING_OBSTACLE has no transition back?
The robot may remain stuck indefinitely and fail to complete the delivery. This is a liveness problem. TR-04 is required. A timeout or fault path is also needed if the obstacle cannot be avoided safely.

### (c) Can AVOIDING_OBSTACLE → DELIVERING occur?
No, not in the normal workflow. The robot must first return to NAVIGATING after confirming the obstacle is cleared. It may deliver only after reaching the destination.

### (d) What happens on CriticalBattery during AVOIDING_OBSTACLE or DELIVERING?
Without a defined transition, the robot might continue an unsafe action. TR-10 and TR-11 address this problem by entering Safe Mode. A return to the warehouse is allowed only if movement is confirmed safe.

## 3.5 Additional Verification Findings

- **Invalid transition:** IDLE → DELIVERING.
- **Invalid transition:** AVOIDING_OBSTACLE → DELIVERING.
- **Missing-transition risk:** Critical faults and communication loss must be handled from every relevant active state.
- **Deadlock risk:** Obstacle avoidance needs a safe exit or escalation path.
- **Safety priority:** Critical events override normal delivery progress.
- **Link-loss protection:** Immediate safety decisions must not depend on delayed remote commands.
