# hanoitoys Package
last change 08-Feb-2022
### A collection of Tower of Hanoi solutions.
aka Tower of Lucas, 
aka Tower of Brahma, 
mostly toy programs showing non recursive solutions to the Classical (3 pegs/needles) Hanoi.  
One solution generator to Super Hanoi aka Hanoi on steroids.
This is the Tower of Hanoi with more than 3 pegs/needles. This is the Steward Frame algorithm.
This has an optional turtle grapics display.


## The toy solutions
### HanoiClassical.py
This is the classical Hanoi solution using recursion. The CS 101 method.  
#### 1st the classical Tower of Hanoi
The classical Tower of Hanoi has 3 posts or needles and n discs. One of the posts is where all n discs are located at when the puzzle starts. This post 
is called source.
The goal is to move all the discs to the destination peg. Subject to 3 rules.  
Rule # 1 : A larger disc is never placed on top of a smaller disc.  
Rule # 2 : Only one Disc may be moved at a time.  
Rule # 3 is that in that the sequence of moves leading to the solution may not have any repeated states.  
There is another disc labeled Store.   


This leads us to the fact there is only one sequence of states that leads to a solution for classical Tower of Hanoi.  

In a second sense this is the classical most known solution, the recursive solution.  
The base case is you can move one disk from x to y when y does not have any smaller disc.
Each level of recursion does relabel the pegs. The former labeling is restored by returning.
```
def recursive_hanoi (source, store, destination, n):
    if n == 1:
        move the one disc
        return
    recursive_hanoi (source, destination, store, n - 1
    recursive_hanoi (source, store, destination, 1)
    recursive_hanoi (store, source, destination, n - 1)
```
#### HanoiOlive.py is a nonrecursive Olive algorithm.
Francois Edourard Lucas who wrote about this problem and gave it the Tower of Hanoi name (writing under the nom de plume of M Claus, Claus being an anagram of Lucas) 
. Raoul Olive, a nephew of Lucas, observed the smallest
disc moves every other move. The motion of this peg depends on the parity of n.  
When n is odd the smallest disc moves from source to destination to store, then back to source.  
When n is even the smallest disc moves from source to store to destination, then back to source.  
When the smallest disc is not being moved there is only 1 legal move involving the two pegs the smallest disc is not occupying.
#### HanoiIdlePeg.py is another non recursive refinement.
Here we add a marker which is like a smaller disc which marks the Idle (non participating peg). With a real puzzle with discs and wooden pegs you can often
let a sewing thimble represent the Idle peg. We do not count the Idle Peg token in the number of discs n.  
For Odd n we move clockwise source -> store -> destination -> source.  
For Even n we move counter clockwise source -> destination -> store -> source.  
we repeatedly move the Idle token then find the legal move that can be made with the remaining participating pegs.

#### HanoiNR.py
HanoiNR.py is also included. This is the Wikipedia non recursive Tower of Hanoi solution.
## Tower of Hanoi on steroids. Number of pegs > 3. The Steward, Frame Algorithm.  CS 501
While Francois Edourard Lucas aka M Claus did undoubtedly write on the more than three peg Tower of Hanoi, we will now visit that problem disguised as the Reve's
Puzzle. In 1907 Henry Ernest Dudeney wrote a book of mathematical puzzles called "*The Canterbury Puzzles and Other Curious Problems*". In this work the characters are from Chaucer's "*The Canterbury Tales*" are on a pilgrimage and resting at a tavern where each of them presents a puzzle to their fellow travelers. The first of these is the Reve's[^1] puzzle. In the Reve's puzzle the are 4 stools and 8 cheeses of various sizes. These are on a stool in order, the smaller the cheese the closer to the top of the stack. The puzzle is to move the cheeses to another stool in the same order. Only one cheese can be moved at a time. A cheese can only be moved to an empty stool or the top of a larger cheese. Only one cheese can be moved at a time.

Clearly  this is equivalent to a 4 peg/needle 8 disc Tower of Hanoi problem.

### Reve's puzzle being solved by Dan's Hanoi (Hanoi.py in this package)

![Reve's puzzle being solved by Dan's Hanoi](Reves.png)

