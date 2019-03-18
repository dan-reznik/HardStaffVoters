
<!-- README.md is generated from README.Rmd. Please edit that file -->
Dear Emily, have fun!
---------------------

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0       ✔ purrr   0.3.1  
    ## ✔ tibble  2.0.1       ✔ dplyr   0.8.0.1
    ## ✔ tidyr   0.8.3       ✔ stringr 1.4.0  
    ## ✔ readr   1.3.1       ✔ forcats 0.4.0

    ## ── Conflicts ───────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

Simulating your data frame
--------------------------

Skewing toward Conservative votes...

``` r
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
```

|  pipd| wave | party        |
|-----:|:-----|:-------------|
|     1| a    | Conservative |
|     1| b    | Conservative |
|     1| c    | UKIP         |
|     1| d    | Conservative |
|     1| e    | Conservative |
|     1| f    | Conservative |
|     1| g    | Conservative |
|     2| a    | Conservative |
|     2| b    | Conservative |
|     2| c    | Conservative |

Voting history
--------------

The results you want appear in the party\_1st, party\_2nd, and "flipped" columns

``` r
df_voting_history <- df_voters %>%
  group_by(pipd) %>%
  summarize(party_n=n_distinct(party),
            party_order=list(rle(party))) %>%
  mutate(voted_parties=map(party_order,"values"),
         voted_times=map(party_order,"lengths"),
         party_1st=map_chr(voted_parties,first),
         party_2nd=map_chr(voted_parties,~{if(length(.x)==1) NA else .x[2]}),
         flipped=!is.na(party_2nd)) %>%
  select(-party_order) %>%
  select(pipd,party_1st,party_2nd,flipped,everything())
```

|  pipd| party\_1st   | party\_2nd   | flipped |  party\_n| voted\_parties                                                                       | voted\_times        |
|-----:|:-------------|:-------------|:--------|---------:|:-------------------------------------------------------------------------------------|:--------------------|
|     1| Conservative | UKIP         | TRUE    |         2| c("Conservative", "UKIP", "Conservative")                                            | c(2, 1, 4)          |
|     2| Conservative | NA           | FALSE   |         1| Conservative                                                                         | 9                   |
|     3| Conservative | NA           | FALSE   |         1| Conservative                                                                         | 8                   |
|     4| Conservative | Labour       | TRUE    |         2| c("Conservative", "Labour", "Conservative")                                          | c(7, 1, 3)          |
|     5| Conservative | UKIP         | TRUE    |         3| c("Conservative", "UKIP", "Conservative", "Labour", "Conservative")                  | c(4, 1, 1, 1, 1)    |
|     6| Conservative | Labour       | TRUE    |         3| c("Conservative", "Labour", "Conservative", "Labour", "Spont.Other", "Conservative") | c(2, 1, 2, 1, 1, 6) |
|     7| Conservative | Spont.Other  | TRUE    |         2| c("Conservative", "Spont.Other", "Conservative")                                     | c(1, 1, 11)         |
|     8| Labour       | Conservative | TRUE    |         2| c("Labour", "Conservative")                                                          | c(1, 12)            |
|     9| Labour       | Conservative | TRUE    |         2| c("Labour", "Conservative", "Labour")                                                | c(1, 4, 1)          |
|    10| Conservative | NA           | FALSE   |         1| Conservative                                                                         | 8                   |

Hope this helps!
----------------
