Introduction
============

This document gives an overview that broadly covers the following
information:

-   What is tidy data?
-   Why use the `tidyverse`?
    -   `%>%`
    -   `mutate`
    -   `tibble`
    -   miscellaneous data processing tools in the `tidyverse`

What is tidy data?
==================

In short, tidy data has one row per observation, and one column per
measurement. For example, the `who` data set is well organized and
compact, but it is does not meet the definition of tidy:

    # load the who data set from the tidyr package
    require(tidyr)
    data(who)

    # display who
    who

    ## # A tibble: 7,240 x 60
    ##    country     iso2  iso3   year new_sp_m014 new_sp_m1524 new_sp_m2534
    ##    <chr>       <chr> <chr> <int>       <int>        <int>        <int>
    ##  1 Afghanistan AF    AFG    1980          NA           NA           NA
    ##  2 Afghanistan AF    AFG    1981          NA           NA           NA
    ##  3 Afghanistan AF    AFG    1982          NA           NA           NA
    ##  4 Afghanistan AF    AFG    1983          NA           NA           NA
    ##  5 Afghanistan AF    AFG    1984          NA           NA           NA
    ##  6 Afghanistan AF    AFG    1985          NA           NA           NA
    ##  7 Afghanistan AF    AFG    1986          NA           NA           NA
    ##  8 Afghanistan AF    AFG    1987          NA           NA           NA
    ##  9 Afghanistan AF    AFG    1988          NA           NA           NA
    ## 10 Afghanistan AF    AFG    1989          NA           NA           NA
    ## # ... with 7,230 more rows, and 53 more variables: new_sp_m3544 <int>,
    ## #   new_sp_m4554 <int>, new_sp_m5564 <int>, new_sp_m65 <int>,
    ## #   new_sp_f014 <int>, new_sp_f1524 <int>, new_sp_f2534 <int>,
    ## #   new_sp_f3544 <int>, new_sp_f4554 <int>, new_sp_f5564 <int>,
    ## #   new_sp_f65 <int>, new_sn_m014 <int>, new_sn_m1524 <int>,
    ## #   new_sn_m2534 <int>, new_sn_m3544 <int>, new_sn_m4554 <int>,
    ## #   new_sn_m5564 <int>, new_sn_m65 <int>, new_sn_f014 <int>,
    ## #   new_sn_f1524 <int>, new_sn_f2534 <int>, new_sn_f3544 <int>,
    ## #   new_sn_f4554 <int>, new_sn_f5564 <int>, new_sn_f65 <int>,
    ## #   new_ep_m014 <int>, new_ep_m1524 <int>, new_ep_m2534 <int>,
    ## #   new_ep_m3544 <int>, new_ep_m4554 <int>, new_ep_m5564 <int>,
    ## #   new_ep_m65 <int>, new_ep_f014 <int>, new_ep_f1524 <int>,
    ## #   new_ep_f2534 <int>, new_ep_f3544 <int>, new_ep_f4554 <int>,
    ## #   new_ep_f5564 <int>, new_ep_f65 <int>, newrel_m014 <int>,
    ## #   newrel_m1524 <int>, newrel_m2534 <int>, newrel_m3544 <int>,
    ## #   newrel_m4554 <int>, newrel_m5564 <int>, newrel_m65 <int>,
    ## #   newrel_f014 <int>, newrel_f1524 <int>, newrel_f2534 <int>,
    ## #   newrel_f3544 <int>, newrel_f4554 <int>, newrel_f5564 <int>,
    ## #   newrel_f65 <int>

