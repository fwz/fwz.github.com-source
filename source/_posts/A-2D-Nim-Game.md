title: "A 2D Nim Game"
date: 2018-02-21 21:15:26
tags: [Game Theory]
categories:
- Game
- Board Game
---


### Intro

This article is about a Nim game played around 1997 (LOL). Allow  me to introduce the rule of this game:

* There are 16 stones, arranged in 4 row * 4 column grid
* Each player take stones in turn
* Once the stone is taken, there is a gap on it's original location on the grid
* In each take, one should take no more than 3 stones (1/2/3). Only those in the same row or in the same column without gap between them could be taken.
* Player who takes the last stone *LOSE*

As mentioned, it's a 4*4 grid. Assuming `1` means there is stone on the specific cell and `0` represent a gap in the following chart.

* the opening could be represented as (I)
* the ending could be represented as (II)
* one of the losing situation (LS) for current turn player could be represented as (III) 

```
1111   0000   0000
1111   0000   0000
1111   0000   0010
1111   0000   0000
(I)    (II)   (III)
```

To be more specific, let me demonstrate a possible game.

<!--more-->
```
Org    (A)    (B)    (A)    (B)    (A)    (B)    (A)    
1111   1110   1110   0110   0110   0110   0000   0000
1111   1110   1110   1110   1110   1110   1110   0000
1111 > 1111 > 1000 > 1000 > 1000 > 0000 > 0000 > 0000
1111   1111   1111   1111   1100   0100   0100   0100
```

### Initial analysis

According to my primary playing experience, the following situation is to be LS (and welcome to try it). 

```
1100   1001   1011   1010
1100   1001   0111   1010
0000   1001   0000   0000
0000   0000   0000   0000
```

Believe it or not, knowing this situation make you 99% unbeatable in primary school.

But we want to know the essence of this game. So let's do some deep dive in 20 years later.

Instead of staying on the number axis, such as some basic Nim game [subtraction problem](https://en.wikipedia.org/wiki/Nim#The_subtraction_game_S(1,_2,_._._.,_k)) , this game is in a 2D space, and there are a lot of variation in each take. The first thought come to me is that this game is about graph theory and the situation analysis becoming connected graph analysis since removing stones require connectivity. After a simple estimation of my poor graph theory knowledge, I give up this direction shamelessly and try to analyse the state space of this game.

Thanks to the simple nature of this game (and primary school), this game are consist of only 2^16 = 65536 states because there are 2 states for each stone and there are 16 stones. 

And we know that the following 16 states are absolutely going to lose the game. If we could leave this 16 situations to our opponent, then we could win the game. 

```
1000  0100  0010  0001  0000  0000  ...  0000  0000
0000  0000  0000  0000  1000  0100  ...  0000  0000
0000  0000  0000  0000  0000  0000  ...  0000  0000
0000  0000  0000  0000  0000  0000  ...  0010  0001
```


So any state that transfer to these 16 states in one move, will be our winning state because if we get any of those state, then we have at least one way to transfer this state to the above lose state for our opponent. For example, these states are some of the winning state towards the first state in above situations.

``` txt
1100 1110  1111  1000  1000  1000
0000 0000  0000  1000  1000  0100
0000 0000  0000  0000  1000  0000
0000 0000  0000  0000  0000  0000  ...
```

After calculate all such states, we can easily get 600+ win states.

Great, but what would be the next? Any state could transfer to a win-state in one move is lose-state? Not necessarily, check the first 2 state `1100` and `1110`in the above graph. `1110` can transfer to `1100` but it still win because we could choose to leave only `1000` to our opponent.

But it give us a hint for the deduction, for two smart enough player,
> if **ANY** next state of current state **LOSE**, current state **WIN**

And with further consideration,
> if **ALL** next state of current state **WIN**, current state **LOSE**

And this could be test quickly with previous lose-state. The ZERO state, and player get this state, means his opponent take the last stones, so ZERO state is a winning state. The previous 16 states have only 1 next state, the ZERO state. So they are indeed the lose state.

Seems we are making progress, but how to expand the lose-state / win-state set.

### Zermelo's Theoerem 

First of all, we take a look the following concept:

* [Zero sum Game](https://en.wikipedia.org/wiki/Zero-sum_game)
* [Perfect information Game](https://en.wikipedia.org/wiki/Perfect_information) 

> Chess is an example of a game with perfect information as each player can see all of the pieces on the board at all times. Other examples of games with perfect information include tic-tac-toe, checkers, infinite chess, and Go. A game has perfect information if each player, when making any decision, is perfectly informed of all the events that have previously occurred.

Our game is a typical case of zero-sum perfect-information game. In such game, for any state, one side of the game will have a series of strategy that could definitely win the game according to [Zermelo's Theorem](https://en.wikipedia.org/wiki/Zermelo%27s_theorem_(game_theory)). 

Zermelo's theorem give us a fairly big help on solving our game, because it help prove the following statement:

> for ANY state in our game, either the current player could force a win, or the next player could force a win.

Here are how it comes: for each state, it can be treat as the opening of  a game with perfect information, so it's either.

### Solution generation
Then, things will be come easier, we are able to iterate all the state through the current principle:

1. transfer every possible state to number via binary notation. For example the following state could be converted as (1111000011111111)2 = 61695
``` txt
1111   
0000
1111 
1111    
```

2. define state 0 as winning state since the opponent must take the last stone.

``` python
    lose_set = set()
    win_set = set()

    # init
    win_set.add(0)
```

3. Now the iteration
``` python

    lose_set = set()
    win_set = set()

    # init
    win_set.add(0)

    # start calc
    for i in range(1, 65536):
        flag = True

        # if all next step of current board is a win board, current board lose
        for board, number in find_all_adj_board(num2board(i), 1):
            if number not in win_set:
                flag = False
                break
        if flag:
            lose_set.add(i)
        else:
            # must be win or lose, according to Zermelo
            win_set.add(i)
```

Easy, right? The nice thing here is that, since we iterate from a state with smaller binary, all next states have been computed because some "1" in the binary will change to "0", but no "0" will change to "1". so the number represented the next possible state board will always be smaller. 
The upper bound of count for next states is 4\*\4\*2\*3, because for any possible cell, one could remove 3 types of length for 2 directions (down / right)

To generalize, for an N\*N Grid with above rules, this algorithm take O(2^(N\*N)\*2\*3\*N\*N) to iterate all possible states. If N = 4, it takes 30 seconds to generate all states on my computer. so it might take 12 hours for N = 5, and much more for N = 6.

If you want to know whether the first player or the second player could force a win, checkout the [game script I wrote: 2d-nim](https://github.com/fwz/2d-nim/) and test it!

### Summary
We discuss the 2d nim game and the Zermelo's Theorem in this article to get the ultimate strategy for this game. Although we reached the core of the  answer, I still believe there are some topology based solutions with a lower complexity.


