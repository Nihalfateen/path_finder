# RM_TP3 - Path Finder Robot

Mobile Robotics Assignment 2 project for the DETI Path Finder Robot.

The robot follows black lines on a white floor using five IR ground sensors,
explores a line maze, detects a wide-line target, computes an optimized return
route, and navigates back toward the starting position.

## Project Status

Main agent:

```text
rm_deti_rob/src/rm-pathfinder.c
```

Current terminal banner:

```text
Path Finder Robot - exploration left probe-target-v44-snapshot-decision
```

Implemented behavior:

- line following with a weighted five-sensor error
- intersection detection with debounce
- left-first exploration using `L`, `S`, `R`, and `B` moves
- dead-end detection and line reacquire before U-turn
- target detection using wide-line width and sustained `11111` readings
- raw path recording and path simplification
- return path generation by inverting the optimized path
- automatic return after target confirmation, with a 3 second pause
- odometry-assisted stop near the original start pose

## Repository Layout

```text
.
├── README.md
├── rm_2526_assignment2.md
├── rm_deti_rob/
│   ├── src/
│   │   ├── rm-pathfinder.c    # main robot agent
│   │   ├── rm-mr32.c          # robot support library
│   │   ├── rm-mr32.h
│   │   └── rm-example.c       # original example
│   └── bluetooth/
└── outputs/
    ├── path-finder-robot-presentation.pptx
    └── report/
        ├── path_finder_robot_lncs.pdf
        └── path_finder_robot_lncs.tex
```

## Assignment Requirements Covered

The assignment asks for a robot that can:

1. follow a black line;
2. choose left at intersections during exploration;
3. find the final target marked by a wide black line;
4. return from the target to the start using the shortest path;
5. stop at the start position.

This implementation addresses those requirements with a finite-state flow:

```text
Idle -> Explore -> Target Found -> Return -> Finished
```

At the target, the robot no longer waits for a second start-button press. It
prints:

```text
Target reached; auto return in 3 seconds
```

Then it starts return navigation automatically.

## Build

The official assignment toolchain uses PIC32 compiler tools. On a Linux
machine, VM, or compatible course environment:

```sh
cd rm_deti_rob/src
pcompile rm-pathfinder.c rm-mr32.c
```

Expected generated output:

```text
rm-pathfinder.hex
```

The PIC32 toolchain provided for the course is usually a Linux binary. On macOS
Apple Silicon, use a Linux VM, university Linux machine, or compatible
container rather than running the compiler directly on the host.

## Upload and Run

Connect the robot over USB, then run:

```sh
ldpic32 -w rm-pathfinder.hex
pterm
```

Press the robot start button once to begin exploration.

Expected startup log:

```text
Path Finder Robot - exploration left <version>
Press start to run
Start pose reset x=0 y=0 theta=0mrad
```

## Runtime Log Milestones

Useful messages to confirm each phase:

```text
Intersection detected
Line width ticks=<n> fullMask=<n>
Exploration move selected: L/S/R/B
Dead end line reacquired backup=<n>
Target confirmed at intersection sensors=11111 width=<n> fullMask=<n>
Recorded path (...)
Optimized path (...)
Return path (...)
Target reached; auto return in 3 seconds
Return navigation starting
Return move: L/S/R/B
Finished; press stop to reset
```

## Important Constants

The most useful physical tuning values are in `rm-pathfinder.c`:

```c
#define BASE_SPEED 30
#define TURN_GAIN 6
#define SEARCH_SPEED 20
#define TURN_SPEED 25
#define TARGET_WIDTH_MIN_TICKS 18
#define TARGET_FULL_MASK_MIN_TICKS 5
#define TARGET_RETURN_PAUSE_TICKS 150
#define START_REACHED_RADIUS_MM 300
```

Timing note:

```text
TARGET_RETURN_PAUSE_TICKS = 150
150 * 20ms = 3 seconds
```

## Deliverables

Generated deliverables are available under `outputs/`:

- `outputs/path-finder-robot-presentation.pptx`
- `outputs/report/path_finder_robot_lncs.pdf`
- `outputs/report/path_finder_robot_lncs.tex`

The report is formatted with the Springer LNCS LaTeX class.

## Demo Checklist

Before the final demo, verify:

- the robot follows straight and curved line segments;
- normal intersections are not confused with the target;
- dead ends print `Dead end line reacquired backup=<n>`;
- the target prints `Target confirmed ...`;
- the robot pauses at the target and starts return automatically;
- the return path is consumed with `Return move: ...` logs;
- the robot stops near the start pose.

## Notes

- The environment is assumed to have no cycles, matching the assignment brief.
- The raw path keeps dead-end backtracks so the optimizer can simplify them.
- If the robot passes over the target without confirming it, tune
  `TARGET_WIDTH_MIN_TICKS`, `TARGET_FULL_MASK_MIN_TICKS`, or the physical
  alignment at the target marker.
- If the robot spins after losing the line, inspect the dead-end reacquire logs
  before changing the exploration strategy.
