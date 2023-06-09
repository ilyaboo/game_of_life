# Conway's Game of Life

<p align = "center">
  <img src = "https://user-images.githubusercontent.com/108785038/231323309-37a6fc78-022b-4e50-afbb-f61d9f1e2add.gif">
</p>
  
This is a C program that implements [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life). The program allows the user to select the initial alive cells manually as well as to select one of the presets.

## Background information

Conway's Game of Life (classified as a cellular automaton) is a game that simulates the evolution of the cells. The player can interact with the field in order to create the initial configuration of the cells. The evolution of this configuration is then simulated by the game and does not require any user input. That is why the game is sometimes called [zero-player game](https://en.wikipedia.org/wiki/Zero-player_game).

## Rules of the game

The game consists of a two-dimensional grid of squares, each representing one cell. The sell can be either dead (empty) or alive (filled with color). The future state of each cell is determined by its neighbors – cells that are adjacent to it vertically, horizontally, and diagonally (total of 8 cells). The game itslf consists of the simulation of the evolution of the cells, which is visualized by consequent updates of the game field. Each field update is done by determining the new state of each cell on the field according to the following rules.
#### If the cell was alive:
- If the cell has fewer than 2 live neighbors, it dies, as if by underpopulation.
- If the cell has 2 or 3 live neighbors, it lives on to the next generation.
- If the cell has more than 3 live neighbors, it dies, as if by overpopulation.
#### If the cell was dead:
- If the cell has exactly 3 live neighbors, it becomes a live cell, as if by reproduction.
- If the number of live neighbors is not equal to 3, it remains dead.

The game continues updating the field with newer generations of the cells by applying the rules listed above to each cell of the previous generation.

## Implementation

### Relevant libraries

In order to implement the game I used the following libraries (headers).
- **stdio.h** – standard C library for input/output
- **unistd.h** – standard C library needed for setting the field refresh rate
- **gfx.h** – [graphics library](https://www3.nd.edu/~dthain/courses/cse20211/fall2013/gfx/) needed for all the graphics of the project
```c
#include <stdio.h>
#include <unistd.h>
#include "gfx.h"
```
*Notice the **gfx.c** and **gfx.h** files in the repository needed for the **gfx** library to work*

### Headers

#### ```params.h```

This file contains a number of parameters which affect the behaviour of the program as well as some display properties. These global parameters include the following.
```c
const int max_epochs = 50;   // the maximum number of epochs after which the program stops, default: 50
const int num_squares = 40;   // number of sqaures in a row/column, default: 40
```
These parameters affect the general game mechanics. By changing `max_epochs` one can change the maximum number of generations the program will process until it stops. `num_squares` is responsible for the number of squares that fit the width/height of the square field.

```c
const int side = 15;   // length of the side of one square, default: 15
const int margin = 10;   // margin between the field and edge of the window, default: 10
```
These parameters are responsible for visual representation of the field. `side` is the side of one cell in pixels, whereas `margin` is the distance between the edge of the window and the field measured in pixels.

Lastly, there are parameters that set the sizes and locations of the interface buttons.

Even though some of these parameters repeat themselves, I intentionally left them like that in order to be able to adjust button positions and sizes.

#### ```interface.h```

Interface functions use the gfx library and are used to set up the interface of the program. Most of these function utilize the helper function `void fill(x1, y1, x2, y2)`, which fills the rectangular area between points `(x1, y1)` and `(x2, y2)` with color. Other functions in these categories include helper functions, which simplify the use of the "fill" function, functions that generate buttons, `void gen_board()` function that reads from the `field` 2D array to display its current state, and `void update_all()` function that applies all the changes made to the board.

#### ```interact.h```

These are used to process user input. This section has a `int click_pos(int x, int y)` function that identifies on which part of the interface did the user click. Helper functions `int get_row(int x)` and `int get_col(int y)` help identify the cell on which the user clicked.

#### ```field.h```

Functions contained in this header manipulate the `field` 2D arrray. These include `void empty_field()` which sets all the values of the array to zeros, preset functions, which set the `field` state according to the provided preset, as well as the `int update_field()` which updates the `field` by one generation accoring to the rules of game. Notice that this function treats the field as a confined area, making the cells on one edge of the field adjacent to the ones on the opposite side of the field. This implementation enhances the game experience while still allowing the user to explore classical patterns.

### ```main.c```

This file contains the `int main()` function which is responsible for utilizing the headers described above in order to rub the game. It has the initializations of `ysize` and `xsize`, which are the height and the width of the program window respectively.
```c
int ysize = margin * 2 + side * num_squares;
int xsize = margin * 3 + side * num_squares + start_button_width;
```
Notice that both integers are calculated dynamically depending on some of the parameters defined in the beginning of the program. This allows the size of the window to perfectly fit all the elements of the interface.

It also creates a variable ```sleep_time``` which corresponds to the time delay between the field updates.
```c
int sleep_time = 10000;   // time for refreshing the field in microseconds
```
Then the program creates a window with calculated dimensions.
```c
gfx_open(xsize, ysize, "Game of Life");
```
Then the default color is set to green.
```c
gfx_color(0, 200, 100);
```
Finally, the call for the `update_all()` function makes all the interface elements appear in the window.
```c
update_all(sleep_time);
```
The program then inters an infinite loop which reads user's input – a click of the left mouse button (LMB) on one of the interface elements.
```c
int click = gfx_wait();
```
After detecting the click, the program calls the ```click_pos()``` function to evaluate the click.
```c
int result = click_pos(x, y);
```
The program will then adjust the interface according to the result of the evaluation. Options include adding or removing the cell from the initial setup, resetting the setup, loading one of the presets, changing the framerate, and launching the simulation. Notice that active simulation will either finish after creating a maximum number of generations of generations specified in the beginning of the program, or when it detects that the field reached a static state.

## Installation

Clone this repository by entering the following command in the terminal:

```bash
$ git clone https://github.com/ilyaboo/game_of_life.git
```

Navigate to the project directory and compile the program using the Makefile:

```bash
$ cd game_of_life
$ make
```

## Starting the program

To launch the program, run the compiled binary:

```bash
$ ./game_of_life
```

## How to play

The interface of the game consists of the following elements:

- *Field* (grid) that consists of the empty squares. Click on the square to place a live cell in it. Click in the live cell once again to remove it.
- *Green play button* that will launch the simulation from the current state of the field. The simulation will **automatically end** after processing 50 (by default) generations of the cell or when it detects a static state (new generations are identical to the previous ones), including death of all of the cells. Note that to continue the simulation after the program processed 50 generations you can simply click on the green button again.
- *Red reset button* allows you to reset the current state of the field, removing any live cells.
- *Orange framerate change buttons* are responsible for changing the framerate of the simulation. The one with upward arrow increases the framerate, whereas the other one decreases it. Current framerate is shown on the indicators above the buttons.
- *Gray buttons* are responsible for loading presets. Click on a gray button to make the preset appear on the field. You can either launch the simulation from there by clicking on the green button or adjust the field the way you want.

You can generate your own cell configurations or watch the pre-loaded ones evolve.

## Enjoy!
