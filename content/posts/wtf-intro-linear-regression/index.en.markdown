---
title: What they forgot to teach you in your introduction to linear regression with R
author: Adam Shen
date: '2025-06-29'
slug: wtf-intro-linear-regression
categories:
  - r
  - education
tags:
  - r
  - linear-regression
subtitle: ''
summary: "R functions that would have enhanced your coding experience when modelling"
featured: no
reading_time: false
---



There exists a handful of functions in base-R that are closely related to the typical introductory
linear regression curriculum. More often than not, I feel that these functions are not exposed to
students in any introductory linear regression course where R is used for the computational
component. For students who are taking or have recently finished an introductory linear regression
course, if there were any instances where you came across some code that made you think, "there must
be an easier, cleaner way to do this", you were probably right!

There are a number of reasons why I have wanted to make this post for a while:

- The functions that I will discuss in this post can be extended to other contexts, such as GLMs,
with one particular function being applicable to more general contexts.

- It encourages you early on to write clean code and manage your workspace effectively. In cases
where you come across code that feels busy, inefficient, or clutters up your workspace with
unnecessary transitory variables, take the initiative to seek out cleaner alternatives.

- Follow the philosophy of "work smarter, not harder".

- I have met **a lot** of students who are just starting to use R and already dread using/learning
it. What they don't know is that this is often a result of the inefficient misuse by instructors who
barely know the language themselves (it happened more often than I would have hoped). Therefore, I
hope this post gives you an ounce of hope that R is not as messy/ugly as you might have been shown,
and that you do not become a hateR.

## Setup