The `who` data set has information from the World Health Organization on
TB cases recorded by country, age and sex. It is not tidy because
multiple variables are encoded in many of the columns, and incidence is
spread over columns 5 through 60 (e.g. `new_sp_m014` contains incidence
for males between 0 and 14 years of age, who were diagnosed using a
positive pulmonary smear). To covert this to a tidy format we could do
the following:

    # save country codes in a separate table
    isoCodes <- select(who, country, iso2, iso3) %>%
                unique()

    # tidy the who data set
            # gather incidence into one column
    who2 <- gather(who, 'group', 'incidence', -country, -iso2, -iso3, -year) %>%
     
            # spread group out into separate columns
            mutate(group = str_replace(group, 'new_?', ''),        # remove "new" and "new_" from the group name
                            
                   diagnosis = str_split_fixed(group, '_', 2)[,1], # grab the diagnosis type
                   
                   sexAgeCat = str_split_fixed(group, '_', 2)[,2], # grab sex/age category
                   sex = substr(sexAgeCat, 1, 1),                  #    first character is sex
                   ageCat = substr(sexAgeCat, 2, 5)) %>%           #    next 4 characters are age category
        
            # drop rows with missing incidence
            filter(!is.na(incidence)) %>%

            # drop redundant columns
            select(-group, -iso2, -iso3, -sexAgeCat)

    who2

    ## # A tibble: 76,046 x 6
    ##    country      year incidence diagnosis sex   ageCat
    ##    <chr>       <int>     <int> <chr>     <chr> <chr> 
    ##  1 Afghanistan  1997         0 sp        m     014   
    ##  2 Afghanistan  1998        30 sp        m     014   
    ##  3 Afghanistan  1999         8 sp        m     014   
    ##  4 Afghanistan  2000        52 sp        m     014   
    ##  5 Afghanistan  2001       129 sp        m     014   
    ##  6 Afghanistan  2002        90 sp        m     014   
    ##  7 Afghanistan  2003       127 sp        m     014   
    ##  8 Afghanistan  2004       139 sp        m     014   
    ##  9 Afghanistan  2005       151 sp        m     014   
    ## 10 Afghanistan  2006       193 sp        m     014   
    ## # ... with 76,036 more rows

For some additional information on tidy data, see [Ravi's
slides](blob/master/TidyingData-StatForLunch.pdf) from his Statistics
for Lunch talk.

Why use the `tidyverse`?
========================

There are many reasons to use `tidyverse` - I've listed my favorite
parts of the `tidyverse` here, but there are many more that we don't
have room to discuss. For starters, `library(tidyverse)` will load the
following packages (it is really an empty package with a bunch of
dependencies):

-   ggplot2: This is a completely new way of generating graphics in R
    that is based on the grammar of graphics.
-   tibble: This implements a reimmagining of the `data.frame`. The
    `tibble` or `data_frame` is fairly backward compatible with
    functions that work with the `data.frame`. The only exception I know
    of is the `missForest` package.
-   tidyr: Provides `gather` and `spread` functions for data tidying.
-   readr: Redesigned import functions for reading csv, tsv... text data
    files.
-   purrr: New tools, similar to the `apply` family, for working with
    the `tibble`.
-   dplyr: Data manipulation tools.
-   stringr: String manipulation tools.
-   forcats: Categorical data (i.e. `factor`) manipulation tools.
-   magrittr: This package isn't actually loaded, but the pipe operator,
    `%>%`, is imported.

There are a number of additional dependencies that are installed when
you use the `dependencies = TRUE` option (click on the links for more
details):

