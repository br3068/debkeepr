
<!-- README.md is generated from README.Rmd. Please edit that file -->
debkeepr: Analysis of Non-Decimal Currencies and Double-Entry Bookkeeping
=========================================================================

[![Travis build status](https://travis-ci.org/jessesadler/debkeepr.svg?branch=master)](https://travis-ci.org/jessesadler/debkeepr) [![Coverage status](https://codecov.io/gh/jessesadler/debkeepr/branch/master/graph/badge.svg)](https://codecov.io/github/jessesadler/debkeepr?branch=master)

`debkeepr` provides an interface for analyzing non-decimal currencies that use the tripartite system of pounds, shillings, and pence. It includes functions to apply arithmetic and financial operations to single or multiple values and to analyze account books that use either [single-entry bookkeeping](https://en.wikipedia.org/wiki/Single-entry_bookkeeping_system) or [double-entry bookkeeping](https://en.wikipedia.org/wiki/Double-entry_bookkeeping_system) with the latter providing the name for `debkeepr`. The use of non-decimal currencies throughout the medieval and early modern period presents difficulties for the analysis of historical accounts. The pounds, shillings, and pence system complicates even relatively simple arithmetic manipulations, as each unit has to be [normalized](https://en.wikipedia.org/wiki/Arithmetic#Compound_unit_arithmetic) or converted to the correct base.

`debkeepr` implements two classes — `lsd` and `lsd_list` — to store pounds, shillings, and pence values and associate the values with bases for the shillings and pence units. The `lsd` class stores pounds, shillings, and pence values as numeric vectors of length three and possesses a bases attribute for the shillings and pence units. The `lsd_list` class consists of a list of `lsd` objects that have the same bases attribute that also possesses its own matching bases attribute. In addition, `debkeepr` provides an interface for analyzing pounds, shillings, and pence values stored as three separate variables in a data frame.

The system of recording value according to pounds, shillings, and pence — or to use the Latin terms from which the English derived [libra](https://en.wikipedia.org/wiki/French_livre), [solidus](https://en.wikipedia.org/wiki/Solidus_(coin)), and [denarius](https://en.wikipedia.org/wiki/Denarius) — developed in the 7th and 8th century in the Carolingian Empire. The [ratios](https://en.wikipedia.org/wiki/Non-decimal_currency) between a libra, solidus, and denarius, or [lsd](https://en.wikipedia.org/wiki/%C2%A3sd) for short, were never completely uniform, but most commonly there were 12 denarii in a solidus and 20 solidi in a libra. The custom of counting coins in dozens (solidus) and scores of dozens (libra) spread throughout the Carolingian Empire and became engrained in much of Europe until decimalization after the French Revolution.

Installation
------------

`debkeepr` is still under development and in beta. Basic functionality should be stable at this point, but there could still be some breaking changes. You can install `debkeepr` from GitHub with [devtools](https://github.com/hadley/devtools):

``` r
# install.packages("devtools")
devtools::install_github("jessesadler/debkeepr")
```

Overview
--------

-   All of the functions in `debkeepr` begin with the prefix `deb_`, which is short for double-entry bookkeeping.
-   The nomenclature used throughout the package follows the [original Latin terms](https://en.wikipedia.org/wiki/%C2%A3sd) in using `l`, `s`, and `d` to represent librae, solidi, and denarii respectively and to refer to such values as lsd values. These terms were translated into the various European languages.
    -   English: pounds, shillings, pence
    -   French: livres, sols or sous, deniers
    -   Italian: lire, soldi, denari
    -   Flemish: ponden, schellingen, groten or penningen
-   The functions are designed to be used with three types of input objects:
    1.  Numeric vectors of length 3 in which the value for the first position represents librae (`l`), the second position solidi (`s`), and the third position denarii (`d`). Such lsd vectors can either be a single numeric vector or a list of such vectors. The `lsd` and `lsd_list` classes attach a bases value for the solidus and denarius units to lsd vectors and lists of lsd vectors.
    2.  A data frame that contains pounds, shillings, and pence variables alongside any other variables. The pounds, shillings, and pence columns can have any desired names, but the default is to have the columns named “l”, “s”, and “d” respectively.
    3.  The final object is a data frame that mimics the structure of an account book and can be thought of as a transactions data frame. In addition to pounds, shillings, and pence variables that denote the value of each transaction, a transactions data frame contains variables recording the [creditor and debtor](https://en.wikipedia.org/wiki/Debits_and_credits) for each transaction.
-   There are equivalent functions to manipulate lsd vectors and data frames with lsd variables. Almost anything that can be done on an lsd vector can also be done to a data frame with lsd values. Functions that use a transactions data frame that also possess credit and debit variables do not have equivalent functions of lsd vectors.

### lsd objects

`debkeepr` provides a consistent manner to manipulate single or sets of lsd values. Pounds, shillings, and pence values are stored as either numeric vectors of length 3 or lists of numeric vectors of length 3 that possess a bases attribute to record the bases for the solidus and denarius units of the values. An object of class `lsd` is a named numeric vector with a bases attribute. An `lsd_list` object builds upon the `lsd` class. It is a list of `lsd` objects that share the same bases, and it possesses a matching bases attribute itself.

`lsd` and` lsd_list` objects are created using the `deb_as_lsd()` function, which accepts a numeric vector of length 3 or a list of such vectors for the lsd value(s) and a numeric vector of length 2 for the bases attribute of the lsd object. All functions in `debkeepr` that accept `lsd` and `lsd_list` objects will also accept numeric vectors and lists of numeric vectors alongside a bases argument. While `deb_as_lsd()` is the most explicit way to create an `lsd` or `lsd_list` object, all `debkeepr` functions that use lsd vectors convert any numeric vectors to `lsd` or `lsd_list` objects.

The default for the bases argument in `debkeepr` functions is `c(20, 12)`, which conforms to the most widely used system of 1 pound = 20 shillings and 1 shilling = 12 pence. However, there also existed [systems that used different bases](https://en.wikipedia.org/wiki/Non-decimal_currency) for the solidus and denarius units. For example, the [money of account](https://en.wikipedia.org/wiki/Unit_of_account) in the [Dutch Republic](https://en.wikipedia.org/wiki/Stuiver) consisted of gulden, stuivers, and penningen. Like the more prevalent system of libra, solidus, and denarius, there were 20 stuivers in a gulden, but there were 16 penningen in a stuiver. `lsd` and `lsd_list` object keep track of the solidus and denarius units through the bases attribute. The bases attribute of the object is used instead of the bases argument in `debkeepr` functions. You can access the bases attribute with `deb_bases()`, and the only way to change the bases attribute is to do so explicitly with `deb_convert_bases()`.

``` r
# Load debkeepr
library(debkeepr)

# Create lsd vector equivalent to £5 8s. 11d.
lsd_vector <- c(5, 8, 11)

# Create an object of class lsd
lsd <- deb_as_lsd(lsd = c(5, 8, 11), bases = c(20, 12))

# Create an object of class lsd for guilders value
guilders <- deb_as_lsd(lsd = c(7, 16, 13), bases = c(20, 16))

# Check the bases of guilders
deb_bases(guilders)
#>  s  d 
#> 20 16

# Create a list of lsd vectors
list_lsd_vector <- list(c(12, 7, 9), c(5, 8, 11), c(3, 18, 5))

# Create an object of class lsd_list
lsd_list <- deb_as_lsd(lsd = list_lsd_vector, bases = c(20, 12))
```

`debkeepr` also works with data frames that contain separate variables for pounds, shillings, and pence. The functions default to use “l”, “s”, and “d” as the names of the pounds, shillings, and pence variables, so using these names simplifies the use of the functions.

``` r
# Create data frame with lsd values
lsd_df <- data.frame(l = c(12, 5, 3),
                     s = c(7, 8, 18),
                     d = c(9, 11, 5))
```

`debkeepr` has a set of helper functions to go between lists of lsd vectors and data frames with lsd variables. `deb_list_to_df()` transforms a list of lsd vectors into a [tibble](https://tibble.tidyverse.org) object. `deb_df_to_list()` does the opposite. Any non-lsd variables in the data frame are dropped in order to create numeric vectors of length 3.

``` r
# list of lsd vectors to tibble or tbl_df
deb_list_to_df(lsd = lsd_list)
#> # A tibble: 3 x 3
#>       l     s     d
#>   <dbl> <dbl> <dbl>
#> 1    12     7     9
#> 2     5     8    11
#> 3     3    18     5

# data frame to list of lsd vectors
deb_df_to_list(df = lsd_df, l = l, s = s, d = d)
#>      l  s  d
#> [1] 12  7  9
#> [2]  5  8 11
#> [3]  3 18  5
```

Transaction data frames present a way to record data from an account book. They consist of five variables: "credit", "debit", "l", "s", and "d". The credit and debit variables contain information about the accounts involved in the transactions, and the lsd variables record the value of the transactions. Following the principles of double-entry bookkeeping, the credit account is the account that gives or sends the value, and the debit account receives the value. The `debkeepr` functions that work specifically on transaction data frames use "credit" and "debit" as the default names for these variables, though one could follow the convention of [network analysis](http://kateto.net/network-visualization) and name the variables "from" and "to".

Here is an example transactions data frame with four accounts named "a", "b", "c", and "d" and 15 transactions with values randomly created:

``` r
set.seed(240)
transactions_df <- data.frame(credit = sample(letters[1:4], 15, replace = TRUE),
                              debit = sample(letters[1:4], 15, replace = TRUE),
                              l = sample(1:30, 15, replace = TRUE),
                              s = sample(1:19, 15, replace = TRUE),
                              d = sample(1:11, 15, replace = TRUE),
                              stringsAsFactors = FALSE)
```

`debkeepr` contains two data sets from the example journal and ledger in Richard Dafforne's *Merchant's Mirrour* from 1660. `dafforne_transactions` is a transactions data frame with 177 transactions. `dafforne_accounts` possesses information about the accounts in the journal and ledger.

lsd vectors
-----------

lsd vectors and lists of such vectors simplify the process of doing quick calculations on values in historical sources. A particularly useful function for one-off uses is `deb_normalize()`, which normalizes an lsd vector or list of vectors to the desired bases for the `s` and `d` units. Thus, adding together a set of lsd values by hand might result in the non-standard form of £10 64s. 21d. `deb_normalize()` normalizes the value to the proper bases for the solidus and denarius units.

``` r
# Normalize a non-standard lsd vector
deb_normalize(lsd = c(10, 64, 21))
#>      l s d
#> [1] 13 5 9
```

The bases for the solidus and denarius units can be changed on to any value with the bases argument.

``` r
# Normalize a non-standard lsd vector with alternative bases
deb_normalize(lsd = c(10, 64, 21), bases = c(20, 16))
#>      l s d
#> [1] 13 5 5
```

Another option available in all of the functions that deal with lsd values is to round the pence or denarius unit to a specified decimal place. The `round` argument helps to simplify the output and avoids the issue of rounding lsd values to a non-normalized value. The default is to round to the 5th decimal place.

``` r
# Using round outside of debkeepr functions can lead to non-normalized value
round(c(9, 19, 11.999999), digits = 5)
#> [1]  9 19 12

# Normalization applied within rounding
deb_normalize(lsd = c(9, 19, 11.999999), round = 5)
#>      l s d
#> [1] 10 0 0

# Rounding is useful to return all pence values as whole numbers
deb_normalize(lsd = c(5, 11, 8.95), round = 0)
#>     l  s d
#> [1] 5 11 9
```

You can normalize multiple lsd values by placing lsd vectors into a list. The below example demonstrates some of the types of values that can be properly normalized by `deb_normalize()`. The function accepts negative values or even a mix of positive and negative values within an lsd vector. Any of the values can also possess decimalized values. All functions in `debkeepr` that work with lsd vectors also accept lists of lsd vectors.

``` r
# To normalize multiple lsd values use a list of lsd vectors
deb_normalize(lsd = list(c(4, 34, 89), c(-9, -75, -19), c(15.85, 36.15, 56)))
#>       l   s    d
#> [1]   6   1  5.0
#> [2] -12 -16 -7.0
#> [3]  17  17  9.8
```

### Arithmetic functions

`debkeepr` contains functions to perform arithmetic operations such as addition, subtraction, multiplication, and division on lsd vectors.

There are two different ways to add lsd vectors and/or lists of lsd vectors. `deb_sum()` takes any number of lsd vectors and/or lists of lsd vectors and returns a single numeric vector of length 3 that is the sum of the input values. `deb_add()` has a different use case. It only accepts two lsd vectors and/or lists of lsd vectors: `lsd1` and `lsd2`. If `lsd1` and `lsd2` in `deb_add()` are both vectors, the output will be equivalent to `deb_sum()`. However, if one or both of the inputs are lists of lsd vectors, the output will be the length of the longer list and each element of the list is added to the equivalent element of the other input. If one input is shorter than the other, the shorter input will be recycled. `deb_subtract()` is equivalent to `deb_add()` with `lsd2` subtracted from `lsd1`.

``` r
# Use deb_sum to reduce multiple lsd values to a single value
deb_sum(c(8, 14, 11), c(5, 13, 8))
#>      l s d
#> [1] 14 8 7
deb_sum(lsd_list, c(5, 13, 8))
#>      l s d
#> [1] 27 8 9

# deb_add is the same as deb_sum if inputs are two lsd vectors
deb_add(lsd1 = c(8, 14, 11), lsd2 = c(5, 13, 8))
#>      l s d
#> [1] 14 8 7

# If one input is a list, the output is also a list and thus different
# from deb_sum. Here, £5 13s. 8d. is added to each lsd vector in lsd_list.
deb_add(lsd1 = lsd_list, lsd2 = c(5, 13, 8))
#>      l  s d
#> [1] 18  1 5
#> [2] 11  2 7
#> [3]  9 12 1

# deb_subtract has the same use case as deb_add
deb_subtract(lsd1 = c(8, 14, 11), lsd2 = c(5, 13, 8))
#>     l s d
#> [1] 3 1 3

# Subtract lsd vector from a list of lsd vectors
deb_subtract(lsd1 = lsd_list, lsd2 = c(5, 13, 8))
#>      l   s  d
#> [1]  6  14  1
#> [2]  0  -4 -9
#> [3] -1 -15 -3
```

Multiplication and division work similarly to `deb_add()` and `deb_subtract()`, but instead of using a second lsd value, `deb_multiply()` and `deb_divide()` multiply and divide an lsd vector or list of such vectors by a single value: `x`.

``` r
# Multiplcation
deb_multiply(lsd = c(5, 13, 8), x = 5)
#>      l s d
#> [1] 28 8 4

# Multiplication of a list of lsd vectors
deb_multiply(lsd = lsd_list, x = 5)
#>      l  s d
#> [1] 61 18 9
#> [2] 27  4 7
#> [3] 19 12 1

# Division
deb_divide(lsd = c(136, 17, 9), x = 5)
#>      l s   d
#> [1] 27 7 6.6

# Division of a list of lsd vectors
deb_divide(lsd = lsd_list, x = 3)
#>     l  s       d
#> [1] 4  2 7.00000
#> [2] 1 16 3.66667
#> [3] 1  6 1.66667
```

Because `lsd` is always the first argument in `debkeepr` functions that deal with lsd vectors and the inputs and outputs are the same, `debkeepr` functions can be chained together with the pipe (`%>%`). For instance, you can find the revenue gained by each of two partners who worked on a commission of 3% of goods valued at £1583 15s. 8d.

``` r
# 3% commision between two partners
deb_multiply(lsd = c(1583, 15, 9), x = 0.03) %>% 
  deb_divide(x = 2)
#>      l  s     d
#> [1] 23 15 1.635

# Here, it may be practical to round to the nearest pence
deb_multiply(lsd = c(1500, 15, 8), x = 0.03) %>% 
  deb_divide(x = 2, round = 0)
#>      l  s d
#> [1] 22 10 3
```

Changing the value for the `round` argument may be especially useful when dealing with multiplication and division. For example, an lsd value divided by a number and then the result multiplied by the same number may not be equivalent because of the default rounding.

``` r
# Not equivalent due to default rounding
deb_divide(c(6, 8, 1), x = 3) %>% 
  deb_multiply(x = 3)
#>     l s       d
#> [1] 6 8 0.99999

# Can be solved by reducing the rounding argument
deb_divide(c(6, 8, 1), x = 3) %>% 
  deb_multiply(x = 3, round = 4)
#>     l s d
#> [1] 6 8 1
```

### Financial functions

Alongside arithmetic operations, `debkeepr` possesses functions to do common financial operations such as calculate interest and the exchange between different currencies. `deb_interest()` possesses arguments for the interest rate, the duration over which the interest should be calculated, and whether to include the principal or not in the answer. The default interest rate is 6.25%, which was the standard interest rate in the Low Countries at the end of the 16th century.

``` r
# Interest rate at 6.25% over a 5 year period with principal
deb_interest(lsd = c(100, 0, 0), interest = 0.0625, duration = 5)
#>       l s d
#> [1] 131 5 0

# Calculate only the accrued interest
deb_interest(lsd = c(100, 0, 0), interest = 0.0625, duration = 5, with_principal = FALSE)
#>      l s d
#> [1] 31 5 0
```

Exchange between different currencies creates a number of potential problems, but `debkeepr` provides multiple ways to handle the exchange between currencies. The most straightforward method for calculating the exchange between two currencies is to use `deb_exchange()`, which uses shillings to compare the currencies. This was a common unit used in providing an exchange rate between similar currencies. If the exchange rate includes a pence value, you can either convert to a decimalized solidus by hand or add the pence over the base of the pence as shown below. However, the exchange rate may not always be given in terms of shillings. For instance, a rate might be provided as 90d. This could be normalized to shillings, but in this case it may be easier to use `deb_multiply()` and compare 90d. to 240d. in a pound.

``` r
# Exchange between currencies at rate of 31 shillings or 1 to 1.55
deb_exchange(lsd = c(100, 0, 0), shillings_rate = 31)
#>       l s d
#> [1] 155 0 0

# Exchange rate of 31s. 4d.
deb_exchange(lsd = c(100, 0, 0), shillings_rate = 31 + 4/12)
#>       l  s d
#> [1] 156 13 4

# Exchange rate of 90d. using deb_multiply
deb_multiply(lsd = c(100, 0, 0), x = 90 / 240)
#>      l  s d
#> [1] 37 10 0
```

`debkeepr` provides two functions that can help in dealing with exchange rates between currencies. `deb_exchange_rate()` accepts two lsd vectors of lists of vectors to find the exchange rate between the values. The first argument, `lsd1`, is the fixed currency that is reduced to £1 and `lsd2` is the variable currency. Because it may not always make sense to return the exchange rate in a single format, this function has an option to return the exchange rate as a normalized lsd vector, in terms of shillings and pence as used by `deb_exchange()`, or just in terms of pence. The second helper function is `deb_invert_rate()`. Given an exchange rate, `deb_invert_rate()` provides a means to find the exchange rate in the opposite direction. Like `deb_exchange_rate()`, it has options to return the exchange rate as a normalized lsd vector, in terms of shillings and pence, or just in terms of pence. The default output for both `deb_exchange_rate()` and `deb_invert_rate()` is shillings and pence.

``` r
# Find the exchange rate between two values
deb_exchange_rate(lsd1 = c(100, 0, 0), lsd2 = c(166, 13, 4))
#>     l  s d
#> [1] 0 33 4

# Find the same rate in terms of pence
deb_exchange_rate(lsd1 = c(100, 0, 0), lsd2 = c(166, 13, 4), output = "pence")
#>     l s   d
#> [1] 0 0 400

# The above exchange rate is 33s. 4d. Can find the exchange in the
# opposite direction by changing lsd1 and lsd2
deb_exchange_rate(lsd1 = c(166, 13, 4), lsd2 = c(100, 0, 0))
#>     l  s d
#> [1] 0 12 0

# Or you can use deb_invert_rate
deb_invert_rate(exchange_rate = c(0, 33, 4))
#>     l  s d
#> [1] 0 12 0

# This can also be expressed in terms of pence
deb_invert_rate(exchange_rate = c(0, 33, 4), output = "pence")
#>     l s   d
#> [1] 0 0 144
```

The most challenging exchange between currencies takes place when the bases for the solidus and denarius units differ between the two currencies. Though the system of 20-base solidus and 12-base denarius was widespread, various currencies used other bases. `deb_convert_bases()` provides a way to convert from one base system to another. It also has a `ratio` argument that can be used similarly to `deb_multiply()` to include an exchange rate between currencies in addition to the base conversion. A good example of this is the conversion between guilders and Flemish pounds, which were both used in the Low Countries in the seventeenth century. Flemish pounds used the 20 and 12 base system, but guilders consisted of 20 stuivers of 16 penningen. In addition, the guilders and Flemish pounds currencies were tied together at a rate of 6 guilders to £1 Flemish. `deb_convert_bases()` can perform this and various other conversions between forms of measurement that use the tripartite based system of lsd.

``` r
# Convert Flemish pounds to guilders
deb_convert_bases(lsd = c(104, 8, 3),
                  bases1 = c(20, 12),
                  bases2 = c(20, 16),
                  ratio = 6)
#>       l s d
#> [1] 626 9 8

# Convert guilders to FLemish pounds
deb_convert_bases(lsd = c(1224, 19, 8),
                  bases1 = c(20, 16),
                  bases2 = c(20, 12),
                  ratio = 1 / 6)
#>       l s d
#> [1] 204 3 3

# Convert French crowns of 60 sous and 12 deniers to pound sterling
# at the rate of 72d. French crowns equals £1 sterling or 240d. sterling
deb_convert_bases(lsd = c(214, 50, 10),
                  bases1 = c(60, 12),
                  bases2 = c(20, 12),
                  ratio = 72 / 240)
#>      l s d
#> [1] 64 9 1

# Base conversion can also be done in concert with deb_exchange()
# Convert from guilders to pounds sterling at the rate of 12s. Flemish
deb_convert_bases(lsd = c(1224, 19, 8),
                  bases1 = c(20, 16),
                  bases2 = c(20, 12),
                  ratio = 1 / 6) %>% 
  deb_exchange(shillings_rate = 12)
#>       l s    d
#> [1] 122 9 11.4
```

Decimalization
--------------

Sometimes it is useful to decimalize lsd values, reducing the values to a single unit with a base of 10. Alternatively, you may come across decimalized values that you want to expand to lsd values. `debkeepr` provides functions to convert between decimalized and non-decimalized values for both vectors and variables in data frames.

All of the decimalization functions follow the naming convention of `input-unit_output-unit`. For instance, to convert from an lsd vector to decimalized librae or pounds you use `deb_lsd_l()`, while conversion between decimalized pounds to lsd values is done with `deb_l_lsd()`. There are functions to do the same process with both shillings and pence.

``` r
# lsd to decimalized denarii
deb_lsd_d(lsd = c(10, 12, 7))
#> [1] 2551

# Decimalized denarii to lsd
deb_d_lsd(d = 2551)
#>      l  s d
#> [1] 10 12 7
```

Useful works on monetary systems and the history of accounting
--------------------------------------------------------------

-   Peter Spufford, *Money and its Use in Medieval Europe* (Cambridge: Cambridge University Press, 1988), especially pages 411–414, for a discussion of money of account in medieval Europe.
-   John Richard Edwards and Stephen P. Walker, eds. *The Routledge Companion to Accounting History* (New York: Routledge Taylor & Francis Group, 2009).
-   John Geijsbeek, *Ancient Double-Entry Bookkeeping: Lucas Pacioli’s Treatise (A.D. 1494, the Earliest Known Writer on Bookkeeping) Reproduced and Translated with Reproductions, Notes, and Abstracts from Manzoni, Pietra, Mainardi, Ympyn, Stevin, and Dafforne* (Denver: John Geijsbeek, 1914).
