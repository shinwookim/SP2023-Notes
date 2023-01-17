Regression deals with **continunous** responses (as opposed to discrete values). In the simpliest terms, we are dealing with decimal numbers (i.e, floating point numbers).

Formally, we are considering the infinite number of real values within an interval.

Recall that we said all relationships learned from data to be an approximate (i.e., $y \approx f(\textbf{x})$).

## Example
We begin with an example in which we know that **ground truth**. That is, we know the true relationship between a *noise-free* signal and a single input variable $x$.

Our True relationship will be assumed to be a quadratic relationship: $\text{TRUTH}=\beta_{*0}+\beta_{*1}x+\beta_{*2}x^2$, where $\beta_{*0}=0.33$, $\beta_{*1}=1.15$, and $\beta_{*2}=-2.25$.


However, in reality, we can never observe the TRUTH. When we collect data, we are always recording a *noisy* observation. That is, the true, or noise-free signal is hidden from us, and we see a corrupted signal.

For our example, we will collect $N=30$ observations and denote the observed inputs as$\textbf{x}=\{ x_1,x_2,...,x_n,...,x_N\}$; and observed responses as $\textbf{y}=\{ y_1,y_2,...,y_n,...,y_N\}$.

We now have $N=30$ input-output pairs: $\{x_n,y_n\}^{N=30}_{n=1}$ 

We are abstracting the details of how the data was *collected*. For now, simply presume that 30 input-output pairs have been collected.
Plotting the 30 pairs on a scatterplot (black = noisy observation; red = TRUTH)
![](Pasted%20image%2020230117113913.png)
But how would we know that the TRUTH is a parabola if the TRUTH trend line wasn't shown?


---

### Linear Fitting
Assuming that we do not know the TRUTH,  we begin by **training** or **fitting** a model between the response $y$ and the input $x$. We start with a simple linear relationship: $\beta_0+\beta_1x+\text{error}$. (Note that any learning can only produce apporximate relationships. Thus, there will always be an error).

The beta($\beta$)s are the coefficients or parametrs of the model. We learn or estimate their values from the data by minimizing the error.

We can use `R` to fit the model to our data. First, store the data in the object `my_train` which contains two columns (variables) `x` and `y`.

```R
# Fit a linear relationship using `lm()` function and the formula interface
mod1 <- lm(y~x, data = my_train)
## Formula interface (`y~x`) allows us to specify the response and inputs (predictors) to the model, <OUTPUT VAR NAMES> ~ <INPUT VAR NAME>
### Read as "the output, `y`, is a function of the input, `x`"
## We also assign `my_train` object to the `data` argument of `lm()` function

# We can summarize the results with the `summary()` function
summary(mod1)
# `(Intercept)` = β_0
# `x = β_1

# We can visualize the coefficient estimates and confidence intervals
coefplot::coefplot(mod1) +
	labs(title = "Coefficient plot for linear relationship model") +
	theme_bw() + 
	theme(axis.text.y = element_text(size = 14))
```
![](Pasted%20image%2020230117115015.png)
If Zero(0) is contained within the confidence interval of the slope ($\beta_1$), we cannot say the slope is definitely negative (even if the estimate is negative). Thus, we conclude that $x$ is not statistically significant.

Conversely, if zero(0) is not contained, we can say definitively that the slope is positive (or negative), thus we say that $x$ is statistically significant.