-   [broom](https://github.com/tidyverse/broom)
-   [cli](https://github.com/r-lib/cli)
-   [crayon](https://github.com/r-lib/crayon)
-   [dbplyr](https://github.com/tidyverse/dbplyr)
-   [haven](https://github.com/tidyverse/haven)
-   [hms](https://github.com/tidyverse/hms)
-   [httr](https://cran.r-project.org/web/packages/httr/index.html)
-   [jsonlite](https://cran.r-project.org/web/packages/jsonlite/index.html)
-   [lubridate](https://github.com/tidyverse/lubridate)
-   [modelr](https://github.com/tidyverse/modelr)
-   [readxl](https://github.com/tidyverse/readxl)
-   [reprex](https://github.com/tidyverse/reprex)
-   [rlang](https://cran.r-project.org/web/packages/rlang/index.html)
-   [rstudioapi](https://cran.rstudio.com/web/packages/rstudioapi/index.html)
-   [rvest](https://cran.r-project.org/web/packages/rvest/rvest.pdf)
-   [xml2](https://cran.r-project.org/web/packages/xml2/index.html)
-   [feather](https://blog.rstudio.com/2016/03/29/feather/)
-   [knitr](https://cran.r-project.org/web/packages/knitr/index.html)
-   [rmarkdown](https://cran.r-project.org/web/packages/markdown/index.html)

Pipes
-----

The `magrittr` package was merged into the `tidyverse` early in its
development, and the `tidyverse` makes extensive use of the pipe
operator, `%>%`. Compare the code above to this code used to tidy the
`who` data set.

    # drop redundant country information
    who3 <- subset(who, select = c(1, 4:60))

    ##### tidy the who data set #####
    # melt who3
    require(reshape2)
    who3 <- melt(who3, id = c('country', 'year'), na.rm = TRUE, value.name = 'incidence')

    # split `variable` into diagnosis/sexAgeCat vectors
    tmp <- strsplit(gsub("new_?", "", as.character(who3$variable)), "_")

    who3$diagnosis <- sapply(tmp, `[`, 1)            # get diagnosis
    who3$sex <- substr(sapply(tmp, `[`, 2), 1, 1)    # get sex
    who3$ageCat <- substr(sapply(tmp, `[`, 2), 2, 5) # get age category

    # get rid of `variable`
    who3$variable <- NULL

    str(who3)

    ## 'data.frame':    76046 obs. of  6 variables:
    ##  $ country  : chr  "Afghanistan" "Afghanistan" "Afghanistan" "Afghanistan" ...
    ##  $ year     : int  1997 1998 1999 2000 2001 2002 2003 2004 2005 2006 ...
    ##  $ incidence: int  0 30 8 52 129 90 127 139 151 193 ...
    ##  $ diagnosis: chr  "sp" "sp" "sp" "sp" ...
    ##  $ sex      : chr  "m" "m" "m" "m" ...
    ##  $ ageCat   : chr  "014" "014" "014" "014" ...

-   `tidyverse` functions were built with `%>%` in mind, so we always
    know that the first argument is data. This makes passing output from
    one function to the next easy. Other traditional R functions are
    inconsistent (e.g. the first argument of `substr` is the data, but
    it is the third argument of `gsub`).
-   It is clear in the `tidyverse` code where the tidying begins and
    ends.
-   Each step of tidying is well defined (e.g.
    `gather %>% mutate %>% filter %>% select`).
-   `who2` is only created once, while we have to modify `who3` several
    times before the data are tidy.
-   String parsing is much more readable when using the pipes.

Mutate
------

We saw an example of `mutate` above. In general, using the `dplyr`
package will result in cleaner, simpler code, and it will save you time
(e.g. because your code will be less buggy). There is no real equivalent
to `mutate` in base R.

-   `mutate` allows you to add all new variables in one construct,
    rather than spreading them out over multiple lines of code.
-   `mutate` (as well as `data_frame`) allows you to use variables you
    have created in the same function call (e.g. we created variables
    `group` and `sexAgeCat` above and were able to use them to create
    other variables in the same call to `mutate`).

tibbles
-------

In addition to what has already been discussed, `tibble` has a few more
features I would like to highlight:

-   Variables in `data.frame` are limited to vectors, but in
    `data_frame` you can have a list (e.g. `who2$group` was a list of
    string vectors).
-   `tibble` is more computationally efficient.
-   Import functions (e.g. `read_csv`) are more computationally
    efficient.

Click [here](blob/master/1-Tidyverse.md) for more information and
examples.

Miscelaneous data processing tools in the `tidyverse`
-----------------------------------------------------

### lubridate

The
[lubridate](https://github.com/tidyverse/lubridate/blob/master/vignettes/lubridate.Rmd)
package has some additional functions that will help us work with dates
and time.

-   `now()` essentially does the same thing as `Sys.time()` by default,
    but you can optionally specify a different timezone.
-   `second()`, `minute()`, `hour()`, `day()`, `year()`, ... allow you
    to extract or set (change) the specified part of a POSIXlt formatted
    variable.
-   `seconds()`, `minutes()`, `hours()`, `days()`, `years()`, ... allow
    you to specify a period of time (e.g. `days(5)` is 5 days).

Examples:

    require(lubridate)

    # by default, these both produce the same result
    Sys.time()

    ## [1] "2018-04-11 10:52:35 EDT"

    now()

    ## [1] "2018-04-11 10:52:35 EDT"

    # however, now() can be used to get the time in a different timezone
    now("GMT")

    ## [1] "2018-04-11 14:52:35 GMT"

    #### the Wall Street Market crash of 1929
    crash <- strptime("Oct 29, 1929 9:30 AM", format = "%B %d, %Y %H:%M %p")

    # what day of the week was the Wall Street Market crash of 1929?
    wday(crash, label = TRUE)

    ## [1] Tue
    ## Levels: Sun < Mon < Tue < Wed < Thu < Fri < Sat

    # really the crash started on Monday and continued into Tuesday
    # this is the interval over which the crash happened
    crash <- interval(crash - days(1),                        # Monday @ 9:30 AM
                      crash + period(hours = 6, minutes = 30))# Tuesday @ 4:00 PM


    #### arithmetic can be interesting
                        # leap year    not leap year
    jan28 <- strptime(c("2016-01-28", "2017-01-28"), format = "%Y-%m-%d")

    # we can add a month and then a day is OK
    jan28 + months(1) + days(1)

    ## [1] "2016-02-29 EST" "2017-03-01 EST"

    # adding a day then a month can be problematic
    jan28 + days(1) + months(1)

    ## [1] "2016-02-29 EST" NA

Note: there seems to be a bug with R's handling of the default time zone
settings on some systems (as of early 2018). If you happen to run into
this error,

    Error in (function (dt, year, month, yday, mday, wday, hour, minute, second,  : 
      Invalid timezone of input vector: ""

this bit of code will set things straight (or something similar, e.g.
EDT or GMT):

    Sys.setenv(TZ="EST")

### stringr

The [stringr]() package reproduces many of the same functions from base
R, but their names and arguments are much more consistent (e.g. the
input string is always the first argument, thus they work better with
`%>%`). Some examples that I like include:

-   Instead of the single function, `grep`, `stringr` employs three
    different functions - I find this easier to read and easier to use.
    -   `str_detect` is the equivalent of `grepl`.
    -   `str_which` is the equivalent of the default `grep`.
    -   `str_subset` is the equivalent of `grep(..., value = TRUE)`.
-   `str_split_fixed` returns a matrix of substrings (`str_split` is
    equivalent to `strsplit`).
-   `str_to_title` selectively capitalizes the first letter of each
    word - suitable for using as a title.

Examples:

    require(stringr)

    # group names from the who data set that we parsed above
    strings <- names(who)[5:9]

    # Find all columns from the 0-14 age group
    grepl("014", strings)

    ## [1]  TRUE FALSE FALSE FALSE FALSE

    str_detect(strings, "014")

    ## [1]  TRUE FALSE FALSE FALSE FALSE

    grep("014", strings)

    ## [1] 1

    str_which(strings, "014")

    ## [1] 1

    grep("014", strings, value = TRUE)

    ## [1] "new_sp_m014"

    str_subset(strings, "014")

    ## [1] "new_sp_m014"

    # remove "new_" from the string and split on _ (return list)
    gsub("new_?", "", strings) %>%
        strsplit("_")

    ## [[1]]
    ## [1] "sp"   "m014"
    ## 
    ## [[2]]
    ## [1] "sp"    "m1524"
    ## 
    ## [[3]]
    ## [1] "sp"    "m2534"
    ## 
    ## [[4]]
    ## [1] "sp"    "m3544"
    ## 
    ## [[5]]
    ## [1] "sp"    "m4554"

    str_replace(strings, "new_?", "") %>%
        str_split("_")

    ## [[1]]
    ## [1] "sp"   "m014"
    ## 
    ## [[2]]
    ## [1] "sp"    "m1524"
    ## 
    ## [[3]]
    ## [1] "sp"    "m2534"
    ## 
    ## [[4]]
    ## [1] "sp"    "m3544"
    ## 
    ## [[5]]
    ## [1] "sp"    "m4554"

    # remove "new_" from the string and split on _ (return matrix)
    gsub("new_?", "", strings) %>%
        strsplit("_") %>%
        unlist() %>%
        matrix(ncol = 2, byrow = TRUE)    

    ##      [,1] [,2]   
    ## [1,] "sp" "m014" 
    ## [2,] "sp" "m1524"
    ## [3,] "sp" "m2534"
    ## [4,] "sp" "m3544"
    ## [5,] "sp" "m4554"

    str_replace(strings, "new_?", "") %>%
        str_split_fixed("_", 2)

    ##      [,1] [,2]   
    ## [1,] "sp" "m014" 
    ## [2,] "sp" "m1524"
    ## [3,] "sp" "m2534"
    ## [4,] "sp" "m3544"
    ## [5,] "sp" "m4554"