In 1941 B. M. Stewart and J. S. Frame each independently came up with
a algorithm to solve the generalized (n > 3) Tower of Hanoi puzzle. This
partitioned the initial stack of disks into sequential groups. The
subproblems being to move each group to a peg (a recursive task) and
reassemble them on the destination peg/needle.  They believed this
solution was 'Optimal'. This is known as the Frame Stewart conjecture.
In 2018 this conjecture was proved by Roberto Demontis.  
#### What is Optimal?
  In classical (3 peg/needle) Hanoi the constraint to not
repeat a state means there is only one solution. That solution is
optimal, there are no solutions using fewer moves because there are no
other solutions period. The optimal solution has 2^n^-1 solutions.
This is not true when you go beyond 3 pegs. There are many solutions,
even when we keep the no repeated state requirement. The ones where no
other solutions that take less moves are optimal. Clearly you can
remap all pegs/needles other than the source and destination in (n-2)!
ways. And at each sub problem you can remap. There are therefore many
Optimal solutions.  

#### The solutions take far fewer moves than the classical Tower of Hanoi does.
Consider the 64 disc 4 needle
solutions: You could just not use one needle and solve this
classically. Or you could move the top disc to a needle and solve the
rest as a 63 disc classical Tower of Hanoi. This would take a tiny bit
more than half the moves of the 64 disc classical Tower of Hanoi
solution. Or move two discs and then 62 classical, which would be a
fourth the moves. The first few n values of this are: 
|n|Number of moves| 
|---|------------------------| 
|0|18,446,744,073,709,551,615|
|1|9,223,372,036,854,775,809|
|2|4,611,686,018,427,387,909|
|3|2,305,843,009,213,693,965|
|4|1,152,921,504,606,847,005|
|5|576,460,752,303,423,549|
At n=53 we do the best we can at 18,433 moves.  

#### We need an estimate for cost.  
We can guide our algorithm by a Cost[^2] Estimator. 
It too will be recursive. We just have to
know how many moves, not actually make the moves. However the
estimator has to not only tell us how few moves we can get away with
but what direction we have to go to get there.  
Here is an estimator:
``` 
@functools.lru_cache(maxsize=3000) 
def est (NumDiscs: int, NumStore: int) -> tuple: 
    # returned tuple (num of moves, first move this many to storage needle 
    # can use min on list of these tuples to get tuple to return.  
    # or alternatively track the minimum without a list 
    if NumStore == 1 or NumDiscs < 3:
       # base case is classic Hanoi.
       return ((1 << NumDiscs) - 1, 1) # 2 ** NumDiscs - 1; this is faster.
    possibilities = list() 
    # build a list of sub problems 
    for n in range(1, NumDiscs):
	moves = est(n, NumStore)[0] << 1		# n src->stor, n
       stor->dest moves += est(NumDiscs - n, NumStore - 1)[0] # NumDiscs-n
       src->dest possibilities.append((moves, n))	
       # ok in list see if it is best later 
       return min(possibilities) 
```
This recursive solution is of order n factorial O(n!) Far worse for n > 3 than say Fibbinacci
O(2^n^). If you try this for yourself best use *memoization*.  The
lru_cache decorator from functools is a godsend. Fail to do this means
the mean time to fail for the hardware may foil your attempt.  
The estimator 'est' returns a tuple, the first element is the number of
moves (the thing we want to minimize), The second is the number of
discs from the top of the source peg/needle to move to one of the available storage
peg/needles.

 As to which available peg/needle it does not matter.
Initially this was coded with the Python set object to hold the
available storage pegs, but to make it seem as if it was not just
guessing, I implemented a ordered set which is just an int with bits
set for pegs.  Note: At a any particular point in the recursion some
of the available pegs may have discs on them, only they will be bigger
discs then the ones you are trying to move.  

#### Labeling the pegs/needles.

When there are more than 3 needles, calling them source, store and destination no longer works.
The Hanoi.py in the package names them first after the English alphabet Uppercase. The 27th is lower case 'a'.
We continue on through the lower case letter to the 52nd which is lower case zed 'z'. Needles 53 through 76 are given
Greek letters, which are spelled out in English in the normal output and use unicode equivalents in the graphics.


The source at the top level is always labeled 'A'. The destination is last needle, if it were the 64th it would be 'nu'.

#### The Dialogue.

The Dialogue is not a Turtle dialogue or a tkinter dialogue. This is a default stdin/stdout dialog. Turtle is not even initialized until
we know we need a graphical output. There are two flavors: text mode output and graphics. The graphics dialog needs the same information and
a little bit more than the text dialog. So the last question of the text mode dialogue is "Do you want a graphical representation?" to which
a reply of the single character 'n' is acceptable. The other questions are "Enter Number of Needles :" and
"Enter Number of Discs :".


