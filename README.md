# Apple Juice Production Line PLC Simulation

This project is a PLC-controlled apple juice production line simulation created with **Siemens TIA Portal** and **Factory I/O**.

The system simulates an automatic production process where water and apple ingredients are filled into separate tanks, processed into apple juice, bottled in 1-liter units, and moved by a conveyor. The PLC logic is implemented using an SCL-based state machine.

## Technologies Used

- Siemens TIA Portal
- SCL / Structured Control Language
- Factory I/O
- PLC state-machine logic
- Digital inputs and outputs
- Analog tank level and valve control

## Project Goal

The goal of this project is to simulate a small automated apple juice production process.

The user can enter the desired apple juice amount, start the process, and the PLC automatically handles:

- filling the water and apple tanks
- calculating the required ingredient ratio
- filling the apple juice tank
- bottling the juice in 1-liter portions
- moving bottles or boxes using a conveyor
- stopping and draining the system
- emergency stop and reset behavior

## Process Overview

The production process follows these steps:

1. The system waits for the user to enter an apple juice amount.
2. The user presses the Start button.
3. The PLC checks if the entered value is valid.
4. Water and apple tanks are filled.
5. The required water and apple amount is calculated.
6. Apple is added first into the apple juice tank.
7. Water is added after the apple.
8. Apple juice is bottled in 1-liter units.
9. The conveyor moves the bottle or box after each filling cycle.
10. The process continues until Stop is requested or the ingredients must be refilled.
11. When Stop is pressed, the system finishes the current cycle and drains all tanks.
12. Emergency Stop immediately stops all valves and the conveyor.

## Recipe Formula

The apple juice recipe is calculated using percentage values:

```text
waterNeeded = requestedJuiceL * WATER_PERCENT / 100
appleNeeded = requestedJuiceL * APPLE_PERCENT / 100
```

Example:

```text
requestedJuiceL = 5 L
WATER_PERCENT = 30%
APPLE_PERCENT = 70%

waterNeeded = 1.5 L
appleNeeded = 3.5 L
```

## Main Features

### Automatic Tank Filling

The system fills the water and apple tanks until they reach the maximum charge level.

```text
Water tank full -> water charge valve closes
Apple tank full -> apple charge valve closes
```

### Ingredient Ratio Control

The PLC calculates how much water and apple is needed for the requested amount of juice.

The system then discharges only the calculated amount from each tank.

### Apple-First Filling Logic

The process fills the apple juice tank in two steps:

```text
1. Apple is discharged first
2. Water is discharged second
```

This makes the production sequence easier to understand and control.

### Bottling Logic

Each bottle or box represents **1 liter** of apple juice.

For every bottle:

```text
1. Apple juice discharge valve opens
2. Apple juice level decreases by 1 liter
3. Valve closes
4. Conveyor starts
5. Bottle or box leaves the filling position
6. Bottle counter increases
```

### Conveyor Control

The conveyor moves the filled bottle or box away from the filling station.

A Factory I/O sensor detects when the bottle or box leaves the position, then the conveyor stops.

### Bottle Counter

The PLC counts every completed bottle or box and displays the current produced amount on the HMI value.

```text
1 bottle = 1 liter
```

### Stop Button Logic

The Stop button does not stop the process immediately.

Instead, it requests a controlled stop:

```text
Stop pressed
-> finish current bottling cycle
-> drain all tanks
-> reset process values
-> return to waiting state
```

This is different from Emergency Stop.

### Emergency Stop Logic

The Emergency Stop button immediately stops the process.

When Emergency Stop is pressed:

```text
All charge valves close
All discharge valves close
Conveyor stops
Red light blinks
Reset light turns on
Current state is saved
```

After the emergency button is released, the user must press Reset.

The Reset button resumes the process from the saved state or from the correct matching start state.

## Control Buttons

| Button | Function |
|---|---|
| Start | Starts the production process if the entered value is valid |
| Stop | Requests a controlled stop after the current cycle |
| Reset | Resumes the process after Emergency Stop |
| Emergency Stop | Immediately stops all valves and conveyor |

## Signal Lights

| Light | Meaning |
|---|---|
| Green | Production is running |
| Yellow | Waiting, ready, or process indication |
| Red | Error, invalid input, stop drain, or emergency warning |

For emergency warning, the red light uses a blinking signal:

```text
Clock_0.5Hz = 1 second ON, 1 second OFF
```

## Important States

The project is built around a state machine.

Main states include:

```text
STATE_INIT
STATE_WAIT_FOR_START_AND_SAVE_APPLE_JUICE_VALUE
STATE_LIGHT_UP_RED_LIGHT
STATE_FILL_APPLE_AND_WATER_TANK
STATE_WAIT_FOR_FILL_UP
STATE_CALCULATE_JUICE_CONSTANTS
STATE_START_FILL_APPLE_JUICE_TANK_WITH_APPLE
STATE_WAIT_APLE_JUICE_FILL_UP_APPLE
STATE_START_FILL_APPLE_JUICE_TANK_WITH_WATER
STATE_WAIT_APLE_JUICE_FILL_UP_WATER
STATE_CALCULATE_BOTTLE_VALUE
STATE_START_FILL_BOTTLE
STATE_WAIT_FILL_BOTTLE
STATE_START_BOTTLE_CONVEYER
STATE_STOP_BOTTLE_CONVEYER
STATE_CHECK_FOR_WATER_AND_APPLE
STATE_START_DRAIN_ALL_TANKS
STATE_WAIT_DRAIN_ALL_TANKS
```

## Factory I/O Usage

Factory I/O is used as the visual simulation environment.

The simulation includes:

- water tank
- apple tank
- apple juice tank
- analog level meters
- analog valves
- conveyor
- bottle or box sensor
- start, stop, reset, and emergency controls
- signal lights

The analog valves use a `0.0 - 10.0` value range:

```text
0.0 = closed
10.0 = fully open
```

## Input Validation

The system does not start if the requested apple juice amount is zero or invalid.

If the user presses Start with a zero value:

```text
Production does not start
Red light turns on for a short warning period
System returns to waiting state
```

## Safety Behavior

The emergency system has priority over the normal process.

If Emergency Stop is active, the PLC disables every actuator and prevents the normal state machine from continuing.

The process can only resume after:

```text
1. Emergency button is released
2. Reset button is pressed
```

## Current Limitations

This is a simulation project, not a real industrial system.

Current limitations:

- no real flow meters
- tank levels are used as volume references
- 1-liter bottle filling is simulated using tank level decrease
- conveyor movement is based on Factory I/O sensor behavior
- no real CIP cleaning cycle
- no alarm history
- no full HMI screen design included

## Possible Future Improvements

Possible improvements for future development:

- add an HMI screen with recipe selection
- add multiple recipes
- add flow meter simulation
- add an alarm list
- add manual mode
- add maintenance mode
- add bottle presence check before filling
- add maximum and minimum tank safety limits
- add production statistics
- add total produced liters counter
- add cleaning or drain cycle
- clean up variable names and typos

## Example Production Scenario

User enters:

```text
5 L
```

The PLC calculates the recipe:

```text
Water = 1.5 L
Apple = 3.5 L
```

Then the system:

```text
fills tanks
adds apple first
adds water second
creates apple juice
fills 1 L bottle
moves conveyor
counts bottle
repeats the cycle
```

If Stop is pressed:

```text
current cycle finishes
all tanks are drained
system returns to waiting mode
```

If Emergency Stop is pressed:

```text
everything stops immediately
red light blinks
reset is required
```
