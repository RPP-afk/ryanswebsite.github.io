# John Conway's Game of Life

This coding challenge looks at creating [John Conway's](https://en.wikipedia.org/wiki/John_Horton_Conway) Game of Life simulation in Python. Before we get into the details of the code, let us review what this game is.

The Game of Life is as it says on the tin: a simulation that is meant to mimic basic life using only a few simple rules. This is is an example of [cellular automata](https://en.wikipedia.org/wiki/Cellular_automaton) which evolve from an initial state and converge to some steady state in some finite period of time. The Game of Life is an example of one of these automata, constructed by Conway in 1970.

The rules for the game of life are as follows:

Given a grid of square grid cells

- Any live cell with fewer than two live neighbours dies, as if by underpopulation.
- Any live cell with two or three live neighbours lives on to the next generation.
- Any live cell with more than three live neighbours dies, as if by overpopulation.
- Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

Below is a list of steps that need to be taken to run the game.

### To Do:

1) Define a grid of cells to run the game on.
2) Code up the above rules and apply create a future grid.
3) Replace the current grid with the new grid.
4) Carry on the loop for a set number of generations.

## Step 0: Imports and variables

First we need to define some inputs and imports for the program.

```python
# imports 
import numpy as np
import matplotlib.pyplot as plt
import pygame
import pygame.surfarray as surfarray
import random
# define the number of pixels along each pixel edge
cell_size = 20
# height and width of the grid (in pixels)
width = 1500
height = 1500
# define the number of rows and columns of cells needed to fill the grid
rows = int(height / cell_size)
cols = int(width / cell_size)
# define the max generations to run the simulation
maxGenerations = 2000
# initial setup for pygame
pygame.init()
screen = pygame.display.set_mode((height, width))
clock = pygame.time.Clock()
```

## Step 1: Construct A Grid

To construct a grid of cells we first need to define a `Cell` class. I did this as follows:

```python
class Cell:

    def __init__(self, x_index, y_index, width, state, screen):
        self.x_index = x_index
        self.y_index = y_index
        self.width = width
        self.state = state
        self.screen = screen

    def makeRect(self):
        if self.state == 1:
            color = "black"
        else:
            color = "white"

        return pygame.draw.rect(
            self.screen,
            color,
            [
                self.x_index * self.width,
                self.y_index * self.width,
                self.x_index * self.width + self.width,
                self.y_index * self.width + self.width,
            ],
        )
```


## Step 2: Apply The Logic

In order to apply the rules to the game, we need to be able to work out the number of living neighbours around a given cell. To do this we can write the following function:

```python
def countNeighbors(grid, x, y):
    sum = 0
    for i in range(-1, 2):
        for j in range(-1, 2):
            # we use the modulus operator here as means to apply cyclic boundary conditions 
            col = (x + j + cols) % cols
            row = (y + i + rows) % rows
            sum += grid[col][row].state
    sum -= grid[x][y].state
    return sum
```
<!-- Draw a grid to demonstrate the process of counting the neighbours -->

