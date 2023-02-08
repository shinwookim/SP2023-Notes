# Intro to Regression

# Regression and Continuous Responses

Regression is a staple of classical statistical modeling, which deals with **continuous** responses (as opposed to discrete values). In simple terms, regression deals with decimal numbers (e.g., floating point numbers). Formally, working with continuous responses means considering the infinite number of real values within an interval.

## Simple Example
Previously, we said that all relationships *learned* from data are an approximate (i.e., $y \approx f(x)$). However, let us create an example in which we know the **ground truth**. That is, we know the *true* relationship between a ***noise-free*** signal and a single input variable, $x$.

For our example, we will set the true relationship to be quadratic: $\text{Truth}=\beta_{*0}+\beta_{*1}x+\beta_{*2}x^2$, where $\beta_{*0}=0.33$, $\beta_{*1}=1.15$, and $\beta_{*2}=-2.25$. Graphically, we can observe that the true quadratic trend is a parabola.

Note that in reality, we can never observe the exact truth. When we collect data, our recordings are always obfuscated by **noise**. That is, the true (or noise-free) signal is hidden from us, and we see a corrupted signal.

For this example, we will collect $N=30$ *noisy* observations. For simplicity, we can denote the observed inputs as $\textbf{x}=\{ x_1,x_2,...,x_n,...,x_N\}$; and observed responses as $\textbf{y}=\{ y_1,y_2,...,y_n,...,y_N\}$. Thus, we now have $N=30$ input-output pairs: $\{x_n,y_n\}^{N=30}_{n=1}$ 
If we visualize our observations on a scatter plot, we can see that our observation doesn't fit the truth (in red) perfectly. ![](Noisy%20Data.png) In fact, without the true trend displayed, we could not figure out that the real truth was a parabola.

Let us now assume that we do not know the truth (much like a real problem). We, now, want to **train** or **fit** a model between the response $y$ and the input $x$.

## Linear Fitting
We begin our training with a simple linear relationship: $$y=\beta_0+\beta_1x+\text{error}$$
Note that since learning from data can only approximate relationships, we always expect some error. Our goal will be to minimize this error by varying the parameters (coefficients) for our model: $\beta_0$ and $\beta_1$.

Later in the course, we will see just how exactly this learning process is implemented. For now, we will rely on **R** to conduct most of the work. To do so, we will first store the data in an object titled `my_train` so that it contains two columns named `x` and `y`; Next, we will use the `lm()` function and the formula interface to fit a linear relationship/.

Notice that we can print out a few rows of `my_train` using  `my_train %>% dplyr::select(x, y)`. This will let us see the variables we are working with. 

To fit the model, we will use the `lm()` function:
```R
### Fit a linear model using lm() and the formula interface
mod1 <- lm(y ~ x, data = my_train)

### The assignment operator, <-, assigns an object to a variable.

### The formula interface allows us to specify the responses and inputs (predictors) to the model.
# Basic Expression: <OUTPUT VARIABLE NAMES> ~ <INPUT VARIABLE NAMES>
# Read as: "The output, y, is a function of the input, x"

### We also assign the my_train object to the data argument
```

Once we have fit the model and assigned it, we can summarize the results with the `summary()`function:
![](lm%20Summary.png)
Here, the `(Intercept)` is our $\beta_0$ and `x` is our $\beta_1$. For now, we need not know what these all mean. Visually, we can represent our model (its coefficient estimates and confidence intervals) by using the `coefplot` package:
```R
coefplot::coefplot(mod1) +
	labs(title = "Coefficient plot for linear relationship model") +
	theme_bw() +
	theme(axis.test.y = element_text(size = 14))
```
![](Coefplot%20for%20LM.png)
The blue dot at the top represents our estimate for $\beta_1$ while the blue dot at the bottom represents our estimate for $\beta_0$. The horizontal lines (intervals) represent the **uncertainty** in the coefficient. Notice that the interval for our estimate for $\beta_1$ contains zero(0). Therefore, we cannot say for sure that the slope is definitely negative (even if the estimate is negative). Thus, we conclude that $x$ is *not* **statistically significant**. Conversely, if zero(0) were not contained in the interval, we could conclude that the slope was definitively negative, and thus $x$ is statistically significant.

## Quadratic Fitting
We've attempted to fit a single linear model to our data. Now, we will fit a quadratic relationship between the response and the input:$$y=\beta_0+\beta_1x+\beta_2x^2+\text{error}$$
To fit a quadratic model, we need to adjust the formula interface. However, everything else is the same as for fitting a linear model: `mod2 <- lm(y ~ x + I(x^2), data = my_train)`.

