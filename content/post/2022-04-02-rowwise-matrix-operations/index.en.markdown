---
title: Rowwise matrix operations
author: Adam Shen
date: '2022-04-02'
slug: 2022-04-02-rowwise-matrix-operations
categories:
  - r
  - data-wrangling
tags:
  - r
subtitle: ''
summary: 'Applying matrix operations to data rowwise'
featured: no
---



# Packages and setup


```r
library(tibble)
library(dplyr)

set.seed(20)
```

# The `rowwise` function

The [`rowwise`](https://dplyr.tidyverse.org/reference/rowwise.html) function allows for the grouping
of data by rows in order to perform some operation across the values found in its columns.

Some basic data:


```r
nums <- tibble(
  x1 = sample(1:5, size=6, replace=TRUE),
  x2 = sample(1:5, size=6, replace=TRUE),
  x3 = sample(1:5, size=6, replace=TRUE),
  x4 = sample(1:5, size=6, replace=TRUE)
)

nums
```

```
## # A tibble: 6 x 4
##      x1    x2    x3    x4
##   <int> <int> <int> <int>
## 1     3     1     4     5
## 2     2     3     1     1
## 3     1     5     5     5
## 4     2     1     1     2
## 5     5     5     1     1
## 6     5     2     4     5
```

I won't actually be using `x4` in the demo below, but I'm including it to make the data a bit more
realistic. Oftentimes, you will have more variables in your data set than the ones you are trying
to operate on, i.e. you can't always just `tidyselect::everything()`!

## Using `c_across` with `rowwise`

The [`c_across`](https://dplyr.tidyverse.org/reference/c_across.html) function is often used with
`rowwise` in order to **combine values across columns**. What happens if we use `c_across` without
`rowwise`?


```r
nums %>%
  mutate(test = mean(c_across(x1:x3)))
```

```
## # A tibble: 6 x 5
##      x1    x2    x3    x4  test
##   <int> <int> <int> <int> <dbl>
## 1     3     1     4     5  2.83
## 2     2     3     1     1  2.83
## 3     1     5     5     5  2.83
## 4     2     1     1     2  2.83
## 5     5     5     1     1  2.83
## 6     5     2     4     5  2.83
```

It can be seen that the `test` column has a single value that is repeated throughout. Without
grouping the data rowwise, the mean is computed using **all** the values across `x1`, `x2`, and
`x3`:


```r
mean(c(nums$x1, nums$x2, nums$x3))
```

```
## [1] 2.833333
```

Now, if we group the data rowwise before computing the mean across `x1`, `x2`, and `x3`:


```r
nums %>%
  rowwise() %>%
  mutate(test = mean(c_across(x1:x3)))
```

```
## # A tibble: 6 x 5
## # Rowwise: 
##      x1    x2    x3    x4  test
##   <int> <int> <int> <int> <dbl>
## 1     3     1     4     5  2.67
## 2     2     3     1     1  2   
## 3     1     5     5     5  3.67
## 4     2     1     1     2  1.33
## 5     5     5     1     1  3.67
## 6     5     2     4     5  3.67
```

We can check that the results are what we wanted:


```r
mean(c(3, 1, 4))
```

```
## [1] 2.666667
```

```r
mean(c(5, 2, 4))
```

```
## [1] 3.666667
```

They match &#8212; great!

**Note:** `rowwise` is a type of grouping. From the first two lines of the tibble output above, we
can see that after creating the `test` variable, the data is still grouped (rowwise)! Don't forget
to `ungroup` when you're done mutating.

## Rowwise matrix operations

Suppose you wanted to perform the following computation using the values found in each row:

$$ \mathbf{x}'\mathbf{A}\mathbf{x} $$

We will first create a function that:

1. Takes the combined values over a selection of columns (i.e. the result of `c_across`)
2. Converts the values to a matrix (a $ n \times 1 $ column vector)
3. Performs the matrix operation and reduces the resulting $ 1 \times 1 $ matrix to a vector of
length 1

\


```r
quadratic <- function(x, A) {
  x <- as.matrix(x)
  
  drop(t(x) %*% A %*% x)
}
```

For the $ A $ matrix, let's use:


```r
A <- matrix(
  sample(0:5, size=9, replace=TRUE),
  nrow=3, ncol=3
)

A
```

```
##      [,1] [,2] [,3]
## [1,]    1    2    4
## [2,]    2    0    5
## [3,]    4    3    4
```

Putting it all together:


```r
nums %>%
  rowwise() %>%
  mutate(test = quadratic(c_across(x1:x3), A)) %>%
  ungroup()
```

```
## # A tibble: 6 x 5
##      x1    x2    x3    x4  test
##   <int> <int> <int> <int> <dbl>
## 1     3     1     4     5   213
## 2     2     3     1     1    72
## 3     1     5     5     5   361
## 4     2     1     1     2    40
## 5     5     5     1     1   209
## 6     5     2     4     5   353
```

Checking our work:


```r
quadratic(c(3, 1, 4), A)
```

```
## [1] 213
```

```r
quadratic(c(5, 2, 4), A)
```

```
## [1] 353
```
