---
title: Row-binding lists of stuff
author: Adam Shen
date: '2022-03-09'
slug: 2022-03-09-row-binding-lists-of-stuff
categories:
  - r
  - data-wrangling
tags:
  - r
subtitle: ''
summary: 'Converting lists of vectors and matrices into a single data frame'
featured: no
---



The problem of needing to convert a single-level list of data into a single tibble comes up once
in while and I can never remember how I did it the last time I did it. So here we are! I would also
like to stay in the tidyverse where possible in order to keep the flow of my pipelines. I'd say the
only exception is the usage of `paste0` over `str_c`/`glue`.

# Packages and setup


```r
library(tibble)
library(dplyr)
library(purrr)
library(ggplot2) # optional
library(tidytext) # optional

theme_set(theme_bw()) # optional

set.seed(20)
```

# Lists of vectors

Recently, I needed to find the roots of arbitrary quadratic equations using the
[`polyroot`](https://stat.ethz.ch/R-manual/R-devel/library/base/html/polyroot.html) function. The
equation was of the form:

$$ a\_{22}x\_{2}^2 \,+\, (2a\_{12}x\_{1} + b\_{2})x\_{2} \,+\, (c + b\_{1}x\_{1} + a\_{11}x\_{1}^2) \,=\, 0. $$

If the equation had two distinct roots, a vector of length two was returned, e.g.:


```r
polyroot(c(6, 5, 1))
```

```
## [1] -2+0i -3-0i
```

If the equation had one distinct root or was not actually quadratic ($a = 0$), a vector of length
one was returned, e.g.:


```r
polyroot(c(1, 2))
```

```
## [1] -0.5+0i
```

If the equation was inconsistent ($a,\, b = 0,\, c \neq 0$), a vector of length zero was returned,
e.g.:


```r
polyroot(3)
```

```
## complex(0)
```

In addition to finding the root, I also wanted to know the value of $ x\_1 $ where this root
occurred.

Let's first create a function that will calculate the required quadratic coefficients, given the
input value `x1`.


```r
get_coefs <- function(x1) {
  coef_a <- 0.002
  coef_b <- 2*(-0.013)*x1 - 0.23
  coef_c <- 1.08 + 1.23*x1 + 0.098*x1^2
  
  c(coef_c, coef_b, coef_a)
}
```

Now, let's find some roots for some basic values of `x1`.


```r
some_roots <- seq(1, 1.60, length.out=7) %>%
  map(get_coefs) %>%
  map(polyroot)

head(some_roots)
```

```
## [[1]]
## [1]  10.22268-0i 117.77732+0i
## 
## [[2]]
## [1]  10.76278-0i 118.53722+0i
## 
## [[3]]
## [1]  11.30435-0i 119.29565+0i
## 
## [[4]]
## [1]  11.84739-0i 120.05261+0i
## 
## [[5]]
## [1]  12.39188-0i 120.80812+0i
## 
## [[6]]
## [1]  12.93782-0i 121.56218+0i
```

To convert this data into a tibble with all the roots in one column, we should map `as_tibble_col`
over all list elements.


```r
some_roots %>%
  head() %>%
  map_dfr(as_tibble_col, column_name="root")
```

```
## # A tibble: 12 x 1
##    root        
##    <cpl>       
##  1  10.22268-0i
##  2 117.77732+0i
##  3  10.76278-0i
##  4 118.53722+0i
##  5  11.30435-0i
##  6 119.29565+0i
##  7  11.84739-0i
##  8 120.05261+0i
##  9  12.39188-0i
## 10 120.80812+0i
## 11  12.93782-0i
## 12 121.56218+0i
```

While this works well in row-binding the roots into a single column tibble, it's unclear which roots
belong to which values of `x1`. In addition, it's not a simple as just column-binding back the
initial values of `x1` that we passed in since some roots are of length one and some are of length
two.

The solution is to give the intial sequence of `x1` values names that match its values and make use
of `map_dfr`'s `.id` argument, e.g.


```r
seq(-13, 33, by=0.10) %>%
  set_names(., .) %>%
  head()
```

```
##   -13 -12.9 -12.8 -12.7 -12.6 -12.5 
## -13.0 -12.9 -12.8 -12.7 -12.6 -12.5
```

Putting it all together:


```r
some_roots <- seq(-13, 33, by=0.10) %>%
  set_names(., .) %>%
  map(get_coefs) %>%
  map(polyroot) %>%
  map_dfr(as_tibble_col, column_name="root", .id="x1")

some_roots
```

```
## # A tibble: 922 x 2
##    x1    root              
##    <chr> <cpl>             
##  1 -13   -27.00000+9.84886i
##  2 -13   -27.00000-9.84886i
##  3 -12.9 -26.35000+8.14049i
##  4 -12.9 -26.35000-8.14049i
##  5 -12.8 -25.70000+5.97244i
##  6 -12.8 -25.70000-5.97244i
##  7 -12.7 -25.05000+2.28199i
##  8 -12.7 -25.05000-2.28199i
##  9 -12.6 -19.38801-0.00000i
## 10 -12.6 -29.41199+0.00000i
## # ... with 912 more rows
```

Note that at this point, `x1` is a column of type character, not numeric. Coercion of `x1` to
numeric will be required for plotting.

## Optional plots of real roots


```r
real_roots <- some_roots %>%
  mutate(root = zapsmall(root)) %>%
  filter(Im(root) == 0) %>%
  mutate(
    root = Re(root),
    x1 = as.numeric(x1)
  )

real_roots
```

```
## # A tibble: 900 x 2
##       x1   root
##    <dbl>  <dbl>
##  1 -12.6 -19.4 
##  2 -12.6 -29.4 
##  3 -12.5 -16.3 
##  4 -12.5 -31.2 
##  5 -12.4 -13.9 
##  6 -12.4 -32.3 
##  7 -12.3 -11.7 
##  8 -12.3 -33.2 
##  9 -12.2  -9.76
## 10 -12.2 -33.8 
## # ... with 890 more rows
```


```r
ggplot(real_roots, aes(x=x1, y=root)) +
  geom_point()
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-12-1.svg" width="672" style="display: block; margin: auto;" />

# Lists of matrices with rownames

Let's first generate some data.


```r
phi <- function(n) {
  out <- matrix(
    runif(n),
    dimnames = list(
      stringi::stri_rand_strings(
        n = n,
        length = sample(3:8, size=n, replace=TRUE),
        pattern = "[a-z]"
      ),
      "probability"
    )
  )
  
  out / colSums(out)
}
```


```r
topic_term_prob <- rep(5, 3) %>%
  set_names(., paste0("topic", 1:length(.))) %>%
  map(phi)

topic_term_prob
```

```
## $topic1
##          probability
## sta       0.25680383
## tfli      0.22490878
## chvmalc   0.08163767
## gbxzbril  0.15485806
## venmmxqg  0.28179166
## 
## $topic2
##          probability
## mhy       0.25738923
## dnalha    0.01656447
## kcybybw   0.19192038
## pdoakeox  0.20242189
## knmrhfx   0.33170403
## 
## $topic3
##        probability
## wwt     0.02444428
## jawymz  0.37437258
## orxtty  0.28268967
## nfxzi   0.17689973
## vfbql   0.14159375
```

I'd like to convert this into a single tibble via row-binding, while moving the rownames into its
own column, and also creating another column to keep track of the topic from which these topic-term
distributions originated.

The `as_tibble` method for matrices indeed supports the `rownames` argument, allowing input of a
single string for the name of new column of rownames. This is not really documented on the
[`as_tibble` reference](https://tibble.tidyverse.org/reference/as_tibble.html) under
`# S3 method for matrix`, but is documented under
[`as_tibble.matrix should keep rownames attribute #288`](https://github.com/tidyverse/tibble/issues/288#issuecomment-334244077).

For the topic numbers, we can once again make use of the `.id` argument of `map_dfr`.


```r
topic_term_prob_tibble <- topic_term_prob %>%
  map_dfr(as_tibble, rownames="term", .id="topic")

topic_term_prob_tibble
```

```
## # A tibble: 15 x 3
##    topic  term     probability
##    <chr>  <chr>          <dbl>
##  1 topic1 sta           0.257 
##  2 topic1 tfli          0.225 
##  3 topic1 chvmalc       0.0816
##  4 topic1 gbxzbril      0.155 
##  5 topic1 venmmxqg      0.282 
##  6 topic2 mhy           0.257 
##  7 topic2 dnalha        0.0166
##  8 topic2 kcybybw       0.192 
##  9 topic2 pdoakeox      0.202 
## 10 topic2 knmrhfx       0.332 
## 11 topic3 wwt           0.0244
## 12 topic3 jawymz        0.374 
## 13 topic3 orxtty        0.283 
## 14 topic3 nfxzi         0.177 
## 15 topic3 vfbql         0.142
```

<img src="./img/satisfying.png" style="display: block; margin: auto;" />

## Optional plots of topic term distributions


```r
topic_term_prob_tibble <- topic_term_prob_tibble %>%
  mutate(term = reorder_within(term, by=probability, within=topic))

ggplot(topic_term_prob_tibble, aes(x=probability, y=term, fill=topic)) +
  geom_col(show.legend=FALSE) +
  scale_y_reordered() +
  facet_wrap(~topic, scales="free_y")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-17-1.svg" width="672" style="display: block; margin: auto;" />
