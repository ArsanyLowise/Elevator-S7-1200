# Ladder networks

The whole cycle is in one function block, `FB_ElevatorCycle`. It is written in SCL, and this file gives the same logic as Ladder networks so it can be built by hand in TIA Portal if you prefer. The step numbers match the integers in the block.

The instance data block is called `DB_Elevator`. Put one call in OB1, in network 1, with this mapping:

| Block pin | PLC tag |
| :-- | :-- |
| Start | "Start" |
| Stop | "Stop" |
| Reset | "Reset" |
| EStop | "EStop" |
| AutoMode | "AutoMode" |
| BoxAtEntry | "BoxAtEntry" |
| BoxOnElevator | "BoxOnElevator" |
| BoxAtExit | "BoxAtExit" |
| AtLow[0] | "AtLow0" |
| AtLow[1] | "AtLow1" |
| AtLow[2] | "AtLow2" |
| AtLow[3] | "AtLow3" |
| AtHigh[0] | "AtHigh0" |
| AtHigh[1] | "AtHigh1" |
| AtHigh[2] | "AtHigh2" |
| AtHigh[3] | "AtHigh3" |
| PlateSeen[1] | "PlateSeen1" |
| PlateSeen[2] | "PlateSeen2" |
| PlateSeen[3] | "PlateSeen3" |
| Belt[0] | "Belt0" |
| Belt[1] | "Belt1" |
| Belt[2] | "Belt2" |
| Belt[3] | "Belt3" |
| BeltExit | "BeltExit" |
| BeltElevator | "BeltElevator" |
| Up | "Up" |
| Down | "Down" |
| Slow | "Slow" |
| LampFloor[1] | "LampFloor1" |
| LampFloor[2] | "LampFloor2" |
| LampFloor[3] | "LampFloor3" |
| LampStart | "LampStart" |
| LampStop | "LampStop" |
| LampReset | "LampReset" |
| LampWarning | "LampWarning" |

The outputs Run, Busy, Fault and PlatesLoaded have no terminal of their own. Leave them open, or send them to a watch table while you commission the line.

## Network 1, call

One normally open contact of "Always TRUE", or the block called unconditionally, and the block EN goes to the instance `DB_Elevator`.

## Network 2, outputs released

A first scan of every output coil, each one normally open with the step bit of the reset path, so the belts, Up, Down and Slow start every scan dropped. The SCL does this at the top of the body with a FOR loop.

## Network 3, panel buttons

Three branches. A normally closed "EStop" contact in series with a coil that sets the fault memory, resets the start memory and loads zero into the step word. A rising edge of "Reset" clears the fault memory and loads 90 into the step word, which is the way home. A normally open "Start" contact in parallel with the start memory sets that memory, and a normally open "Stop" contact resets it.

## Network 4, timers

Six timer coils, each gated by a comparison of the step word so it only runs in its own step.

| Timer | Runs in step | Preset | What it waits for |
| :-- | :-- | :-- | :-- |
| tonSettle | 10 | 700 ms | box on the elevator and clear of the entry sensor |
| tonNoBox | 10 | 15 s | no box on the elevator |
| tonPush | 30 | 2 s | plate sensor of the target level |
| tonNoPlate | 30 | 10 s | plate sensor of the target level quiet |
| tonUnload | 50 | 1 s | exit sensor seen and then released |
| tonTravel | 20 and 90 | 30 s | travel watchdog |

## Network 5, step 0, idle

Compare the step word with zero. A normally open "start memory" contact and a normally closed "fault memory" contact move the step word to 5.

## Network 6, step 5, new cycle

Compare with 5. Move zero into the plate counter, one into the schedule index, zero into the target level, then load the first entry of the schedule into the target and 30 into the next step, and finish with 10 in the step word. In Ladder this is a block move of five words, which is why the SCL version is shorter.

## Network 7, step 10, load the box

Compare with 10. Coils "Belt0" and "BeltElevator". The done bit of tonSettle moves 20 into the step word, and the done bit of tonNoBox sets the fault memory, loads zero and goes back to the idle step.

## Network 8, step 20, travel

Compare with 20. The step word is safe to compare with the target because the block only leaves a level through this step.

Going up is a branch of a greater comparison of target and current level, and it carries "Up" with a normally closed "AtHigh" of the target level, and "Slow" with a normally open "AtLow" of the target level. Going down is the mirrored branch with "Down" and the two sensors exchanged. A third branch, target equal to current, jumps straight to the next step.

Arrival is one parallel pair. Up to the target and "AtHigh" of that level, or down to the target and "AtLow" of that level, moves the target into the current level and the next step into the step word.

## Network 9, step 30, plate out

Compare with 30. A normally closed "push done" contact drives the belt coil of the target level, indexed by the target level. When the push timer finishes, the plate counter goes up by one, the schedule index goes up by one, and the target becomes the next entry of the schedule, or zero with step 50 after the fifth plate. The no plate timer sets the fault memory instead.

## Network 10, step 50, send the box out

Compare with 50. Coils "BeltElevator" and "BeltExit". A normally open "BoxAtExit" contact sets a memory that says the box has reached the outfeed side. The done bit of tonUnload with an open "AutoMode" contact and an open "start memory" contact goes to step 5 for the next box, and without that branch it clears the start memory and goes to the idle step.

## Network 11, step 90, way home after reset

Compare with 90, or compare with zero if step zero is used for idle and the reset path instead loads a bit. A normally open "AtLow[0]" contact goes straight to the idle step and stores zero in the current level, and the rest of the branch drives "Down", with "Slow" taken from "AtHigh[0]".

## Network 12, travel watchdog

A normally open "travel done" contact of tonTravel, in series with a normally closed comparison against 90, sets the fault memory and loads zero. This catches a level sensor that never answers, which otherwise would leave the elevator driving into the end of the shaft.

## Network 13, lamps

For each floor from 1 to 3, a parallel pair of "AtLow" and "AtHigh" drives the floor lamp, so the lamp is on when the elevator is level with that floor or on its way into it. Then "Run" is a comparison of the step word against zero in series with a normally closed comparison against 90, "Busy" is a parallel pair of "Up" and "Down", and the four lamps follow the memories: start from Run, stop from an inverted start memory, reset from the Reset input, warning from the emergency stop or the fault memory in parallel.
