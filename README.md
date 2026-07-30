Youssef Alkhoder and Zakaria Zakaria Project (ULFGI 3rd year)
================================================================

ULTIMATE XO / ULTIMATE TIC-TAC-TOE ON DEEDS DMC8 
========================================================

Project goal
------------
This project implements an Ultimate XO game using the DEEDS DMC8 processor, external button input, decoders, RAM state storage, 8x8 display cells, and one large 64x8 display.

Game idea
---------
The main game contains 9 small XO blocks arranged as:

[ 0  1  2 ]
[ 3  4  5 ]
[ 6  7  8 ]

Each small block contains 9 local cells arranged as:

[ 0  1  2 ]
[ 3  4  5 ]
[ 6  7  8 ]

The global playable cell number is:

global_cell = block_number * 9 + local_cell_number

Full playable numbering:

[  0   1   2 |  9  10  11 | 18  19  20 ]
[  3   4   5 | 12  13  14 | 21  22  23 ]
[  6   7   8 | 15  16  17 | 24  25  26 ]
------------------------------------------------
[ 27  28  29 | 36  37  38 | 45  46  47 ]
[ 30  31  32 | 39  40  41 | 48  49  50 ]
[ 33  34  35 | 42  43  44 | 51  52  53 ]
------------------------------------------------
[ 54  55  56 | 63  64  65 | 72  73  74 ]
[ 57  58  59 | 66  67  68 | 75  76  77 ]
[ 60  61  62 | 69  70  71 | 78  79  80 ]

Display numbering
------------------
0..80   = playable game cells where X/O are drawn.
81..89  = isolated visual result cells, one for each small block.
90      = large 64x8 message display for NEW BLK / X WIN / O WIN / DRAW.
91	= Display of the current player (X or O) turn.

Isolated result display:

[ 81  82  83 ]
[ 84  85  86 ]
[ 87  88  89 ]

Important: the large display is 90 decimal = 5AH. With the OA enable bit set, the value sent to OA is DAH.

Ports
-----
IA = 00H : input from priority encoder/buttons.
OA = 00H : display W-pin selector. Bits 0..6 select display number; bit 7 enables write.
OE = 04H : column number for the selected display.
OF = 05H : byte pattern/lighted points for that column.
OG = 06H : current block indicator.

OA write rule
-------------
To write to a display, send:

OA = display_number OR 80H (bit 7 used for enabling the 2->4 decoder)

Examples:
playable cell 0  -> OA = 80H
playable cell 80 -> OA = D0H
result cell 81   -> OA = D1H
large display 90 -> OA = DAH

After writing, disable the W pins:

LD A,00H
OUT (OA),A

RAM organization
----------------
8000H..8050H : 81 playable cells.
8100H..8108H : 9 small-block results.
9000H..9005H : current block/cell and temporary variables.

Cell/result values:
FFH = empty/open
A0H = X
50H = O
D0H = draw

Register roles
--------------
IX points to the selected playable cell address.
IY points to the first cell address of the current block.
D stores turn and flags:
  bit 0 = turn: 0 for X, 1 for O
  bit 1 = draw flag
  bit 2 = win flag
  bit 3 = closed/occupied flag
  bits 4..7 = result high nibble: A for X, 5 for O, D for draw

Input routine
-------------
The buttons enter through a priority encoder. The final DMC8 input is 1..9 when a button is pressed and 0 when idle. The code decrements this once, so the game receives 0..8.

button 1 -> 0
button 2 -> 1
...
button 9 -> 8

The input routine should wait for release to avoid reading the same press twice.

Game flow
---------
1. Initialize RAM cells/results to FFH.
2. X starts.
3. Show NEW BLK on the large display 90.
4. Player chooses any available block.
5. Player selects a local cell in the current block.
6. If the cell is occupied, ask again without toggling turn.
7. If empty, draw X/O, store A0H/50H in RAM, and check the small block.
8. If the small block has a winner, store the result and draw it in the isolated result display.
9. If the small block is full with no winner, store D0H and draw the draw symbol.
10. Check the main game using 8100H..8108H.
11. If X/O wins the main game, show X WIN or O WIN on display 90.
12. If all 9 small-block results are known and no winner, show DRAW on display 90.
13. Otherwise, the local cell just played becomes the next forced block.
14. If that forced block is already closed, show NEW BLK and let the next player choose any open block.

Small-block win lines
---------------------
Rows:    [0,1,2], [3,4,5], [6,7,8]
Columns: [0,3,6], [1,4,7], [2,5,8]
Diags:   [0,4,8] (\), [2,4,6](/)

Main game win uses the same line positions in RAM 8100H..8108H.
D0H draw must not count as a winner.

Circuit summary
---------------
The DMC8 connects to button input through IA, and to displays through OA/OE/OF. OA selects the W pin through a decoder tree. OE/OF provide column index and column bitmap data. OG indicates the current active block as 10H + block_number.

OG values:
block 0 -> 10H
block 1 -> 11H
block 2 -> 12H
block 3 -> 13H
block 4 -> 14H
block 5 -> 15H
block 6 -> 16H
block 7 -> 17H
block 8 -> 18H

Testing checklist
-----------------
[ ] On reset, all 8000H..8050H and 8100H..8108H are FFH.
[ ] NEW BLK appears on display 90.
[ ] OA for display 90 is DAH while writing.
[ ] OA for playable cell 0 is 80H.
[ ] OA for playable cell 80 is D0H.
[ ] OA for result cell 81 is D1H.
[ ] OG outputs 10H..18H for blocks 0..8.
[ ] X writes A0H and O writes 50H in RAM.
[ ] Draw writes D0H only in block result RAM, not as a winner.
[ ] Occupied cells do not toggle the turn.
[ ] Forced next block works correctly.
[ ] If forced block is closed, NEW BLK appears and free block choice is enabled.
