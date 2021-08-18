---
title: "He's a cold hearted snake!"
author: Adam Shen
date: '2021-08-18'
slug: 2021-08-18-cold-hearted
categories:
  - r
  - text-data
tags: [r, text-data]
subtitle: ''
summary: "Analysing the lyrics to Paula Abdul's song, \"Cold Hearted\""
featured: no
---



I have been organizing a series of R workshops at work to introduce my colleagues to R, as well as
the tidyverse. The most recent workshop focused on working with text data where we looked at the
lyrics to Paula Abdul's "Cold Hearted", a song [supposedly](https://genius.com/3689571) written 
about her ex-boyfriend, John Stamos.

# Packages


```r
library(tidyverse)
library(geniusr)
library(magick)
```



# Obtain the lyrics

The lyrics can be obtained using the `geniusr` package and the code below. An API token is *not*
required for the searching of lyrics. I usually use `genius` for the scraping of lyrics but lately 
I've been using `geniusr` since `genius` has been encountering very strange issues with the scraping
of lyrics.


```r
cold <- get_lyrics_search(artist_name="Paula Abdul", song_title="Cold Hearted")

head(cold)
```


```
## # A tibble: 6 x 5
##   line                       section_name section_artist song_name   artist_name
##   <chr>                      <chr>        <chr>          <chr>       <chr>      
## 1 He's a cold-hearted snake  Chorus       Paula Abdul    Cold Heart~ Paula Abdul
## 2 Look into his eyes, uh-oh  Chorus       Paula Abdul    Cold Heart~ Paula Abdul
## 3 He's been tellin' lies     Chorus       Paula Abdul    Cold Heart~ Paula Abdul
## 4 He's a lover boy at play   Chorus       Paula Abdul    Cold Heart~ Paula Abdul
## 5 He don't play by rules, u~ Chorus       Paula Abdul    Cold Heart~ Paula Abdul
## 6 Girl, don't play the fool~ Chorus       Paula Abdul    Cold Heart~ Paula Abdul
```

Since this data only contains one artist and one song, we can actually just drop all columns except
the one containing the lyrics. I'm also going to convert everything to lowercase to make the
later regexing a bit easier.


```r
cold <- cold %>%
  select(line) %>%
  mutate(line = str_to_lower(line))
```

I'm going to do some additional light cleaning to make the text a bit more *formal*.


```r
cold <- cold %>%
  mutate(
    line = str_replace_all(line, pattern="in'", replacement="ing"),
    line = str_remove_all(line, pattern="(c-)+"),
    line = str_replace_all(line, pattern="snnnnnake", replacement="snake")
  )
```


# What is she saying about "him"?

I'm mostly interested in the snippets of the lines that mention he/he's ..., e.g. "he's a 
cold-hearted snake", "he don't play by rules", etc.

To do:

First we need to get all the lyrics that mention he/he's. Then, we're going extract the part that 
comes after he/he's, e.g. "he's a cold-hearted snake" &rarr; "a cold-hearted snake".

Before we can visualise it, we're going to need to obtain the counts. In addition:


```r
filter(cold, str_detect(line, pattern="don't play by"))
```

```
## # A tibble: 6 x 1
##   line                             
##   <chr>                            
## 1 he don't play by rules, uh-oh    
## 2 he don't play by rules, uh-oh    
## 3 he don't play by rules, uh-oh    
## 4 he don't play by the rules, uh-oh
## 5 ooh, ahh, he don't play by rules 
## 6 he don't play by the rules, uh-oh
```

some of the lyrics will need to be collapsed since we have "don't play by rules, uh-oh",
"don't play by *the* rules, uh-oh", and "don't play by rules". In the initial data, there
was also "he's cold as ice" and "he's c-cold as ice", but that was taken care of in the
initial cleaning.

Finally, we can visualise the counts. I also want to include a picture of John Stamos' face in the
plot, since this song is *supposedly* about him.

## Get the relevant lyrics and extract the descriptions

Technically, we should only remove the first instance of he/he's in a line, e.g.
"do you really think he thinks about you when he's out?".


```r
he <- cold %>%
  filter(str_detect(line, pattern="\\bhe('s)?\\b")) %>%
  mutate(
    line = str_extract(line, pattern="\\bhe('s)?\\b.+"),
    line = str_remove(line, pattern="\\bhe('s)?\\b"),
    line = str_squish(line)
  )

he
```

```
## # A tibble: 34 x 1
##    line                           
##    <chr>                          
##  1 a cold-hearted snake           
##  2 been telling lies              
##  3 a lover boy at play            
##  4 don't play by rules, uh-oh     
##  5 needs it                       
##  6 off and running with the crowd 
##  7 thinks about you when he's out?
##  8 a cold-hearted snake           
##  9 been telling lies              
## 10 a lover boy at play            
## # ... with 24 more rows
```

## Collapse similar lyrics

Since the lyric lines are eventually going to be plotted, I'm going to convert them to sentence
case.


```r
he <- he %>%
  mutate(
    line = if_else(str_detect(line, pattern="don't play by"), "don't play by rules", line),
    line = str_to_sentence(line)
  )
```

## Get the counts

To avoid overplotting, I'm only going to keep the lines that appeared more than once.


```r
he_counts <- he %>%
  count(line) %>%
  arrange(desc(n)) %>%
  filter(n > 1)

he_counts
```

```
## # A tibble: 5 x 2
##   line                     n
##   <chr>                <int>
## 1 Been telling lies        7
## 2 A cold-hearted snake     6
## 3 Don't play by rules      6
## 4 A lover boy at play      5
## 5 Cold as ice              3
```

## Visualisation

### Prep the image of John Stamos

Grabbed his headshot from his Wikipedia page.

I don't want him to be the centre of attention in the plot. So I'm going to fade his picture out a
bit to match the plotting background (white).


```r
stamos_face <- image_read("./img/stamos-face.png") %>%
  image_colorize(opacity=85, color="#ffffff")

plot(stamos_face)
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-13-1.png" width="672" style="display: block; margin: auto;" />

### Sort the lyric lines by their counts

A bar chart of unsorted counts is no fun to read. To ensure that our plot will have bars sorted
by the counts, we will need to convert the text to factor and define an order within them.

The lines should be sorted by ascending count, since we'll have text on the y-axis and counts on
the x-axis.


```r
he_counts_reordered <- he_counts %>%
  mutate(line = fct_reorder(line, n))
```

### Make the plot

Since order matters, I'm first going to initialise the canvas, plot his face, then draw the bars
on top.


```r
ggplot(he_counts_reordered, aes(x=n, y=line)) +
  annotation_raster(stamos_face, xmin=4.75, xmax=7.5, ymin=-0.5, ymax=2.75) +
  geom_col() +
  labs(
    title="Sketch of an ex-boyfriend", 
    x="Frequency", y="", 
    caption="Source: Paula Abdul, \"Cold Hearted\""
  ) +
  theme_minimal()
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-15-1.png" width="672" style="display: block; margin: auto;" />

# Conclusions

If your boyfriend has been telling lies, doesn't play by rules, is a cold-hearted snake, a lover boy
at play, and/or is cold as ice, you've found yourself the perfect *ex*-boyfriend.

> *"Girl, don't play the fool"* &mdash; Paula Abdul
