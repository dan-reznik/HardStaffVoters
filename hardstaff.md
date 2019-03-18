---
title: "R Notebook"
output: 
  html_document:
    keep_md: true
---

Dear Emily, have fun!


```r
library(tidyverse)
```

```
## ── Attaching packages ──────────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.1.0       ✔ purrr   0.3.1  
## ✔ tibble  2.0.1       ✔ dplyr   0.8.0.1
## ✔ tidyr   0.8.3       ✔ stringr 1.4.0  
## ✔ readr   1.3.1       ✔ forcats 0.4.0
```

```
## ── Conflicts ─────────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

Simulate your data frame


```r
sim_rows <- 1000
set.seed(0)
parties <- c("Labour","Conservative","Spont.Other","UKIP","Lib Dem")
# skew it a bit
party_probs <- c(20,200,5,5,1) %>% {./sum(.)}
ids=1:100

df_voters <- tibble(pipd=sample(ids,sim_rows,replace=T),
                    party=sample(parties,sim_rows,replace=T,prob=party_probs)) %>%
  arrange(pipd) %>%
  group_by(pipd) %>%
  mutate(wave=c(letters,LETTERS)[row_number()]) %>%
  ungroup() %>%
  select(pipd,wave,party)

df_voters
```

```
## # A tibble: 1,000 x 3
##     pipd wave  party       
##    <int> <chr> <chr>       
##  1     1 a     Conservative
##  2     1 b     Conservative
##  3     1 c     UKIP        
##  4     1 d     Conservative
##  5     1 e     Conservative
##  6     1 f     Conservative
##  7     1 g     Conservative
##  8     2 a     Conservative
##  9     2 b     Conservative
## 10     2 c     Conservative
## # … with 990 more rows
```

Voting history


```r
df_voting_history <- df_voters %>%
  group_by(pipd) %>%
  summarize(party_n=n_distinct(party),
            party_order=list(rle(party))
            #first_party=map_chr(first(party_order),"values"),
            #second_party=nth(party_order,2)
            ) %>%
  mutate(voted_parties=map(party_order,"values"),
         voted_times=map(party_order,"lengths"),
         party_1st=map_chr(voted_parties,first),
         party_2nd=map_chr(voted_parties,~{if(length(.x)==1) NA else .x[2]}),
         flipped=!is.na(party_2nd)) %>%
  select(-party_order) %>%
  select(pipd,party_1st,party_2nd,flipped,everything())

df_voting_history
```

```
## # A tibble: 100 x 7
##     pipd party_1st    party_2nd   flipped party_n voted_parties voted_times
##    <int> <chr>        <chr>       <lgl>     <int> <list>        <list>     
##  1     1 Conservative UKIP        TRUE          2 <chr [3]>     <int [3]>  
##  2     2 Conservative <NA>        FALSE         1 <chr [1]>     <int [1]>  
##  3     3 Conservative <NA>        FALSE         1 <chr [1]>     <int [1]>  
##  4     4 Conservative Labour      TRUE          2 <chr [3]>     <int [3]>  
##  5     5 Conservative UKIP        TRUE          3 <chr [5]>     <int [5]>  
##  6     6 Conservative Labour      TRUE          3 <chr [6]>     <int [6]>  
##  7     7 Conservative Spont.Other TRUE          2 <chr [3]>     <int [3]>  
##  8     8 Labour       Conservati… TRUE          2 <chr [2]>     <int [2]>  
##  9     9 Labour       Conservati… TRUE          2 <chr [3]>     <int [3]>  
## 10    10 Conservative <NA>        FALSE         1 <chr [1]>     <int [1]>  
## # … with 90 more rows
```