The example shown sets up for a textual only solution to the Reve's puzzle.
```
Enter Number of Needles : 4
Enter Number of Discs : 8
Do you want a graphical representation?n
1:Moving disc from A to C
2:Moving disc from A to B
3:Moving disc from A to D
4:Moving disc from B to D
5:Moving disc from A to B
6:Moving disc from D to A
7:Moving disc from D to B
8:Moving disc from A to B
9:Moving disc from C to B
10:Moving disc from A to C
11:Moving disc from A to D
12:Moving disc from C to D
13:Moving disc from A to C
14:Moving disc from D to A
15:Moving disc from D to C
16:Moving disc from A to C
17:Moving disc from A to D
18:Moving disc from C to D
19:Moving disc from C to A
20:Moving disc from D to A
21:Moving disc from C to D
22:Moving disc from A to C
23:Moving disc from A to D
24:Moving disc from C to D
25:Moving disc from B to A
26:Moving disc from B to D
27:Moving disc from B to C
28:Moving disc from D to C
29:Moving disc from B to D
30:Moving disc from C to B
31:Moving disc from C to D
32:Moving disc from B to D
33:Moving disc from A to D
```
Most people will want to answer a single lower case 'y' to the graphics question. The graphics display has the 'A' (source) needle on
the lower left hand side of the display and the Destination needle on the right hand side. There are never any needles above the Destination needle. The whole right hand column except for the destination needle is reserved to move discs vertically to the desired row. This area will be referred to as the 'elevator' In each row there is a little space above the needle for horizontal disc motion. These will be referred to as 'skyways'.

When the graphic dialog asks "How many Columns :" This always excludes the destination needle. On the bottom row there is an extra column. 
The program figures out how many rows it needs. An example needles=8, discs=11, columns=4 will have two rows. The bottom row with 'A', 'B', 'C', 'D' and 'H'.
The top row with 'E', 'F', 'G'.

The next question is "Do you want a faster move of discs?". A 'n' is usually called for. If you want the discs to move in a straight line from where they start to where they are going regardless of what is in the way than a 'y' response is appropriate. This speeds up some puzzles by quite a bit. 

#### The graphics option.

The fast move from A to X is from the top disc on needle A to the place on needle X above the top disc in a straight line.

The normal move is somewhat involved. Where A and X are on same row A's top disc ascends the skyway, moves left or right to just above needle X and 
descends into place.
On different rows A'a top disc ascends to the skyway, moves left to the elevator ascends or descends to the skyway for the row X is on, moves left till it 
is over X and descends into place.

Here is a sample picture captured for the 8 Needle, 11 discs, 3 column example:
![8 Needle, 11 discs, 3 column](N8D11C4.png)

#### Things you might want to try.

In classical Tower of Hanoi with 3 needles and 64 needles, the Sun would become a Red Giant, Earth incinerated and the Milky Way would have collided with Andromeda and you still would not have a solution. But with 4 needles, 64 Discs and no graphics, this takes maybe 7 seconds.


Another to try takes only 127 moves. This is 65 needles, 64 Discs, 8 columns and and fast graphics. This does exactly what you might think. There is no special case for this, it falls out of the more general solution. It is odd the number of moves with 3 needles is 2^n^-1, but with n+1 needles it is 2n-1.

#### For more information.
I highly recommend the following book in either 1st or 2nd edition:  
*The Tower of Hanoi -- Myths and Maths *  
Authored by:  
*Andreas M. Hinz, Sandi Klavzar, Uros Milutinovis, Ciril Petr*  
Published by Birkhauser  
First Edition ISBN 978-3-0348-0237-6  
[Website (unfortunately **unsecure** http) tohbook.info](http://tohbook.info)  
[Widipedia article about the book](https://en.wikipedia.org/wiki/The_Tower_of_Hanoi_%E2%80%93_Myths_and_Maths)


This material is found in a much expanded form in a chapter of a book I have written. I will update this notice when it is published.  
 
[^1]: Reve from the word Reeve: honored official. The Reeve of the Shire is the source of the modern word Sheriff.  
[^2]: Luke 14:28 For which of you, intending to build a tower, sitteth not down first, and counteth the cost, whether he have sufficient to finish it?  
