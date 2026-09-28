# Line Follower: Initio Robot Car

Coursework 1 for **6COM2009 Real-Time and Concurrent Systems (RTCS) 2025**.

An ISO C program that drives an Initio robot car along a black line on a white floor, and stops automatically when an obstacle is in front of it. It runs on a Raspberry Pi using the `initio` library, with `curses` for a simple on-screen status display.

## How It Works

### Sensors

| Sensor | Function | Purpose |
|---|---|---|
| Front obstacle IR (left / right) | `initio_IrLeft()`, `initio_IrRight()` | Detect an object ahead (non-zero = obstacle) |
| Line-following IR (floor-facing, left / right) | `initio_IrLineLeft()`, `initio_IrLineRight()` | Detect black line vs. white floor |

### Actuators

Two DC motors, controlled with PWM (0-100):

- `initio_DriveForward(speed)`
- `initio_SpinLeft(speed)`
- `initio_SpinRight(speed)`

`initio_Init()` and `initio_Cleanup()` handle setup and shutdown.

### Control Logic

Each loop iteration reads all four sensors, then picks one action in priority order:

| Condition | Action |
|---|---|
| Either obstacle sensor triggered | **Stop** |
| Both line sensors on black | **Drive forward** |
| Left black, right white | **Spin left** |
| Left white, right black | **Spin right** |
| Both white (line lost) | **Recover**: spin in the last known turn direction (stop if none) |

**Obstacle avoidance and fault tolerance:** two independent obstacle sensors give redundancy. If one misreads or is misaligned, the other still stops the car. The car only resumes line following once both sensors report clear.

**Line recovery:** a `lastDirection` variable (-1 left, 0 straight, +1 right) remembers the most recent turn. If the line is lost, the car keeps spinning that way to re-acquire it, which helps on curves and noisy line segments.

**Why not PID?** The line sensors are binary (on/off), so continuous control methods like PID don't apply.

### Tunable Parameters

Defined at the top of `line_follower.c`:

| Constant | Value | Meaning |
|---|---|---|
| `forwardSpeed` | 87 | Forward drive PWM |
| `turnSpeed` | 100 | Spin PWM |

If your sensors are inverted (line reads as 0), switch to the alternate `WHITE(a)` macro noted in the source.

## Project Structure

```
line_follower/
├── line_follower.c   # Program source
└── Makefile          # Build and run targets
```

## Requirements

- Initio robot car on a Raspberry Pi
- `gcc`
- Libraries: `initio`, `curses`, `wiringPi`, `pthread`

## Build and Run

```bash
make            # compile
make run        # compile and run
make schedule   # run under the RTCS scheduler (rtcs_schedule)
make clean      # remove the binary
make help       # list commands
```

The Makefile compiles with `-Wall -Werror -mfloat-abi=hard`, so warnings fail the build.

**Controls:** press `q` to stop the program. The car halts and the hardware is cleaned up on exit.

