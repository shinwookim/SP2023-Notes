Regression deals with **continunous** responses (as opposed to discrete values). In the simpliest terms, we are dealing with decimal numbers (i.e, floating point numbers).

Formally, we are considering the infinite number of real values within an interval.

Recall that we said all relationships learned from data to be an approximate (i.e., $y \approx f(\textbf{x})$).

## Example
We begin with an example in which we know that **ground truth**. That is, we know the true relationship between a *noise-free* signal and a single input variable $x$.

Our True relationship will be assumed to be a quadratic relationship: $\text{TRUTH}=\beta_{*0}+\beta_{*1}x+\beta_{*2}x^2$, where $\beta_{*0}=0.33$, $\beta_{*1}=1.15$, and $\beta_{*2}=-2.25$.


However, in reality, we can never observe the TRUTH. When we collect data, we are always recording a *noisy* observation. That is, the true, or noise-free signal is hidden from us, and we see a corrupted signal.

For our example, we will collect $N=30$ observations and denote the observed inputs as$\textbf{x}=\{ x_1,x_2,...,x_n,...,x_N\}$; and observed responses as $\textbf{y}=\{ y_1,y_2,...,y_n,...,y_N\}$.

We now have $N=30$ input-output pairs: $\{x_n,y_n\}$



