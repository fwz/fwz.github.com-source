title: "Linear Algebra in Room Escape"
date: 2015-04-12 12:26:54
categories:
- Game
- Room Escape
tags: 
- Linear Algebra
---

![](http://wenzhong.qiniudn.com/img/blog/lights_out_2.png)

Last weekend I went to play a reality room escape game with some friends. It's a lot of fun and we finally escape on time!

The only thing make it less perfect is that we skip a "very hard" puzzle according to the staff in the room. We spend 1O minutes on it and we could not found an effective way to solve it. 

The game is consisted of a board with 5 rows \* 5 columns = 25 lights. Each light is either on or off. Player could switch any light on/off, but switching any light will also switch it's neighbour on up/down/left/right position at the same time. The goal of this game is for a given status, try to switch some of the lights to make all the lights on.

You could also refer to this graph for the "switch logic".

![Light out example](https://upload.wikimedia.org/wikipedia/commons/thumb/5/59/LightsOutIllustration.png/320px-LightsOutIllustration.png)

To win the game, For example, if the initial status looks like the following board (O mean an enlighted light and X mean an off light), then we could switch 2 lights on (2,1) and (4,3) to make all the light on. But is there an systematic way to get a solution? 

<!--more-->

| | | | | | | | | | | | |  | | | | | 
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|x | O| O| O| O| |O |O |O |O |O | | O |O |O |O |O |
|x | x| O| O| O| |O |O |O |O |O | |O |O |O |O |O |
|x | O| x| O| O|switch (2,1) => |O |O |x |O |O |switch (4,3) => |O |O |O |O |O |
|O | x| x| x| O| |O |x |x |x |O | |O |O |O |O |O |
|O | O| x| O| O| |O |O |x |O |O | |O |O |O |O |O |

But not all cases are so straight forward. For example:

| | | | | | | | | | | | |  | | | | | 
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|x | O| O| O| O| |O |O |O |O |O | | O |O |O |O |O |
|x | O| O| O| O| |O |O |O |O |O | |O |O |O |O |O |
|x | O| O| O| O| OR |O |O |O |O |O |OR |O |O |O |O |O |
|O | O| O| O| O| |O |O |x |O |O | |O |O |O |O |O |
|O | O| O| O| O| |O |O |x |O |O | |X |O |O |O |X |


## Some Helpful Deduction

Before we further illustrate, there are 2 useful tips deduced from the game rule:
1. we don't need to switch a light for more than 1 time.
2. the sequence of switching does not matter.

## Solution 1: Brutal Enumeration
We enumerate all possible combination of switches based on the current status. And see whether there is possible solution. Base on the above deduction, this solution have a time complexity of {% math O(2^{mn}) %}, where m * n is the total number of lights. 
Calculation will be effective if m,n < 5. But for a square grid with m = n = 6, it's already quite a long time for a laptop.

## Solution 2: Linear Algebra Way
The following linear algebra approach is a more systematic way to solve it. In the following illustration, I will use a 3*3 grid for demonstration, after we have understand it, we could extend it to size with any size.

First of all, the Lights status are represented with matrix *L*. Here 1 means the lights are on, 0 for off. 
\\[ L = \begin{vmatrix}
0 & 1 & 0 \\\
1 & 1 & 0 \\\
0 & 1 & 1
\end{vmatrix} \\]

To make all the lights on, we should toggle some lights to generate effects of 
\\[ \overline{L} = \begin{vmatrix}
1 & 0 & 1 \\\
0 & 0 & 1 \\\
1 & 0 & 0
\end{vmatrix} \\]

start from **0** matrix.

The action of the switch placed at (i,j) can be interpreted as the matrix {% math {A}_{ij} %} , where {% math A_{ij} %} is the matrix in which the only entries equal to 1 are those placed at (i,j) and in the adjacent positions; there are essentially three types of matrices {% math {A}_{ij} %}, for different types of position (corner, edge, internal):

{% math_block %}
{A}_{ij}=
\begin{cases}
\begin{vmatrix}
1 & 1 & 0 \\
1 & 0 & 0 \\
0 & 0 & 0
\end{vmatrix}& \text{if i,j refer to top-left corner light}\\
\begin{vmatrix}
1 & 1 & 1 \\
0 & 1 & 0 \\
0 & 0 & 0
\end{vmatrix}& \text{if i,j refer to top-middle light}\\
\begin{vmatrix}
0 & 1 & 0 \\
1 & 1 & 1 \\
0 & 1 & 0
\end{vmatrix}& \text{if i,j refer to internal light}\\
\text{...}\\
\end{cases}
{% endmath_block %}

Every winning combination of moves can be expressed mathematically in the form:

{% math_block %}
\sum_{i,j} {x_{ij} } { {A}_{ij} } = \overline{L}
{% endmath_block %}

each coefficient $x_{ij}$ represents the number of times that switch (i,j) has to be pressed. According to our previous deduction, it could be only 1 or 0. And if we flatten the {% math {A}_{ij} %} into a vector, then we get the following equation:

{% math_block %}
\begin{vmatrix}
1 & 1 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\
1 & 1 & 1 & 0 & 1 & 0 & 0 & 0 & 0 \\
0 & 1 & 1 & 0 & 0 & 1 & 0 & 0 & 0 \\
1 & 0 & 0 & 1 & 1 & 0 & 1 & 0 & 0 \\
0 & 1 & 0 & 1 & 1 & 1 & 0 & 1 & 0 \\
0 & 0 & 1 & 0 & 1 & 1 & 0 & 0 & 1 \\
0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 & 0 & 1 & 1 & 1 \\
0 & 0 & 0 & 0 & 0 & 1 & 0 & 1 & 1 
\end{vmatrix} 

\begin{vmatrix}
{x_{11}} \\
{x_{12}} \\
{x_{13}} \\
{x_{21}} \\
{x_{22}} \\
{x_{23}} \\
{x_{31}} \\
{x_{32}} \\
{x_{33}} \\
\end{vmatrix}
=
\begin{vmatrix}
1 \\
0 \\
1 \\
0 \\
0 \\
1 \\
1 \\
0 \\
0
\end{vmatrix}
{% endmath_block %}

Pretty good, ** But how could we solve this equation? ** It's not a traditional linear equation, it's based on an (mod 2) operation or what we call "XOR". But the basic idea is the same, we just redefine the operation like "add", "multiply" and then compute the equation.

There is a lot of solution given in different languages, let's take a look at a python version provided by [pmneila](github.com/pmneila). I add some comment in the code for a (hopefully) easier understanding.

{% codeblock lightout_solver.py lang:python https://github.com/pmneila/Lights-Out/blob/master/lightsout.py lightsout.py %}
# coding: utf-8

"""
The following code based on
https://github.com/pmneila/Lights-Out/blob/master/lightsout.py
"""

from operator import add
from itertools import chain, combinations
import numpy as np
from scipy import ndimage

# First, we define operation in Galois Field (https://en.wikipedia.org/wiki/Finite_field)
# Here what we need is an Z/2Z (A mod 2 Galois Fields)
class GF2(object):
    """Galois field GF(2)."""

    def __init__(self, a=0):
        self.value = int(a) & 1

    def __add__(self, rhs):
        return GF2(self.value + GF2(rhs).value)

    def __mul__(self, rhs):
        return GF2(self.value * GF2(rhs).value)

    def __sub__(self, rhs):
        return GF2(self.value - GF2(rhs).value)

    def __div__(self, rhs):
        return GF2(self.value / GF2(rhs).value)

    def __repr__(self):
        return str(self.value)

    def __eq__(self, rhs):
        if isinstance(rhs, GF2):
            return self.value == rhs.value
        return self.value == rhs

    def __le__(self, rhs):
        if isinstance(rhs, GF2):
            return self.value <= rhs.value
        return self.value <= rhs

    def __lt__(self, rhs):
        if isinstance(rhs, GF2):
            return self.value < rhs.value
        return self.value < rhs

    def __int__(self):
        return self.value

    def __long__(self):
        return self.value

# Encapsulate operation for vectorization computation
GF2array = np.vectorize(GF2)

def gjel(A):
    """Gauss-Jordan elimination."""
    nulldim = 0
    for i in xrange(len(A)):
        pivot = A[i:,i].argmax() + i
        if A[pivot,i] == 0:
            nulldim = len(A) - i
            break
        row = A[pivot] / A[pivot,i]
        A[pivot] = A[i]
        A[i] = row

        for j in xrange(len(A)):
            if j == i:
                continue
            A[j] -= row*A[j,i]
    return A, nulldim

def GF2inv(A):
    """Inversion(逆) and eigenvectors(特征向量) of the null-space of a GF2 matrix."""
    n = len(A)
    assert n == A.shape[1], "Matrix must be square"

    A = np.hstack([A, np.eye(n)])

    B, nulldim = gjel(GF2array(A))

    inverse = np.int_(B[-n:, -n:])
    E = B[:n, :n]
    null_vectors = []
    if nulldim > 0:
        null_vectors = E[:, -nulldim:]
        null_vectors[-nulldim:, :] = GF2array(np.eye(nulldim))
        null_vectors = np.int_(null_vectors.T)

    print "inverse of matrix:"
    print inverse
    print "eigenvectors of matrix:"
    print null_vectors
    return inverse, null_vectors

def powerset(iterable):
    "powerset([1,2,3]) --> () (1,) (2,) (3,) (1,2) (1,3) (2,3) (1,2,3)"
    s = list(iterable)
    return chain.from_iterable(combinations(s, r) for r in range(len(s)+1))

def lightsoutbase(n):
    # base of the lights out problem of size (n, n)
    a = np.eye(n*n)
    a = np.reshape(a, (n*n, n, n))

    # construct the A_{ij} Matrix
    a = np.array(map(ndimage.binary_dilation, a))
    lo_base = np.reshape(a, (n*n, n*n))
    return lo_base

class LightsOut(object):
    """Lights-Out solver."""
    
    def __init__(self, size=5):
        self.n = size
        self.base = lightsoutbase(self.n)
        self.invbase, self.null_vectors = GF2inv(self.base)
    
    def solve(self, b):
        b = np.asarray(b)
        assert b.shape[0] == b.shape[1] == self.n, "incompatible shape"
        
        if not self.issolvable(b):
            raise ValueError, "The given setup is not solvable"
        
        # Find the base solution.
        first = np.dot(self.invbase, b.ravel()) & 1
        
        # Given a solution, we can find more valid solutions
        # adding any combination of the null vectors.
        # Find the solution with the minimum number of 1's.
        solutions = [(first + reduce(add, nvs, 0))&1 for nvs in powerset(self.null_vectors)]
        final = min(solutions, key=lambda x: x.sum())
        return np.reshape(final, (self.n,self.n))
    
    def issolvable(self, b):
        """Determine if the given configuration is solvable.
        
        A configuration is solvable if it is orthogonal to
        the null vectors of the base.
        """
        b = np.asarray(b)
        assert b.shape[0] == b.shape[1] == self.n, "incompatible shape"
        b = b.ravel()
        p = map(lambda x: np.dot(x,b)&1, self.null_vectors)
        return not any(p)

if __name__ == "__main__":

    b = np.array([[1,0,1],
                  [0,0,1],
                  [1,0,0]])
    print "\nlights status:"
    print b

    lo = LightsOut(3)

    bsol = lo.solve(b)

    print "\nThe solution of"
    print b
    print "is"
    print bsol

{% endcodeblock %}

run it and it give output like:
{% codeblock %}
$ python lightsout_solver.py

lights status:
[[1 0 1]
 [0 0 1]
 [1 0 0]]
inverse of matrix:
[[1 0 1 0 0 1 1 1 0]
 [0 0 0 0 1 0 1 1 1]
 [1 0 1 1 0 0 0 1 1]
 [0 0 1 0 1 1 0 0 1]
 [0 1 0 1 1 1 0 1 0]
 [1 0 0 1 1 0 1 0 0]
 [1 1 0 0 0 1 1 0 1]
 [1 1 1 0 1 0 0 0 0]
 [0 1 1 1 0 0 1 0 1]]
eigenvectors of matrix:
[]

The solution of
[[1 0 1]
 [0 0 1]
 [1 0 0]]
is
[[0 1 0]
 [0 1 0]
 [1 0 0]]

{% endcodeblock %}

To solve this equation to get those coefficent {% math x_{ij} = 1 %} , we get a solution {% math [x_{12}, x_{22}, x_{31}] %}.

| | | | | | | | | | | | | | | |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|X | O| X|  | O|X|O | |O |O |O | | O | O | O |
|O | O| X| switch (1,2) => | O| X |X |switch (2,2) => |X |O |O |switch (3,1) => | O | O | O |
|X | O| O|  | X|O |O | |X |X |O | | O | O | O |

## Solution 3: Light Chasing
The above solution is effective, but it's also crazy to ask someone to solve a equation with 25 parameters in a Room Escape Game. The following approach is easy to follow and to get the solution. The Principle of this solution is "normalize different boards into several solvable boards, and solved them with known strategies". 

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

This approach always lead to an solution (if there is any) for a 5 * 5 Grid. This might be the ideal solution when trapped in the game :).

