---
title: "Testing code"
author: "Laurent Gatto and Stephen Eglen"
---

# Introduction

### Why test?

* Nobody writes buggy code... sure!
* How do you know your code is right unless you test?
* Can act as specification/examples of what the programme should do.

### Inverting some data can have serious consequences ...

http://www.sciencemag.org/content/314/5807/1856.full.pdf

2001 Science paper, and two more Science papers retracted after
inversion along x axis noted.

### Testing needs to be:

* reliable
* fast
* easy to run
* easy to summarise

# stop() vs warning()

* A warning is softer than an error; if a warning is generated
your program will still continue, whereas an error will stop the
program.


```r
log(c(2, 1, -1)); print('end')   # warning 
xor(sum(1, "a"));  print ('end')  # error
```

* If you try to isolate warnings, you can change warnings to
  errors: `options(warn=2)`

* Add warnings and errors to your code using `warning()`,
  `stop()`


```r
x <- -1:1
y <- -5:-1

if (any(x<0)) warning("some elements of x are negative")

if (all(y<0)) stop("all elements of x are negative")

stopifnot(all.equal(pi, 3.1415927),
          2 < 2, all(1:10 < 12),
          "a" < "b")
```

## Catching errors and warnings


```r
safelog <- function(x) {
  tryCatch(log(x),
           error = function(e) "an error",
           warning = function(e) "a warning")
}

safelog(3)
safelog(-5)
safelog("string")
```

## KISS

Keep your functions simple and stupid (and short). 

## Failing fast and well

> Bounds errors are ugly, nasty things that should be stamped out
> whenever possible. One solution to this problem is to use the
> `assert` statement. The `assert` statement tells C++, "This can
> never happen, but if it does, abort the program in a nice way." One
> thing you find out as you gain programming experience is that things
> that can "never happen" happen with alarming frequency. So just to
> make sure that things work as they are supposed to, it’s a good idea
> to put lots of self checks in your program. -- Practical C++
> Programming, Steve Oualline, O'Reilly.


```r
if (!condition) stop(...)
```


```r
stopifnot(TRUE)
stopifnot(TRUE, FALSE)
```

For example to test input classes, lengths, ...


```r
f <- function(x) {
    stopifnot(is.numeric(x), length(x) == 1)
    invisible(TRUE)
}

f(1)
f("1")
f(1:2)
f(letters)
```

The [`assertthat`](https://github.com/hadley/assertthat) package:


```r
x <- "1"
library("assertthat")
stopifnot(is.numeric(x))
assert_that(is.numeric(x))
assert_that(length(x) == 2)
```

* `assert_that()` signal an error.
* `see_if()` returns a logical value, with the error message as an attribute.
* `validate_that()` returns `TRUE` on success, otherwise returns the error as
  a string.


* `is.flag(x)`: is x `TRUE` or `FALSE`? (a boolean flag)
* `is.string(x)`: is x a length 1 character vector?
* `has_name(x, nm)`, `x %has_name% nm`: does `x` have component `nm`?
* `has_attr(x, attr)`, `x %has_attr% attr`: does `x` have attribute `attr`?
* `is.count(x)`: is x a single positive integer?
* `are_equal(x, y)`: are `x` and `y` equal?
* `not_empty(x)`: are all dimensions of `x` greater than 0?
* `noNA(x)`: is `x` free from missing values?
* `is.dir(path)`: is `path` a directory?
* `is.writeable(path)`/`is.readable(path)`: is `path` writeable/readable?
* `has_extension(path, extension)`: does `file` have given `extension`?

# Comparisons

## Floating point issues to be aware of

R FAQ [7.31](http://cran.r-project.org/doc/FAQ/R-FAQ.html#Why-doesn_0027t-R-think-these-numbers-are-equal_003f)?



```r
a <- sqrt(2)
a * a == 2
a * a - 2
```


```r
1L + 2L == 3L
1.0 + 2.0 == 3.0
0.1 + 0.2 == 0.3
```

## Floating point: how to compare

- `all.equal` compares R objects for *near equality*. Takes into
  account whether object attributes and names ought the taken into
  consideration (`check.attributes` and `check.names` parameters) and
  tolerance, which is machine dependent.


```r
all.equal(0.1 + 0.2, 0.3)
all.equal(0.1 + 0.2, 3.0)
isTRUE(all.equal(0.1 + 0.2, 3)) ## when you just want TRUE/FALSE
```

## Exact identity

`identical`: test objects for exact equality


```r
1 == NULL
all.equal(1, NULL)
identical(1, NULL)
identical(1, 1.)   ## TRUE in R (both are stored as doubles)
all.equal(1, 1L)
identical(1, 1L)   ## stored as different types
```

Appropriate within `if`, `while` condition statements. (not
`all.equal`, unless wrapped in `isTRUE`).

# Logging

See for example the [`log4r`](https://github.com/johnmyleswhite/log4r)
package:


```r
## Import the log4r package.
library('log4r')

## Create a new logger object with create.logger().
logger <- create.logger()

## Set the logger's file output: currently only allows flat files.
logfile(logger) <- file.path('base.log')

## Set the current level of the logger.
level(logger) <- "INFO"

## Try logging messages at different priority levels.
debug(logger, 'A Debugging Message') ## Won't print anything
info(logger, 'An Info Message')
warn(logger, 'A Warning Message')
error(logger, 'An Error Message')
fatal(logger, 'A Fatal Error Message')
```