Note that we must place the $x^2$ predictor within the `I()`  function. We will learn later that `^2` is reserved for a shortcut. Thus, the `I()` function tells the formula function *as-is*.

Also, our formula interface is read the same as before, but now our model has more **predictors** (or **features**). That is, we still have 1 **input** ($x$), but 2 **predictors**($x$ and $x^2$).

In general, additive formula expressions with 2 predictors are expressed as `<OUTPUT> ~ <PREDICTOR 1> + <PREDICTOR 2>`.

Again, we can summarize using `summary()` and visualize using `coefplot`. The only change to our summary will be the addition of another row `I(x^2)` which displays our estimate for $\beta_2$.
![](coef%20lm%20quad.png)
Our visualization is the same as before, with estimates marked with dots, and confidence intervals marked with horizontal lines (Here, we've also illustrated the true coefficients  in red). We notice that not only are the estimates pretty close to the true values, but the true values are also contained within the confidence intervals. Furthermore, in comparison to the linear model, our quadratic model seems more accurate.

### Comparing models using performance metrics
Since we cannot compare the coefficients estimates to true values in a real problem, we need a way to assess which model is better.

We started our fitting process with the goal of minimizing error. So, we want some **performance metric** which can measure how much error is incurred from each model. Specifically, we will use the **sum of squared errors** between the model and the observations. (Note that the Least Squares is derived from the sum of squared errors).

A natural choice for a performance metric in a regression problem is the **Mean-Squared Error (MSE)**. The MSE is the mean or average squared error across all observations. However, the MSE is not in the same units as the response (it is squared). Thus, we must take the square root of the MSE to put the performance metric in the same units as the response. Thus, we often consider the **Root Mean Squared Error** (RMSE) as a performance metric.

Note that the RMSE  is just one choice for a performance metric. As an alternative, we could also consider the **Mean Absolute Error (MAE)**.

## Fitting higher order models
Now that we have a performance metric, let's model higher degree polynomials and compare. For this example, we will consider a total of 9 models (starting at 0th degree up to 8th degree polynomial).

This can be done in a similar way to modeling the linear or quadratic models using the formula interface. Once we have the models assigned, we can use the `rmse()` function (`modelr` package, part of `tidyverse`) to calculate the RMSE.
```R
### rmse(<Model Object>, <Data to calculate error relative to>)
modelr::rmse(mod1, my_train)
```
If we calculate the RMSE for all 9 models, and display them visually,![](Pasted%20image%2020230203010725.png), we notice that the 8th degree polynomial has the lowest RMSE.

But we know the true trend to be a parabola (since we designed this problem)! Let's look a but close at the model "fits" or predictions.![](Pasted%20image%2020230203010836.png) We can visually display a **facet** for each model. With training output displayed in red open square, and the model fits as black dots, we can visualize the response in respect to the input. 

Rather than visualizing the fits vs the inputs, we can also visualize the "fits" or model training set predictions with respect to the observed training output in a figure called **predicted vs observed figure**.![](Pasted%20image%2020230203011158.png) Note that the 45-degree line depicts a **perfect linear relationship** between the model prediction and the observation. That is, the closer the predictions fall along the 45-degree line, the greater the **correlation** is between the model and the observations. 

We can also look at the **R-squared**, the squared correlation coefficient between the model and the observation, by using the `rsquare()` function (using the same syntax as `rmse()`):
```R
rsquare(mod2, my_train)
```
![](Pasted%20image%2020230203011503.png)
Once again, the 8th degree is the best in terms of R-squared. We are seeing that the performance metrics improve as the polynomial degere increases. That is because our polynomial degree represents complexity (higher polynomial degree mode has more coefficients meaning greater complexity). Thus, the performace gets better as the models become more and more complex.

## Testing our model on a new training set
Until now, we've only assessed model performance with respect to the traing set. But how does our model perform to new data?

Visualizing model predictions is an important tool for interpreting and understanding model perofmrance. With single input problems, it is easy to create a test grid.
``` R
test_viz <- tibble::tibble(x=seq(-2.1,2.1), length.out -51)
```
- The `seq()` function creates a SEQUENCE of evenly spaced values from `<first argument>` to`<second argument>`.
	- The number of values is specified by the third argument. In this case, `length.out=` tells `seq()`to create 51 evenly spaced points. 
	- Other options exist for the third argument, including `by=` which specifies the interval size between points
	- Since `seq()` creates a vector, we need to assign it to the named variable `x` inside a tibble (modern `data.frame`). This way, the test grid has the same input name as the training set.

If we examine the `test_viz` object, we'll notice 51 evenly spaced points between -2.1 and 2.1. Note that the test grid does not include output values. This is fine because we do not need to know the responses to make predictions. However, we will not be able to compare model predictions to any observations (meaning we cannot calculate errors); but we will still be able to study trends.

## Making predictions
R provides a `predict()` function which makes creating predictions easy:
```R
<result> <- predict(<model object>, <test est>)
```
Note that the `<test set>` could be the training set, if we wanted to remake predictions on the training set later on. The `<result>` of this function will be a numeric vector.

Once we have made the predictions, we can easily visualize them with the test set input value `x` in the x-axis, and the model predicted response, fit, on the y-axis. Note that to clearly visualize the trends, it is useful to vary the y-axis scale among various models (We can do so by using `scales="free_y"` inside the `facet_wrap()` function):
![](Pasted%20image%2020230203102828.png)

This shows us the trends, but what if we want to see the prediction uncertainty? We can tell the `predict()` function to return *two* types of prediction uncertainty:
1. **Confidence Interval**
```R
test_confint_2 <- predict(mod2, test_viz, interval = "confidence")
```
2. **Prediction Interval**
```R
test_predint_2 <- predict(mod2, test_viz, interval = "prediction")
```
Once again, we can visualize these outputs:
![](Pasted%20image%2020230203103044.png)
Note that the y-axis varies among various facets. The mean trends are shown by black curves; the gray ribbons are the confidence intervals; and the orange ribbons are the prediction intervals.

We notice that the y-axis on the higher degree polynomials are huge! Furthermore,   the confidence intervals show that the mean trend of the higher degree polynomials are highly uncertain. That is, the higher degree models are highly variable.

In fact, calculating the coefficients and their confidence interval, we notice that the 8th degree polynomial has coefficients near -20 and 20; and the estimates are very uncertain! This is the **Bias-Variance** trade-off.

## Data splitting

In this example, we knew the truth and thus knew that the higher models were wrong. In a real problem, however, how can we assess our model's behavior if the metrics on the training set tell us the wrong answer?

If we had a new data set, it might help represent the truth...or help us understand how our model **generalizes**. But what if we don't have another data set?

We can break up, or **split,** the data set. That is, we partition the complete data set into a dedicated **training set**, and a **holdout set** (which we can use to assess the model performance).

With our example, let us split our data ($N=30$) into 24 training points, and 6 hold out points. This scheme is known as the **80/20 training/test split**. In R, we accomplish this using the `sample()` function:
```R
set.seed(5501) # Setting the seed allows for reproducibility
id_train <- sample(1:nrow(my_train), 24)
train_split <- my_train %>% slide(id_train)
holdout_split <- my_train %>% slide(-id_train)
```
![](Pasted%20image%2020230203103944.png)
(See the HOML book for different ways to create data splits)

Once we have split our data set, we can use the training set to train each model (We simply specify `data=train_split` in the `lm()` calls). Then, we can use the holdout set to calculate the performance: 
``` R
mod3_split <- lm(y ~ x + I(x^2) + I(x^3), data = train_split)
modelr::rmse(mod3_split, holdout_split)
```

If we repeat these steps for every model (train 9 models on the same training split, and calculate the performance metric on the same holdout test split), we'll notice that the 7th degree polynomial will perform the best!![](Pasted%20image%2020230203170416.png)
But we said the truth was a quadratic...why doesn't the train/test split find the right answer?

Let's try again, but this time rather than splitting the data just once, we split it multiple times. This is called **resampling** (repeat sampling). Using resampling, we can create multiple training and test splits. Thus, we will be able to train each model multiple times and assess each model's performance multiple times.

There are two common approaches to resampling:
1. **Bootstrap**: Resampling *with replacement*
2. **K-fold cross-validation**: ensures each observation is used as a test point once.

### K-fold cross validation
In this scheme, we randomly partition the data into k-**folds** such that each observation is in a test set once and *only* once. The most popular choices for $k$ is 5-fold, or 10-fold.

Although there are various libraries and functions we could use in R, one approach is to use `modelr` (see HOML for other approaches).
```R
set.seed(23413)
cv_info_k05 <- modelr::crossv_kfold(my_train, k=5)
```