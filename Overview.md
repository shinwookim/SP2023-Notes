# Course Introduction & Goals
Machine learning is impacting *all* fields and *all* jobs one way or another.

A common misconception is that machine learning is simply programming (in a language like Python). However, the most important aspect of machine learning is NOT programming. The crucial part of machine learning is **understanding the assumptions** behind the methods.

Understanding the assumptions will allow us to understand what controls the model behavior. It helps us identify the strengths and weaknesses of the various methods. Most importantly, we can understand when a method is NOT   
appropriate. Thus, our focus will primarily be on mathematics, statistics, and  probability.

When we understand the statistics and linear algebra, we will gain the foundation to work in any programming language (after learning the syntax). We will use the **R** statistical programming language as we want to focus on the statistics. (R will also allow us to look at [**tidy data principles**](https://r4ds.had.co.nz/tidy-data.html) and **Grammar of Graphics**.)

---

# Focus: Supervised Learning
Our primary focus will be on predictive modeling, predictive analysis, and **supervised learning**. The goal of supervised learning is to learn the relationship (also called map) between the **input**s and **output** (**responses**, **target**, or **classes**).
- Inputs are typically denoted as $x$ (scalar), $\textbf{x}$ (vector), and $\textbf{X}$ (matrix).
- Responses are typically denoted as $y$($\textbf{y}$,$\textbf{Y}$), or $t$($\textbf{t}$,$\textbf{T}$).
- The relationship is typically denoted as $f(\textbf{x})$.
	- Note that we cannot say $y=f(\textbf{x})$ since the relationships learned from data is *always* an approximate. That is $y \approx f(\textbf{x})$. 
	- However, we can account for the error by writing $y=f(\textbf{x})+\text{error}$. The error is sometimes called the **loss function**
All models have **parameters** (or **coefficients**) we need to learn as part of the model **fitting**, **training**, or **building** process. We can figure out the parameters by **minimizing the model error**. Since the outputs guide/**supervise** t