We will be using the [`{dplyr}`](https://dplyr.tidyverse.org/) package for a tiny amount of data
wrangling. It can be done in base-R, if that is your preference.


``` r
library(dplyr)
```

If you do not have `{dplyr}` installed, you can install it by calling `install.packages("dplyr")`
in the console.

For the following demonstrations, we will be using the `mtcars` data set. This data set is included
in base-R.


``` r
data(mtcars)

head(mtcars)
```

```
##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1
```

For our base linear model, we will use `mpg` as the response and `disp` as a predictor.


``` r
base_model <- lm(mpg ~ disp, data=mtcars)

summary(base_model)
```

```
## 
## Call:
## lm(formula = mpg ~ disp, data = mtcars)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -4.8922 -2.2022 -0.9631  1.6272  7.2305 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 29.599855   1.229720  24.070  < 2e-16 ***
## disp        -0.041215   0.004712  -8.747 9.38e-10 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 3.251 on 30 degrees of freedom
## Multiple R-squared:  0.7183,	Adjusted R-squared:  0.709 
## F-statistic: 76.51 on 1 and 30 DF,  p-value: 9.38e-10
```

## `update()`

The `update()` function allows you to add or modify arguments used to construct a linear model and
subsequently re-fit the model using these new values. This can include modifying the response
variable, the covariates, the data set used, or even other options used to fit the model. There are
two benefits to using `update()` rather than building a new model from scratch.

1. You specify only the arguments whose values should change. As such, all other aspects of the
existing model that should remain unchanged do not need to be re-typed.

2. Since the `update()` function requires the user to specify an object to update, from a workflow
perspective, the code reveals the evolutionary relationship between models in your workspace by
highlighting what is changing between model builds, and provides a "story" for the model building
process.

Suppose we wanted to rebuild my `base_model` using a subset of the `mtcars` data set that includes
only four and eight cylinder cars.


``` r
mtcars_4or8cyl <- mtcars %>%
  filter(cyl %in% c(4, 8))

# Or in base-R
# mtcars_4or8cyl <- mtcars[mtcars$cyl %in% c(4, 8),]
```

Instead of building a linear model from scratch again by calling:


``` r
second_model <- lm(mpg ~ disp, data=mtcars_4or8cyl)
```

We can call:


``` r
second_model <- update(base_model, data=mtcars_4or8cyl)
```

If we wanted to update the formula instead of the data set, we can take advantage of the
`.`-shorthand, which means to keep everything from the respective side of the formula.


``` r
third_model <- update(base_model, . ~ . + hp)
```

Since the formula supplied to `base_model` was `mpg ~ disp`, `. ~ . + hp` will translate to
`mpg ~ disp + hp`.

## `add1()` / `drop1()`

The `add1()` and `drop1()` functions are useful functions for performing the required hypothesis
tests to add or drop a covariate from a set of candidate covariates. Too often have I seen the
following workflow being recommended to students to perform a single variable addition: 


``` r
model4a <- lm(mpg ~ disp + hp, data=mtcars)
model4b <- lm(mpg ~ disp + drat, data=mtcars)
model4c <- lm(mpg ~ disp + wt, data=mtcars)
model4d <- lm(mpg ~ disp + qsec, data=mtcars)

anova(model4a, base_model)
anova(model4b, base_model)
anova(model4c, base_model) # Has the smallest p-value
anova(model4d, base_model)

model4 <- model4c
```

You **do not** need to do all this work! In addition to being a lot of typing (or copying, pasting,
and replacing, which is error-prone), you also end up cluttering your workspace with four additional
models, three of which you will never use again. After deciding on a fourth model, if you had
decided you wanted to proceed with adding a third covariate, you'd continue cluttering up your
workspace with even more models!

The following is all you need:


``` r
add1(base_model, ~ . + hp + drat + wt + qsec, test="F")
```

```
## Single term additions
## 
## Model:
## mpg ~ disp
##        Df Sum of Sq    RSS    AIC F value   Pr(>F)   
## <none>              317.16 77.397                    
## hp      1    33.665 283.49 75.806  3.4438 0.073679 . 
## drat    1    14.263 302.90 77.925  1.3655 0.252097   
## wt      1    70.476 246.68 71.356  8.2852 0.007431 **
## qsec    1     3.622 313.54 79.030  0.3350 0.567196   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

This single line of code builds the required models and returns the p-value of the corresponding
F-test. From the output, we can see that `wt` has the smallest p-value and should be added to the
model. We can then use the `update()` function to create our second model.


``` r
fourth_model <- update(base_model, . ~ . + wt)
```

For single variable deletions, a similar procedure can be followed using `drop1()`. For this, I
build what I consider to be a "full model", which has all the covariates of interest: `disp`, `hp`,
`drat`, `wt`, and `qsec`. 


``` r
full_model <- lm(mpg ~ disp + hp + drat + wt + qsec, data=mtcars)
```

If all covariates are eligible to be dropped, we can call:


``` r
drop1(full_model, ~ ., test="F")
```

```
## Single term deletions
## 
## Model:
## mpg ~ disp + hp + drat + wt + qsec
##        Df Sum of Sq    RSS    AIC F value   Pr(>F)   
## <none>              170.13 65.466                    
## disp    1     3.974 174.10 64.205  0.6074 0.442812   
## hp      1    11.886 182.01 65.627  1.8165 0.189360   
## drat    1    15.506 185.63 66.258  2.3697 0.135791   
## wt      1    81.394 251.52 75.978 12.4390 0.001584 **
## qsec    1    12.708 182.84 65.772  1.9422 0.175233   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

If we only want to consider dropping one of either, for example, `wt` or `qsec`, we can call:


``` r
drop1(full_model, ~ wt + qsec, test="F")
```

```
## Single term deletions
## 
## Model:
## mpg ~ disp + hp + drat + wt + qsec
##        Df Sum of Sq    RSS    AIC F value   Pr(>F)   
## <none>              170.13 65.466                    
## wt      1    81.394 251.52 75.978 12.4390 0.001584 **
## qsec    1    12.708 182.84 65.772  1.9422 0.175233   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

As before, we can finalise the model by making a call to `update()`:


``` r
fifth_model <- update(full_model, . ~ . - qsec)
```

## Optional tidy extensions

The [`{broom}`](https://broom.tidymodels.org/) package from
[`{tidymodels}`](https://www.tidymodels.org/packages/) contains tools for tidying model output into
tibbles (enhanced data frames) for when you require further (possibly programmatic) wrangling of
model output.


``` r
library(broom)
```

When calling `summary()` on a linear model, a lot of information is returned:


``` r
summary(full_model)
```

```
## 
## Call:
## lm(formula = mpg ~ disp + hp + drat + wt + qsec, data = mtcars)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.5404 -1.6701 -0.4264  1.1320  5.4996 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)   
## (Intercept) 16.53357   10.96423   1.508  0.14362   
## disp         0.00872    0.01119   0.779  0.44281   
## hp          -0.02060    0.01528  -1.348  0.18936   
## drat         2.01578    1.30946   1.539  0.13579   
## wt          -4.38546    1.24343  -3.527  0.00158 **
## qsec         0.64015    0.45934   1.394  0.17523   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.558 on 26 degrees of freedom
## Multiple R-squared:  0.8489,	Adjusted R-squared:  0.8199 
## F-statistic: 29.22 on 5 and 26 DF,  p-value: 6.892e-10
```

This information can be returned as two separate tibbles via `tidy()` and `glance()`. 

## `tidy()`

We can call `tidy()` to obtain the coefficient table, standard errors, values of the test
statistics, and p-values.


``` r
tidy(full_model)
```

<PRE class="fansi fansi-output"><CODE>## <span style='color: #555555;'># A tibble: 6 × 5</span>
##   term        estimate std.error statistic p.value
##   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>          <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>
## <span style='color: #555555;'>1</span> (Intercept) 16.5       11.0        1.51  0.144  
## <span style='color: #555555;'>2</span> disp         0.008<span style='text-decoration: underline;'>72</span>    0.011<span style='text-decoration: underline;'>2</span>     0.779 0.443  
## <span style='color: #555555;'>3</span> hp          -<span style='color: #BB0000;'>0.020</span><span style='color: #BB0000; text-decoration: underline;'>6</span>     0.015<span style='text-decoration: underline;'>3</span>    -<span style='color: #BB0000;'>1.35</span>  0.189  
## <span style='color: #555555;'>4</span> drat         2.02       1.31       1.54  0.136  
## <span style='color: #555555;'>5</span> wt          -<span style='color: #BB0000;'>4.39</span>       1.24      -<span style='color: #BB0000;'>3.53</span>  0.001<span style='text-decoration: underline;'>58</span>
## <span style='color: #555555;'>6</span> qsec         0.640      0.459      1.39  0.175
</CODE></PRE>

Optionally, with `tidy()`, we can also obtain the confidence intervals:


``` r
tidy(full_model, conf.int=TRUE)
```

<PRE class="fansi fansi-output"><CODE>## <span style='color: #555555;'># A tibble: 6 × 7</span>
##   term        estimate std.error statistic p.value conf.low conf.high
##   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>          <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>
## <span style='color: #555555;'>1</span> (Intercept) 16.5       11.0        1.51  0.144    -<span style='color: #BB0000;'>6.00</span>     39.1   
## <span style='color: #555555;'>2</span> disp         0.008<span style='text-decoration: underline;'>72</span>    0.011<span style='text-decoration: underline;'>2</span>     0.779 0.443    -<span style='color: #BB0000;'>0.014</span><span style='color: #BB0000; text-decoration: underline;'>3</span>    0.031<span style='text-decoration: underline;'>7</span>
## <span style='color: #555555;'>3</span> hp          -<span style='color: #BB0000;'>0.020</span><span style='color: #BB0000; text-decoration: underline;'>6</span>     0.015<span style='text-decoration: underline;'>3</span>    -<span style='color: #BB0000;'>1.35</span>  0.189    -<span style='color: #BB0000;'>0.052</span><span style='color: #BB0000; text-decoration: underline;'>0</span>    0.010<span style='text-decoration: underline;'>8</span>
## <span style='color: #555555;'>4</span> drat         2.02       1.31       1.54  0.136    -<span style='color: #BB0000;'>0.676</span>     4.71  
## <span style='color: #555555;'>5</span> wt          -<span style='color: #BB0000;'>4.39</span>       1.24      -<span style='color: #BB0000;'>3.53</span>  0.001<span style='text-decoration: underline;'>58</span>  -<span style='color: #BB0000;'>6.94</span>     -<span style='color: #BB0000;'>1.83</span>  
## <span style='color: #555555;'>6</span> qsec         0.640      0.459      1.39  0.175    -<span style='color: #BB0000;'>0.304</span>     1.58
</CODE></PRE>

This is useful since base-R's `confint()` function actually returns a matrix:


``` r
confint(full_model)
```

```
##                   2.5 %      97.5 %
## (Intercept) -6.00372864 39.07086783
## disp        -0.01427933  0.03171968
## hp          -0.05201291  0.01081675
## drat        -0.67585793  4.70740704
## wt          -6.94137515 -1.82955263
## qsec        -0.30404593  1.58434573
```

``` r
class(confint(full_model))
```

```
## [1] "matrix" "array"
```

Compared to base-R's `confint(x)`, `tidy(x, conf.int = TRUE)` saves us a few steps of needing to
convert the matrix of confidence intervals to data frames (or tibbles) and then binding the results
back to the coefficient table so that they can be viewed together.

## `glance()`

To obtain other miscellaneous attributes of the fitted linear model that we usually see in the
output of `summary()`, we can pass it to `glance()`:


``` r
glance(full_model)
```

<PRE class="fansi fansi-output"><CODE>## <span style='color: #555555;'># A tibble: 1 × 12</span>
##   r.squared adj.r.squared sigma statistic  p.value    df logLik   AIC   BIC
##       <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>         <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>
## <span style='color: #555555;'>1</span>     0.849         0.820  2.56      29.2 6.89<span style='color: #555555;'>e</span><span style='color: #BB0000;'>-10</span>     5  -<span style='color: #BB0000;'>72.1</span>  158.  169.
## <span style='color: #555555;'># ℹ 3 more variables: deviance &lt;dbl&gt;, df.residual &lt;int&gt;, nobs &lt;int&gt;</span>
</CODE></PRE>

## `augment()`

The `augment()` function is a quick way to get the fitted values column-binded to the data frame
used to fit the model, i.e. you get all covariate values plus the fitted value, plus additional
diagnostic values.


``` r
augment(full_model)
```

<PRE class="fansi fansi-output"><CODE>## <span style='color: #555555;'># A tibble: 32 × 13</span>
##    .rownames      mpg  disp    hp  drat    wt  qsec .fitted .resid   .hat .sigma
##    <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>        <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>
## <span style='color: #555555;'> 1</span> Mazda RX4     21    160    110  3.9   2.62  16.5    22.6 -<span style='color: #BB0000;'>1.57</span>  0.148    2.59
## <span style='color: #555555;'> 2</span> Mazda RX4 W…  21    160    110  3.9   2.88  17.0    21.8 -<span style='color: #BB0000;'>0.812</span> 0.131    2.60
## <span style='color: #555555;'> 3</span> Datsun 710    22.8  108     93  3.85  2.32  18.6    25.1 -<span style='color: #BB0000;'>2.26</span>  0.067<span style='text-decoration: underline;'>4</span>   2.57
## <span style='color: #555555;'> 4</span> Hornet 4 Dr…  21.4  258    110  3.08  3.22  19.4    21.1  0.329 0.143    2.61
## <span style='color: #555555;'> 5</span> Hornet Spor…  18.7  360    175  3.15  3.44  17.0    18.2  0.473 0.175    2.61
## <span style='color: #555555;'> 6</span> Valiant       18.1  225    105  2.76  3.46  20.2    19.7 -<span style='color: #BB0000;'>1.57</span>  0.219    2.58
## <span style='color: #555555;'> 7</span> Duster 360    14.3  360    245  3.21  3.57  15.8    15.6 -<span style='color: #BB0000;'>1.28</span>  0.153    2.59
## <span style='color: #555555;'> 8</span> Merc 240D     24.4  147.    62  3.69  3.19  20      22.8  1.61  0.129    2.59
## <span style='color: #555555;'> 9</span> Merc 230      22.8  141.    95  3.92  3.15  22.9    24.6 -<span style='color: #BB0000;'>1.75</span>  0.488    2.56
## <span style='color: #555555;'>10</span> Merc 280      19.2  168.   123  3.92  3.44  18.3    20.0 -<span style='color: #BB0000;'>0.792</span> 0.147    2.60
## <span style='color: #555555;'># ℹ 22 more rows</span>
## <span style='color: #555555;'># ℹ 2 more variables: .cooksd &lt;dbl&gt;, .std.resid &lt;dbl&gt;</span>
</CODE></PRE>

You can also obtain confidence or prediction intervals for each observation:


``` r
augment(full_model, interval="confidence")
```

<PRE class="fansi fansi-output"><CODE>## <span style='color: #555555;'># A tibble: 32 × 15</span>
##    .rownames      mpg  disp    hp  drat    wt  qsec .fitted .lower .upper .resid
##    <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>        <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>
## <span style='color: #555555;'> 1</span> Mazda RX4     21    160    110  3.9   2.62  16.5    22.6   20.5   24.6 -<span style='color: #BB0000;'>1.57</span> 
## <span style='color: #555555;'> 2</span> Mazda RX4 W…  21    160    110  3.9   2.88  17.0    21.8   19.9   23.7 -<span style='color: #BB0000;'>0.812</span>
## <span style='color: #555555;'> 3</span> Datsun 710    22.8  108     93  3.85  2.32  18.6    25.1   23.7   26.4 -<span style='color: #BB0000;'>2.26</span> 
## <span style='color: #555555;'> 4</span> Hornet 4 Dr…  21.4  258    110  3.08  3.22  19.4    21.1   19.1   23.1  0.329
## <span style='color: #555555;'> 5</span> Hornet Spor…  18.7  360    175  3.15  3.44  17.0    18.2   16.0   20.4  0.473
## <span style='color: #555555;'> 6</span> Valiant       18.1  225    105  2.76  3.46  20.2    19.7   17.2   22.1 -<span style='color: #BB0000;'>1.57</span> 
## <span style='color: #555555;'> 7</span> Duster 360    14.3  360    245  3.21  3.57  15.8    15.6   13.5   17.6 -<span style='color: #BB0000;'>1.28</span> 
## <span style='color: #555555;'> 8</span> Merc 240D     24.4  147.    62  3.69  3.19  20      22.8   20.9   24.7  1.61 
## <span style='color: #555555;'> 9</span> Merc 230      22.8  141.    95  3.92  3.15  22.9    24.6   20.9   28.2 -<span style='color: #BB0000;'>1.75</span> 
## <span style='color: #555555;'>10</span> Merc 280      19.2  168.   123  3.92  3.44  18.3    20.0   18.0   22.0 -<span style='color: #BB0000;'>0.792</span>
## <span style='color: #555555;'># ℹ 22 more rows</span>
## <span style='color: #555555;'># ℹ 4 more variables: .hat &lt;dbl&gt;, .sigma &lt;dbl&gt;, .cooksd &lt;dbl&gt;, .std.resid &lt;dbl&gt;</span>
</CODE></PRE>

Finally, you can also use augment to obtain the column-binded fitted values of unseen data by
passing it to `newdata`:


``` r
fake_cars <- tibble(
  mpg = c(26, 25),
  disp = c(170, 265),
  hp = c(100, 105),
  drat = c(2.7, 3.11),
  wt = c(3.10, 2.98),
  qsec = c(19.1, 20.3)
)

augment(full_model, newdata=fake_cars)
```

<PRE class="fansi fansi-output"><CODE>## <span style='color: #555555;'># A tibble: 2 × 8</span>
##     mpg  disp    hp  drat    wt  qsec .fitted .resid
##   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>
## <span style='color: #555555;'>1</span>    26   170   100  2.7   3.1   19.1    20.0   5.97
## <span style='color: #555555;'>2</span>    25   265   105  3.11  2.98  20.3    22.9   2.12
</CODE></PRE>

If we know the true values of the response variable (in this case, `mpg`), we can subsequently
compare the `mpg` values with the `.fitted` values to evaluate predictive performance via the
[`yardstick`](https://yardstick.tidymodels.org/) package. For example,
we might do something like:


``` r
more_unseen_data <- tibble(
  . . .
)

augment(full_model, newdata=more_unseen_data) %>%
  yardstick::rmse(truth=mpg, estimate=.fitted)
```

## Supplementary notes

- `update()`, `add1()`, and `drop1()` also work with glms!

- `update()` is a generic method. It works with lms, glms, and any other objects that have a `call`
element. The `update()` method can often be found in other modelling packages for use with those
package-specific models to perform updates in a similar fashion.
