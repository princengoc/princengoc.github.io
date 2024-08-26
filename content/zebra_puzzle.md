title: who-owns-the-zebra
date: 2024-08-15
tags: prolog, gpt4, zebra puzzle, seven languages, recursion, books are the best
description: my prolog aha moment, ChatGPT is a terrible teacher, why that's not good, and why that's not-not-good. 

# Who owns the zebra? 

A year ago, I asked ChatGPT to teach me [Prolog](https://www.swi-prolog.org/) via games. It recommended the [Zebra puzzle](https://en.wikipedia.org/wiki/Zebra_Puzzle). Excellent choice. A couple of hours in, and I learned some valuable lessons. 

1. ChatGPT of August 2023 was terrible at Prolog. It has no sense of logic and programming, but excels at convincing you of its "expertise". Five minutes in, every code snippet it wrote was faulty. Presumably because there's not a lot of training data for it to cut-and-paste from, unlike, say, Javascript or Python. 
2. Having a terrible clueless teacher spouting half-baked knowledge is actually **not so bad**. It thrusted me  into debugging mode, and through that I learned some amount of Prolog in a very short time. 

But there's only so much you can learn from constantly putting out code fires. I never got my Zebra puzzle solver up and running. Instead, I became a tool to relay error messages from the compiler to ChatGPT. I left feeling grumpy, my time wasted by a scammer. A few days later, I promptly forgot all the Prolog along with the suffering.


Turns out, a good teacher is still **better**. A couple of weeks ago, I dusted up my copy of [Seven languages in seven weeks](https://pragprog.com/titles/btlang/seven-languages-in-seven-weeks/) and did the traditional thing: learn line-by-line, do all the exercises, read APIs. I left each lesson feeling stretched, satisfied, **learned**. After a couple of days, I got my Zebra figured out. Most importantly, I got a feel for the language. 

### Lesson 1. Thou shalt think. 

``Prolog`` is efficient exhaustive search. How you setup the problem is life and death. It is the difference between ``OMG it works, amazing!!`` vs ``Are.We.There.Yet???`` (no we're not, but hey the computer fan is running so it's computing... Oh it says ``false.`` FALSE? What do you mean False? Which of these constraints are infeasible? How do I even debug this???). 

Or, less dramatically, problem setup is the difference between ``a lot of cut-paste-rename`` and ``awesome I'm so clever``. Since I'm lazy at cut-and-paste, Prolog really pushes me to find better solutions. This makes programming in Prolog quite fun: the lazy-but-clever me had a chance to kickass my whiny hardworker alter ego.

I had three iterations of the zebra puzzle.

#### Zebra 1: plain prolog

I wanted to use as primitive constructs as possible. I got a toy program running, but then quickly pivoted to ``Zebra 2`` when it was clear that a lot of cut-paste-rename was needed for the code to work. 

The heart of the program was a representation of ``valid_pairs``. For example,

```prolog
valid_color_nation(red,england). 
valid_color_nation(X,Z) :- 
  X \= red, 
  Z \= england.
```
These pairs rules are stitched together like this
```prolog
valid_constraints(X,Y,Z) :- 
  %pairwise valid
  valid_xy(X,Y), 
  valid_yz(Y,Z), 
  valid_xz(X,Z).

%valid_constraints_map/3 does a row-wise constraint validation
%the 3 arguments are list of Colors, Nations, Pets, for example
valid_constraints_map([],[],[]).
valid_constraints_map([HeadX|TailX], [HeadY|TailY], [HeadZ|TailZ]) :- 
  valid_constraints(HeadX, HeadY, HeadZ), 
  valid_constraints_map(TailX, TailY, TailZ).
```

This is quite brittle and clearly doesn't scale.

#### Zebra 2: use_module(library(cplfd))

Seven Languages chapter on Prolog is very good, but it is dated. Per the hot [Prolog-on-Rust](https://www.scryer.pl/) project,

> However, most of (past Prolog books) are not updated to *modern* Prolog. We recommend The Power of Prolog (Markus Triska) for modern Prolog. 

I still don't understand what's Modern vs Old Prolog (perhaps a guru can enlighten me). But digging around modern Prolog took me to ``clpfd`` (Constraint Logic Programming over Finite Domains). Certainly this should kill the Zebra. 

I defined the puzzle as tuple ``Colors, Smokes, Pets, Drinks, Nations``, with
```prolog
Colors = [C1,C2,C3,C4,C5] 
%C1 = color of the first house, ... C5 = color of the 5th house.
```

This turned out to be a disaster (see Zebra 3 below). Tunnel vision and the patience of a beginner let me jumped through a lot of hoops to get the program done. Clever? No. Shortest path? No. More learning? Yes. 

The library ``cplfd`` has nice iff syntaxes, like ``(Nation #= England) #<==> (Color #= Red),``. (Englishman lives in the red house). Unfortunately, the domain must be integers 1 to 5. I don't want to write ``(Nation #= 1) #<==> (Color #= 1)`` because a/ I'm lazy, and b/ I'm error-prone. I thought I could be clever by writing
```prolog
rule_1(X, Y) :- 
  nation(X, england) #<==> color(Y, red)
```
but nope, doesn't work, LHS and RHS must be stuff that can be [reified](https://www.swi-prolog.org/pldoc/man?section=clpfd-reification). I tried dynamically changing #<==> (nope), dynamically inserting rules (maybe?). In the end, I wrote a prolog function to print out these new rules to a file so that I can cut-and-paste. 

```prolog
write_rules(Predicate1, Predicate2) :-
    open('rules.pl', append, Stream),
    findall(_, 
        (Predicate1, Predicate2, format_rule(Stream, Predicate1, Predicate2)), 
        _),
    close(Stream).
```
I'll spare you the ugly formatting codes. The point is, I can write this
```prolog
write_rules(nation(X, england), color(Y, red)),
write_rules(nation(X, spain), pet(Y, dog)), 
```
and outcome this in a file
```prolog
  (Nation #= 1) #<==> (Color #= 1),
  (Nation #= 2) #<==> (Pet #= 1),
```
Excellent, error-free, though it certainly took much longer than cut-and-paste, and I still need to refer back to my rules table like
```prolog
color(1, red).
color(2, green). 
color(3, yellow). 
color(4, ivory). 
color(5, blue). 
```
to make sense of the output. Furthermore, not all rules can be expressed this way. For example, Rule 6. 

> Rule 6. The green house is to the right of the ivory house. 

I solved this essentially by considering all positions where the green house could be. 
```prolog
rule_6(ColorLeft, ColorRight) :- 
  (ColorLeft #= 2) #<==> (ColorRight #= 4).

rule_6_map([C1, C2, C3, C4, C5]) :- 
  maplist(rule_6, [C1, C2, C3, C4], [C2, C3, C4, C5]).

%inside the solver einstein(Colors, ...), have this line
    rule_6_map(Colors), 
```

Most troublesome are rules 11 and 12.

> Rule 11. The man who smokes Chesterfields lives in the house next to the man with the fox.
>
> Rule 12. Kools are smoked in the house next to the house where the horse is kept.

My initial implementation was
```prolog
rule_11([],[]).
rule_11([S1|S_Tail], [P2|P_Tail]) :- 
  (S1 #= 2) #<==> (P2 #= 2), 
  rule_11(S_Tail, P_Tail).

%this rule get called inside the solver as
    Smokes = [S1,S2,S3,S4,S5],
    Pets = [P1,P2,P3,P4,P5],
    %forward shift for rule 11
    rule_11([S1,S2,S3,S4],[P2,P3,P4,P5]),
    %backward shift for rule 11
    rule_11([S2,S3,S4,S5],[P1,P2,P3,P4]),
```

Clever recursion? Well, the program kept on spitting out ``false.``. Some head scratches later, I figured out why. The forward and backward shifts together actually imply
```prolog
(S3 #= 2) #<==> (P4 #= 2) #<==> (P2 #= 2)
```
but because the pets need to be all distinct, the problem is infeasible. 

I tried to fix this with the ``and`` and ``or`` operators, like
```prolog
rule_11([S1|S_Tail], [P2|P_Tail]) :- 
  (S1 #= 2 #/\ P2 #= 2) #\/ rule_11(S_Tail, P_Tail).
```
but tail-recursions cannot be part of the expresion. At this point, I gave up and write out the whole thing. But for a smaller toy Zebra puzzle. :) Gotta cut corners. 

```prolog
%smaller version of Zebra 2 

:- use_module(library(clpfd)).

% colors, pets and drinks order, just for reference. Not used. 
colors([red, green, blue]). 
pets([dog, cat, bird]).
drinks([milk, water, juice]). 

%rules
%1. The Red house is on the left of the Blue house.
%2. The person in the Red house drinks Water.
%3. The person with the Dog lives in the Green house.
%4. The person who has the Bird lives next to the person who drinks Milk.
%5. The person who drinks Juice lives next to the person with the Cat.

relative_rules(Color, Pet, Drink) :- 
  (Color #= 1) #<==> (Drink #= 2), 
  (Pet #= 1) #<==> (Color #= 2). 

%shifting rules
rule_one([C1,C2,C3]) :- 
  (C1 #= 1) #<==> (C2 #= 3), 
  (C2 #= 1) #<==> (C3 #= 3). 
  
rule_four([P1,P2,P3], [D1,D2,D3]) :- 
  (P1 #= 3 #/\ D2 #= 1) #\/
  (P2 #= 3 #/\ D3 #= 1) #\/  
  (P2 #= 3 #/\ D1 #= 1) #\/    
  (P3 #= 3 #/\ D2 #= 1).


rule_five([P1,P2,P3], [D1,D2,D3]) :- 
  (P1 #= 2 #/\ D2 #= 3) #\/
  (P2 #= 2 #/\ D3 #= 3) #\/  
  (P2 #= 2 #/\ D1 #= 3) #\/    
  (P3 #= 2 #/\ D2 #= 3).
  
einstein(Colors, Pets, Drinks) :-
  Colors = [C1, C2, C3], 
  Pets = [P1, P2, P3], 
  Drinks = [D1, D2, D3], 

  append([Colors, Pets, Drinks], Vars), 
  Vars ins 1..3, 
  
  maplist(all_distinct, [Colors, Pets, Drinks]),   
  
  maplist(relative_rules, Colors, Pets, Drinks), 
  
  rule_one(Colors), 
  rule_four(Pets, Drinks), 
  rule_five(Pets, Drinks), 
  
  label(Vars).
```

Program works, dully solves the puzzle, as expected. If I could just symbolically replace ``C1 #= 1`` with ``C1 #= dog``, that would be much easier to read. 

#### Zebra 3. A small rewrite made all the difference. 

I still represent the puzzle as a tuple ``Colors, Smokes, Pets, Drinks, Nations``. But now ``Colors = [Red, Green, Yellow, Ivory, Blue]``, and ``Red #= 2`` means the 2nd house is red. So it's just the inverse map of ``Zebra 2``: instead of mapping the house number to the color (Zebra 2), I map the color to the house number (Zebra 3). This makes a **HUGE** quality-of-life difference. 

```prolog
%classic zebra puzzle
%Red = 2 means 2nd house is red. 

:- use_module(library(clpfd)).

einstein(Colors, Smokes, Pets, Drinks, Nations) :- 

  %sensible names
  Colors = [Red, Green, Yellow, Ivory, Blue], 
  Smokes = [Old_Gold, Chesterfields, Kools, Parliaments, Lucky_Strike], 
  Pets = [Dog, Fox, Horse, Snails, Zebra], 
  Drinks = [Milk, Tea, Coffee, Juice, Water], 
  Nations = [England, Spain, Ukraine, Norway, Japan],
  
  %could maplist here but I prefer verbose in the delirium happiness of beautiful names
  Colors ins 1..5, 
  Smokes ins 1..5, 
  Pets ins 1..5,
  Drinks ins 1..5, 
  Nations ins 1..5, 
  
  maplist(all_distinct, [Colors, Smokes, Pets, Drinks, Nations]),
  
  %rules are TRIVIAL

  England #= Red, 
  Spain #= Dog, 
  Coffee #= Green, 
  Ukraine #= Tea, 
  Green #= Ivory + 1, 
  Old_Gold #= Snails, 
  Kools #= Yellow, 
  Milk #= 3, 
  Norway #= 1, 
  abs(Chesterfields - Fox) #= 1, % rule 11
  abs(Kools - Horse) #= 1, % rule 12
  Lucky_Strike #= Juice,
  Japan #= Parliaments, 
  abs(Norway - Blue) #= 1, 

  %print out solution  
  append([Colors, Smokes, Pets, Drinks, Nations], Vars), 
  label(Vars).  
  
  % DONE!!! Yay. 
```

Yay indeed. 

```
?- einstein([_,_,_,_,_], [_,_,_,_,_], [_,_,_,_, Zebra], [_,_,_,_, Water], [_,_,_,_,_]).
Zebra = 5,
Water = 1 ;
false.
```

There we go. ``Zebra`` is in house 5, ``Water`` is in house 1. All done. It also lends clarity to the problem. We started with 25 variables on an integral cube. Then we cut the set of feasible solutions with a bunch of hyperplanes. No wonder we are left with a single solution. More complicated constraints can easily be implemented, though by then the puzzle is too complicated, the human would have lost interest. 


### Lesson 2. Recursion RULES. 

Writing ``Prolog`` reminded me of writing ``Scala``: a lot of tail recursion. But there's recursion, and then there's ``clever recursion``. Compare, for instance, Seven languages' ``optimized_queens.pl`` vs [Swi-Prolog API example on N-Queens](https://www.swi-prolog.org/pldoc/man?section=clpfd-sudoku)

**Seven Languages in Seven Weeks: Solve 8-Queens with Prolog**

```prolog
valid_queen((_, Col)) :- member(Col, [1,2,3,4,5,6,7,8]).
valid_board([]).
valid_board([Head|Tail]) :- valid_queen(Head), valid_board(Tail). 
    
cols([], []).
cols([(_, Col)|QueensTail], [Col|ColsTail]) :- 
  cols(QueensTail, ColsTail).
  
diags1([], []).
diags1([(Row, Col)|QueensTail], [Diagonal|DiagonalsTail]) :- 
  Diagonal is Col - Row, 
  diags1(QueensTail, DiagonalsTail).
  
  
diags2([], []).
diags2([(Row, Col)|QueensTail], [Diagonal|DiagonalsTail]) :- 
  Diagonal is Col + Row, 
  diags2(QueensTail, DiagonalsTail).

eight_queens(Board) :- 
  %(1,X) means there's a queen on row 1, column X
  %no horizontal attacks and there are 8 queens
  Board = [(1, _), (2, _), (3, _), (4, _), (5, _), (6, _), (7, _), (8, _)], 
  valid_board(Board), 

  cols(Board, Cols), 
  diags1(Board, Diags1), 
  diags2(Board, Diags2), 
  fd_all_different(Cols), %no vertical attacks

  % next two constraints say no diagonal attacks
  % we need two separate definitions to deal with upper and lower diagonal
  fd_all_different(Diags1),   
  fd_all_different(Diags2).
```


Now let's look at the **Swi-Prolog API Example.** It uses the ``clpfd`` (finte-domain constraint language programming) library, which for the purpose of this code is just syntatic sugar using ``ins/2`` instead of ``member/2``. 

```prolog
:- use_module(library(clpfd)) # 

n_queens(N, Qs) :-
        %Qs is an ordered list of queens, each queen is a single number that is its column
        length(Qs, N),
        Qs ins 1..N, %equivalent to valid_queen
        safe_queens(Qs).

safe_queens([]).
safe_queens([Q|Qs]) :- safe_queens(Qs, Q, 1), safe_queens(Qs).

%rule for placing a new queen Q0 to an existing board of queens [Q|Qs], extending the board by 1
safe_queens([], _, _).
safe_queens([Q|Qs], Q0, D0) :-
        Q0 #\= Q, %no shared columns, so no vertical attacks
        abs(Q0 - Q) #\= D0, %no diagonal attacks
        % increment diagonal distance for recursive call
        D1 #= D0 + 1,
        safe_queens(Qs, Q0, D1).
```

Clever recursion. Conveniently generalizable to $N$-queens for arbitrary $N$. Fewer codes, less debugging.. Closer to math. Very cool. 

### Lesson 3. ChatGPT has improved, but still I would not use it as my main programming language teacher. 

As of August 15th, I asked GPT4o (whatever that latest, best paid version is) to solve the classic Zebra puzzle. 

It returned two different answers. One did not finish. Closer code inspection reveals the same error I discovered in ``Zebra 2``, that ``clpfd`` domains must be integers and not string atoms, but the compiler seemingly could not check that, and the program probably got stuck in numerous $5!^5$ permutations that the code was doing. I didn't feel like shelling out eletricity bill to let it finish, nor trying to prompt it to fix the code. This is basically the level of code writing I saw 1 year ago, just a LOT more codes now, and thus even harder to debug.

The other version is a much cleaner version of ``Zebra 1``. It uses pure prolog, no external libraries. It's actually quite good, just a little verbose and doesn't generalize. 

```prolog

% Zebra Puzzle solution in SWI-Prolog

solve(ZebraOwner, WaterDrinker) :-
    Houses = [house(_,_,_,_,_), house(_,_,_,_,_), house(_,_,_,_,_), house(_,_,_,_,_), house(_,_,_,_,_)],
    member(house(red, englishman, _, _, _), Houses),
    member(house(_, spaniard, _, _, dog), Houses),
    member(house(green, _, coffee, _, _), Houses),
    member(house(_, ukrainian, tea, _, _), Houses),
    next_to(house(green, _, _, _, _), house(white, _, _, _, _), Houses),
    member(house(_, _, _, old_gold, snails), Houses),
    member(house(yellow, _, _, kools, _), Houses),
    Houses = [_, _, house(_, _, milk, _, _), _, _],
    Houses = [house(_, norwegian, _, _, _)|_],
    next_to(house(_, _, _, chesterfields, _), house(_, _, _, _, fox), Houses),
    next_to(house(_, _, _, kools, _), house(_, _, _, _, horse), Houses),
    member(house(_, _, orange_juice, lucky_strike, _), Houses),
    member(house(_, japanese, _, parliaments, _), Houses),
    next_to(house(_, norwegian, _, _, _), house(blue, _, _, _, _), Houses),
    member(house(_, ZebraOwner, _, _, zebra), Houses),
    member(house(_, WaterDrinker, water, _, _), Houses).

next_to(A, B, List) :- left_of(A, B, List).
next_to(A, B, List) :- left_of(B, A, List).

left_of(Left, Right, [Left, Right|_]).
left_of(Left, Right, [_|Rest]) :- left_of(Left, Right, Rest).

% Query example:
% ?- solve(ZebraOwner, WaterDrinker).
```

If I were to learn Prolog from the above code, I would have been greatly detered by the need to cut-paste-rename (though it's one of my top uses for GPT right now: cut-paste-rename, so it's less of an issue, but it's still a lot of codes to read). Then I would have wondered if there's another way to approach the problem, and probably GPT would probably have sent me to solution 1,the faulty ``cplfd`` that doesn't finish. Horror repeats. So in short, honest learning on my meat neural network still gives better satisfaction. 

# Summary

This is nothing new, just some banal advice when interacting with our AI overlords. 

1. Use GPT for cut-paste-rename. 
2. Learn from an actual book.
3. Think before you write. 

But since I'm programmed by evolution to be lazy, it's sometimes very tempting to throw it at GPT and ``see if it can``. 

On ``Prolog``, I enjoyed it. Definitely more than ``Ruby``, which I found to be too similar to ``Python``, and ``IO``, which doesn't have a blockbuster application. Though I do like ``IO`` simplicity, and yes I got more comfortable with ``Javascript`` after my skull was drilled on the message passing spiele by ``IO``. Most ``IO`` open source projects are 10+ years old and inactive, unfortunately. I'm still looking for a flagship weekend project idea that showcase ``IO`` strengths. 

Back to ``Prolog``. It's fun. But it has some issues. Unlike ``IO``'s [excellent API](https://iolanguage.org/), the SWI-Prolog documentation often left me confused. The Seven Languages book constantly touts the magic of constraint programming, conveyed as "specify the program, call solve and you are done". It is cool, but it's the default mode of operation for how I interact with convex solvers or training neural networks, for example, so I'm less blown away. Finally, debugging in Prolog is complicated for beginners. I have a good understanding of convex solvers, or at least, I have parameters like ``epsilon`` or ``step size`` that I can tweak. Prolog feels more like a blackbox (even though it's "just" exhaustive search), and I cannot kick the nagging feeling that this has to breakdown or big enough problems, or that I could have done this as an integer program. 

For all these reasons, I'll keep ``Prolog`` as a niche tool that's great for a programming weekend. GPT gave me [some ideas](https://chatgpt.com/share/1d3367f6-a901-4efe-b85b-ab6dc5e3ae18) that would last for a couple of weekends. 




