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

To construct a grid of cells we first need to define a `Cell` class. This can be done as follows:

```python
class Cell:
    """
    A class representing a cell in the game grid.
    """
    def __init__(self, x_index, y_index, width, state, screen):
        self.x_index = x_index
        self.y_index = y_index
        self.width = width
        self.state = state
        self.screen = screen

    def makeRect(self):
        """
        Draw a grid cell as a rectangle and colour it based
        on whether it is alive or dead.
        """
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

Here we define the cell's index location in the grid, `self.x_index, self.y_index`. We also define a state, to highlight whether a the cell is dead (0) or alive (1). There is also the `self.screen` which basically indicates the location to draw the cells, and finally a `self.width` attribute to indicate the number of pixels the cell should have along it's length. 

The only method of this class is the `makeRect()` function. This takes the variables assigned to the cell on initalisation and when called draws a grid square on the screen element - which in this case will be a html canvas element.

To fill this grid, we can therefore do the following: 


```python
rng = np.random.default_rng() # define a random number seed

# Initial grid:
for j in range(rows):
    row_list = []
    for i in range(cols):
        row_list.append(Cell(j, i, cell_size, rng.integers(2), screen))
    grid.append(row_list)
```

## Step 2: Apply The Logic

In order to apply the rules to the game, we need to be able to work out the number of living neighbours around a given cell. To do this we can write the following function:

```python
def countNeighbors(grid, x, y):
    """
    Count the number of living grid cells in the immediate neighbourhood
    of a grid cell at point (x,y) in the grid.
    """
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


For each grid cell, we essentially count the number of living cells in the 8 connecting neighbours. The result is then used in the following function to determine whether a cell lives or dies. 

```python
futureGrid = []

for j in range(rows):
    row_list = []
    for i in range(cols):
        # aliveNeighbours = countNeighbours(grid, i, j)
        state = grid[j][i].state
        aliveNeighbours = countNeighbors(grid, j, i)
        if (aliveNeighbours > 3 or aliveNeighbours < 2) and state == 1:
            row_list.append(Cell(j, i, cell_size, 0, screen))
        elif (aliveNeighbours == 3) and state == 0:
            row_list.append(Cell(j, i, cell_size, 1, screen))
        else:
            row_list.append(Cell(j, i, cell_size, state, screen))
    futureGrid.append(row_list)
```

Note that we are applying the logic to an empty `futureGrid`. This is important as if you update the cells on the same grid, then you are essentially mixing generations which can lead to undesirable behaviour.

## Step 3: Update the grid

Now we have created the future grid, we now need to draw it on the screen, and prepare for the next loop of the simulation.

```python
# cycle through each element of the array and draw it
for i in range(cols):
    for j in range(rows):
        futureGrid[j][i].makeRect()

# swap grids
grid = futureGrid
```

## Step 4: Bring it all together

Below is the full script.

```python
import numpy as np
import matplotlib.pyplot as plt
import pygame
import pygame.surfarray as surfarray
import random

cell_size = 20
width = 1500
height = 1500

rows = int(height / cell_size)
cols = int(width / cell_size)

maxGenerations = 2000

pygame.init()
screen = pygame.display.set_mode((height, width))
clock = pygame.time.Clock()

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

def countNeighbors(grid, x, y):
    sum = 0
    for i in range(-1, 2):
        for j in range(-1, 2):
            col = (x + j + cols) % cols
            row = (y + i + rows) % rows
            sum += grid[col][row].state
    sum -= grid[x][y].state
    return sum


def main():
    running = True
    generation = 0
    grid = []
    
    rng = np.random.default_rng()

    # Initial grid:
    for j in range(rows):
        row_list = []
        for i in range(cols):
            row_list.append(Cell(j, i, cell_size, rng.integers(2), screen))
        grid.append(row_list)

    futureGrid = []

    while running and generation < maxGenerations:

        # poll for events
        # pygame.QUIT event means the user clicked X to close your window
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        for i in range(cols):
            for j in range(rows):
                grid[j][i].makeRect()

        futureGrid = []

        for j in range(rows):
            row_list = []
            for i in range(cols):
                # aliveNeighbours = countNeighbours(grid, i, j)
                state = grid[j][i].state
                aliveNeighbours = countNeighbors(grid, j, i)
                if (aliveNeighbours > 3 or aliveNeighbours < 2) and state == 1:
                    row_list.append(Cell(j, i, cell_size, 0, screen))
                elif (aliveNeighbours == 3) and state == 0:
                    row_list.append(Cell(j, i, cell_size, 1, screen))
                else:
                    row_list.append(Cell(j, i, cell_size, state, screen))
            futureGrid.append(row_list)

        for i in range(cols):
            for j in range(rows):
                futureGrid[j][i].makeRect()

        # swap grid
        grid = futureGrid

        # flip() the display to put your work on screen
        pygame.display.flip()
        generation += 1

        clock.tick(1)  # limits FPS

    pygame.quit()


if __name__ == "__main__":
    main()

```