---
layout: default
title: Hot Wheels Launcher
---

# Hot Wheels Launch System Overview

## State Machine Summary

The device operates using a 5-state control loop managed by a finite state machine. The states are as follows:

### **State 4 – Reset**
- On boot, servos are reset to their default positions using 50 Hz PWM signals.
- Timer0 is used for all PWM output, toggling between pins depending on which servo is active.
- After resetting, the system transitions to **State 0**.

### **State 0 – Wait for Placement**
- Uses a photoresistor (via ADC) to detect whether the car is placed on the track.
- Once detected, transitions to **State 1**.

### **State 1 – Centering Based on Ramp Data**
- The ATMega acts as an SPI slave, receiving ramp data from the Raspberry Pi:
  - **1 byte**: header
  - **1 byte**: horizontal center (x)
  - **1 byte**: ramp height (pixels)
- Data is processed to align the servo so the ramp is centered in view.
- The Raspberry Pi:
  - Captures images using `libcamera`
  - Analyzes blue pixel data with `OpenCV`
  - Computes rectangle from cornermost blue pixels to determine center and height
- Once alignment is within a set threshold, transitions to **State 2**.

### **State 2 – Distance and Launch Power Calculation**
- Ultrasonic sensor measures distance to the ramp.
- With known camera FOV (62°) and ramp pixel height, ramp height is estimated.
- Uses this to:
  - Compute necessary motor speed (PWM duty cycle)
  - Set takeoff ramp servo angle
- Transitions to **State 3**.

### **State 3 – Launch**
- Motors are spun up using two parallel BJTs and the previously computed PWM signal.
- After a 0.5s delay, the launch servo releases the car.
- Foam and rubber components provide grip and cushioning during acceleration.
- Motors are shut off immediately after launch.
- System returns to **State 4** to reset and wait for the next cycle.

[Back](../index.html)

