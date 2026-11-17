# Robot Wars (C++ console simulator)

A turn-based robot battle simulator for the console. You describe a battlefield and a set of robots in a
plain-text file, and the program runs the battle turn by turn, printing the map and robot stats after each
turn. The full battle is also written to `battlefield.txt`.

## Requirements

- A C++17 compiler. This was built with g++ 15.1 from MSYS2 (MinGW-w64) on Windows.

## Building

Run these from the folder containing the `.cpp` and `.h` files.

**g++ (MinGW / MSYS2)**

```
g++ -std=c++17 *.cpp -o robotwars.exe
```

If `g++` isn't recognised, add the compiler's `bin` folder to PATH first. For MSYS2 this is:

```
set PATH=C:\msys64\mingw64\bin;%PATH%
```

The compiler's bin folder also has to be on PATH when you run the program, because the
MinGW runtime DLLs are loaded from there.

**Visual Studio (MSVC)**

Open the *Developer Command Prompt for VS 2022* in the same folder and run:

```
cl /EHsc /std:c++17 *.cpp /Fe:robotwars.exe
```

## Running

```
robotwars.exe
```

The program asks for the name of a battlefield file:

```
Enter the filename you want the simulator to access:
10x10.txt
```

It prints the initial battlefield, then each turn, and finally `End of simulation`. The battle stops when
only one robot is left standing or when the number of turns in the file is reached.

Run the program from the folder that holds your battlefield file, because the filename is read relative to
the current folder. `battlefield.txt` is also written to the current folder. It is recreated on each run.

## Battlefield files

The sample files `10x10.txt`, `20x20.txt` and `30x30.txt` show the format. Each file has a header and then
one line per robot:

```
M by N : 10 10
steps: 30
robots: 7
Madbot Mold 3 6
RoboTank Orion 9 1
Terminator T-1000 4 random
BlueThunder Bast random random
Sentinel Seneca 5 5
RoboCop Ra-One 1 1
TerminatorRoboCop Chrono 8 8
```

| Line | Meaning |
|------|---------|
| `M by N : rows cols` | Size of the battlefield. |
| `steps: n` | Maximum number of turns. |
| `robots: n` | Number of robot lines that follow. **Must match the number of robot lines exactly.** |
| `Type Name X Y` | One robot: its class, its name, and its starting position. |

- `X` and `Y` are grid coordinates, or `random` to place the robot on a free square.
- A robot's map icon is the first letter of its name, so give robots different first letters.
- Robot types are case-sensitive and must be spelled exactly as in the table below.

If `robots:` is higher than the number of robot lines, the program reads blank lines and stops with
`The may be a robot type that has been mispelt`. The sample `10x10.txt` currently says `robots: 9`
but lists 7 robots, so change it to `robots: 7` before running it.

## Robot types

| Type | Behaviour each turn |
|------|---------------------|
| `RoboCop` | Looks at its 8 neighbours. If a robot is adjacent it moves away from it, otherwise it moves one tile at random. Fires 3 shots at random squares within range 10 (x + y ≤ 10). |
| `Terminator` | Looks at its 8 neighbours. If a robot is adjacent it steps onto it, killing it. Otherwise it moves one tile at random. |
| `TerminatorRoboCop` | Moves like `RoboCop`, steps on an adjacent robot like `Terminator` (stepping takes priority), and fires 3 shots within range 10. Shown as `T-RoboCop` in the output. |
| `BlueThunder` | Cannot move. Fires at its 8 neighbours in clockwise order, one per turn. |
| `Madbot` | Cannot move. Fires at a random neighbouring square each turn. |
| `RoboTank` | Cannot move. Fires at a random square anywhere on the map each turn. |
| `UltimateRobot` | Moves and steps like `TerminatorRoboCop`, but fires at random anywhere on the map. |
| `Sentinel` | Moves one tile at random (it can't look). Fires 2 shots within range 10. |
| `PrimeSentinel` | Moves one tile at random and steps on any robot it lands on. Fires at random anywhere on the map. |

## How a battle works

- Each turn, every robot takes its action in list order.
- A shot that lands on a robot kills it. A dead robot goes into a queue.
- At the end of each turn, one robot from the queue respawns at a random square.
- A robot that has died three times is removed for good.
- The battle ends when one robot is left or the step limit is reached.

Each robot class derives from the shared behaviour classes in `Robots.h`: `MovingRobot`, `ShootingRobot`,
`SeeingRobot` and `SteppingRobot`.

## Project layout

| File(s) | Purpose |
|---------|---------|
| `Main.cpp` | Reads the battlefield file, runs the turn loop, writes the output. |
| `Robots.h/.cpp` | Base `Robot` class and the behaviour interfaces. |
| `Map.h/.cpp` | The battlefield grid. |
| `RobotList.h/.cpp` | The list of robots still in the battle. |
| `RobotQueue.h/.cpp` | Robots waiting to be removed (dequeued) after they are destroyed. |
| `*.h/.cpp` for each robot type | The robot implementations. |

## Troubleshooting

| Message | Cause |
|---------|-------|
| `'g++' is not recognized` | The compiler isn't on PATH. See *Building*. |
| `Error opening the .txt file` | The battlefield filename is wrong, or the file isn't in the folder you ran the program from. |
| `The may be a robot type that has been mispelt` | A robot type is misspelt, or `robots:` is larger than the number of robot lines. |
| Program exits immediately with a DLL error | The compiler's `bin` folder isn't on PATH. See *Building*. |
