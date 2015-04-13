title: "Linear Algebra in Room Escape"
date: 2015-04-12 12:26:54
tags:
---

## The "Light Out" game in room escape
Last weekend I went to play a reality room escape game with some of my friends. It's a lot of fun and we finally escape on time!

The only thing make it less perfect is that we skip a "very hard" puzzle according to the staff in the room. We spend 1O minutes on it and we could not found an effective way to solve it. 

The game is consisted of a board with 5 rows \* 5 columns = 25 lights. Each light is either on or off. Player could switch any light on/off, but switching any light will also switch it's neighbour on up/down/left/right position. The goal of this game is for a given status, try to switch some of the lights to make all the lights on.

For example, if the initial status looks like the following board (O mean an enlighted light and X mean an off light), then we could switch 2 lights on (1,0) and (3,2) to make all the light on.

| | | | | | | | | | | | |  | | | | | 
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|x | O| O| O| O| |O |O |O |O |O | | O |O |O |O |O |
|x | x| O| O| O| |O |O |O |O |O | |O |O |O |O |O |
|x | O| x| O| O|switch (1,0) => |O |O |x |O |O |switch (3,2) => |O |O |O |O |O |
|O | x| x| x| O| |O |x |x |x |O | |O |O |O |O |O |
|O | O| x| O| O| |O |O |x |O |O | |O |O |O |O |O |

You could also refer to this graph.

![Light out example](https://upload.wikimedia.org/wikipedia/commons/thumb/5/59/LightsOutIllustration.png/320px-LightsOutIllustration.png)

But not all cases are so straight forward. For example:

| | | | | | | | | | | | |  | | | | | 
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|x | O| O| O| O| |O |O |O |O |O | | O |O |O |O |O |
|x | O| O| O| O| |O |O |O |O |O | |O |O |O |O |O |
|x | O| O| O| O| OR |O |O |O |O |O |OR |O |O |O |O |O |
|O | O| O| O| O| |O |O |x |O |O | |O |O |O |O |O |
|O | O| O| O| O| |O |O |x |O |O | |X |O |O |O |X |

So is there an systemetic way to get a solution? When I am back to home, I do some research and I am a little bit supprise on its difficulty.

## Solution 1
This way is called light chasing. It's easy to follow the rules to get to the solution. The Principle of this solution is "normalize different boards into several solvable boards, and solved them with know strategies". 

Here is how it proceed:

1. rows are manipulated one at a time starting with the top row. All the lights are turned on in the row by toggling the adjacent lights in the next row. 
2. apply the same method on 2-4 row.
3. The last row is solved separately, depending on its active lights. Corresponding lights (see table below) in the top row are toggled and the initial algorithm is run again, resulting in a solution.

|Bottom row| switch Top row|
| - | - |
|XOOOX|XXOOO|
|OXOXO|XOOXO|
|XXXOO|OXOOO|
|OOXXX|OOOXO|
|XOXXO|OOOOX|
|OXXOX|XOOOO|
|XXOXX|OOXOO|

This approach always lead to an solution if there is any.

## Solution 2
The above solution seems not straight forward, how do I know I should pick that response? And yes, now we have a way to solve it. We will use a 3*3 grid for demonstration, after we have understand it, we could extend it to size with any size.

Simple inline $a = b + c$.

$$\frac{\partial u}{\partial t}
= h^2 \left( \frac{\partial^2 u}{\partial x^2} +
\frac{\partial^2 u}{\partial y^2} +
\frac{\partial^2 u}{\partial z^2}\right)$$

$$
\frac{\partial u}{\partial t} = h^2 \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} + \frac{\partial^2 u}{\partial z^2}\right)
$$

The Lights status are represented with matrix *L*. Here 1 means the lights are on, 0 for off. 
\\\\[ L = \\begin{vmatrix}
0 & 1 & 0 \\\\\\
1 & 1 & 0 \\\\\\
0 & 1 & 1
\\end{vmatrix} \\\\]

To make all the lights on, we should toggle some lights to generate effects of \\[ \overline{L} = \begin{vmatrix}
1 & 0 & 1 \\\
0 & 0 & 1 \\\
1 & 0 & 0
\end{vmatrix} \\]

The action of the switch placed at (i,j) can be interpreted as the matrix 
$$${A}_{ij}$$$ 

where $$ {A_i}_j $$ is the matrix in which the only entries equal to 1 are those placed at (i,j) and in the adjacent positions; there are essentially three di
