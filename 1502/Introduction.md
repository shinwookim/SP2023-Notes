# Unsolvable problems
In math and computing, there are some problems that are just unsolvable. For instance, consider the **integral root**:
> The numbers which satisfy the value of a polynomial are called its roots, and the roots which are integers that are not irrational or imaginary are called **integral roots.**

Given a polynomial with multiple variables ($6x^3yz^2+3xy^2-x^3-10$), can we determine if it has integral roots?

Given a set of dominoes ($\{[\frac{b}{ca}],[\frac{a}{ab}], [\frac{ca}{a}], [\frac{abc}{c}] \}$), is it possible to make a horizontal line of one or more dominoes, with duplicates allowed, so that the string obtained by reading across the top halves matches the one obtained by reading across the bottom? (**Post Correspondence Problem**)

There are infinite number of algorithms (some of which have not been discovered or invented yet). We will see how we can **prove** that none of these algorithms can be used to solve these problems?

However, we don't have a mathematical representation that can be used to express all  
algorithms. So, we use a **computational model** to represent a set of algorithms. We will first look at the **finite automata** (which captures a subset of algorithms), and move on to **turing machines** (which captures all algorithms). Then, we will study the limitations of computational models (and thus the limitation of algorithms), and how to formally verify these limitations.

# Unrealistically solvable problems
There are some problem that take too long to solve. They are solvable problems, but are also unrealistically solvable. For instance, consider the **Travel Salesman Problem**.

We can find a route to visit 1000 cities within a distance: $1000!\approx 4\times 10^{2567}$ possible routes. Even if we assume that we are visiting each path in 1 nano-second, it will take us approximately $4\times 10^{2558}$ seconds, or $4\times 10^{2549}$ years (longer than the age of the universe).

We will study 
Do we have a faster algorithm?
Maybe
Maybe not
Need to learn how to spot these kind of problems in advance