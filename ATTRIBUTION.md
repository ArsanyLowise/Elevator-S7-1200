# Attribution

The scene is the one Factory I/O ships as Elevator (Advanced). This project is a rewritten and extended implementation of the work published at https://github.com/hugotallys/factoryio-elevator by Lucas Lemos Cerqueira de Freitas and Hugo Tallys Martins Oliveira, made for the Industrial Automation course of the Institute of Computing at the Federal University of Alagoas, second semester of 2022.

What came from that project: the scene, the level layout, the mapping between the scene tags and the PLC addresses, and the order in which plates are collected. The three screenshots in this folder are taken from its documentation.

What is different here: the cycle is a single function block written in SCL with a step counter instead of a bit per step in a shared data block, the travel logic decides direction and slow down from the current and target level, the plate and box steps have timeouts that raise a fault, the outfeed belt is driven by the program, and the documentation is in English.

The original Ladder diagrams are in its repository as `LADDER.pdf`, and its TIA Portal projects are in the V15 and V16 folders there.
