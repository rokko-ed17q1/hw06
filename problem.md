# hw06: Introduction to R packages
Kenji Sato^[Kobe University. Email: mail@kenjisato.jp]  
`r format(Sys.time(), "%d %B, %Y")`  


> Packages are the fundamental units of reproducible R code. They include reusable R functions, the documentation that describes how to use them, and sample data. --- Hadley Wickham in [_R Packages_](http://r-pkgs.had.co.nz)


# Overview


## Purpose {-}

To learn how to write R packages. 🐱

## Prerequisite {-}

MS Windows users, install development environment first: 

* Rtools: https://cran.r-project.org/bin/windows/Rtools/

Install the following R packages

* **devtools** for writing your packages (and installing others' from source)
* **roxygen2** for documentation
* **testthat** for testing

by executing 


```r
install.packages(c("devtools", "roxygen2", "testthat"))
```

Note: We don't write tests in this assignment. 

## Instructions {-}

In this assignment, you will

- clone the assignment repository and make a working branch (eg. `solution` branch);
- follow the step-by-step instruction in Section \@ref(practice) to create
  an R package; 
- knit `solution.Rmd` to check if your package properly works (`solution.md` and `solution.html` will be produced instead of a pdf file); 
- open a Pull Request.


# Packages and libraries

You can use installed packages from anywhere because `install.packages()` installs packages in standardized locations, called the _libraries_. You can locate your libraries with


```r
.libPaths()
```

```
## [1] "/usr/local/lib/R/3.4/site-library"                                   
## [2] "/usr/local/Cellar/r/3.4.0/R.framework/Versions/3.4/Resources/library"
```

I have many packages installed in the first library, which is for user-installed packages. 


```r
dir(.libPaths()[1])
```

```
##  [1] "assertthat"     "backports"      "base64enc"      "BH"            
##  [5] "bitops"         "blogdown"       "bookdown"       "brew"          
##  [9] "broom"          "callr"          "caTools"        "cellranger"    
## [13] "colorspace"     "commonmark"     "crayon"         "curl"          
## [17] "DBI"            "debugme"        "desc"           "devtools"      
## [21] "dichromat"      "digest"         "dplyr"          "emo"           
## [25] "evaluate"       "forcats"        "formatR"        "ggplot2"       
## [29] "git2r"          "gtable"         "haven"          "highr"         
## [33] "hms"            "htmltools"      "httpuv"         "httr"          
## [37] "hwmgr"          "jsonlite"       "knitr"          "labeling"      
## [41] "lazyeval"       "lubridate"      "macroeconomics" "magrittr"      
## [45] "markdown"       "memoise"        "mime"           "miniUI"        
## [49] "mnormt"         "modelr"         "munsell"        "openssl"       
## [53] "pkgbuild"       "pkgload"        "plyr"           "praise"        
## [57] "processx"       "pryr"           "psych"          "purrr"         
## [61] "R6"             "RColorBrewer"   "Rcpp"           "readr"         
## [65] "readxl"         "rematch"        "reshape2"       "rmarkdown"     
## [69] "roxygen2"       "rprojroot"      "rstudioapi"     "rvest"         
## [73] "scales"         "selectr"        "servr"          "shiny"         
## [77] "sourcetools"    "stringi"        "stringr"        "testthat"      
## [81] "tibble"         "tidyr"          "tidyverse"      "whisker"       
## [85] "withr"          "xaringan"       "xml2"           "xtable"        
## [89] "yaml"
```

The packages shipped with R are placed in the second library.


```r
dir(.libPaths()[2])
```

```
##  [1] "base"         "boot"         "class"        "cluster"     
##  [5] "codetools"    "compiler"     "datasets"     "foreign"     
##  [9] "graphics"     "grDevices"    "grid"         "KernSmooth"  
## [13] "lattice"      "MASS"         "Matrix"       "methods"     
## [17] "mgcv"         "nlme"         "nnet"         "parallel"    
## [21] "rpart"        "spatial"      "splines"      "stats"       
## [25] "stats4"       "survival"     "tcltk"        "tools"       
## [29] "translations" "utils"
```

# Packages on GitHub

Most packages on CRAN have good quality because there is a review process. 
For the same reason, it is difficult to share your packages on CRAN. 

Since CRAN doesn't allow package developers to update their products too often, many package developers have GitHub repositories for their packages to host the source code in development. Users can install the up-to-date version on GitHub by `devtools::install_github()`.

As an example, let's install a small package by Hadley Wickham. https://github.com/hadley/emo

Run the following in the console.


```r
devtools::install_github("hadley/emo")
```

I hope you get the rule. The argument of `devtools::install_github()` is a string of characters in the form of "username/repo-name". 

You can use the installed package just like other packages installed from CRAN. 


```r
emo::ji("smile")
```

```
## 😄
```

I don't think this package is designed for use with `library()` but you can `library()` it if you want to. Then you can call `emo::ji()` function with shorter but less descriptive `ji()`.

# Why write packages?

Now that you can write meaningful functions (eg. `lssr()`), it's time to start learning how to bundle them as a package, install it with `devtools`. 

You may be wondering why `source()`ing script files is not enough. 

Think about a situation that you have two separate scripts `functions.R` and `plot.R` as below. The latter depends on the former.


```r
# functions.R
cobb_douglas_per_capita <- function(A, alpha) {
  function(k) A * k ^ alpha
}
```

`plot.R` does the real job. 


```r
# plot.R
source("functions.R")

f <- cobb_douglas_per_capita(A = 1.1, alpha = 0.3)
plot(f, xlim = c(0, 3))
```


Since `functions.R` defines functions repeatedly used in a project, it can   potentially be used in other projects as well. To source it from within another project, however, you need to do something like 


```r
source("../project-x/functions.R")
```

`source()` the script to use functions defined therein. As you can imagine, it is not easy to move the directories of the calling and called scripts not to mention to share your scripts to others.


In contrast, installed packages can be used from anywhere by `library()`.


```r
library("tibble")
```

It is great if you can use `cobb_douglas_per_capita()` with 


```r
library("macroeconomics")
f <- cobb_douglas_per_capita(A = 1.1, alpha = 0.3)
```

or 


```r
f <- macroeconomics::cobb_douglas_per_capita(A = 1.1, alpha = 0.3)
```

don't you think?


# Make your own package with **devtools** {#practice}

For detailed explanations on how to make R packages, read Hadley Wickham's _R Packages_ (http://r-pkgs.had.co.nz). In this assignment, you will make a small package to experience the workflow. 

First of all, let's import **devtools** package. 


```r
library("devtools")
```

## `devtools::create()`

Package name can contain only alphanumeric characters and dots. It must start with an alphabet [a-zA-Z]. 

`devtools::create()` creates a package skeleton. 


```r
create("macroeconomics")
```

You now have `macroeconomics` folder, where you see several files and empty `R` folder. 

```
.
└── macroeconomics
    ├── .Rbuildignore
    ├── .gitignore
    ├── DESCRIPTION
    ├── NAMESPACE
    ├── R/
    ├── macroeconomics.Rproj
    └── man
```

In the following, I assume that your working directory is set to "hw06", under which you are developing package "macroeconomics". Check your working directory with


```r
getwd()
```

```
## [1] "/Users/kenjisato/Dropbox/Lectures/economic-dynamics/linear-assignments/assignments/hw06"
```

If you prefer to work on the package directory, you can do so by 
`setwd("macroeconomics")` and remove`macroeconomics` argument from all 
devtools function calls below; eg. `load_all("macroeconomics")`
is replaced with `load_all()`. 

## R directory

All R code must live in this directory; sub-directories inside `R` are not allowed. Make a R script file `production-function.R` with the following content and save it in `macroeconomics/R` directory.


```r
cobb_douglas_per_capita <- function(A, alpha) {
  function(k) A * k ^ alpha
}
```


## `devtools::load_all()`

To use and/or test functions defined in the package in development, call `load_all()` function instead of `source()`. 


```r
load_all("macroeconomics")
```

Now you can use it as a regular function. 


```r
cobb_douglas_per_capita(1.1, 0.3)(1)
```


## `devtools::check()`

Let's check if the `macroeconomics` packages follows the R-package guideline. 
Run 


```r
check("macroeconomics")
```

You should see something like

```
R CMD check results
0 errors | 1 warning  | 0 notes
checking DESCRIPTION meta-information ... WARNING
Non-standard license specification:
  What license is it under?
Standardizable: FALSE
```

To get rid of embarrassing "1 warning", you need to set license properly. 
Open DESCRIPTION file, which now looks like

```
Package: macroeconomics
Title: What the Package Does (one line, title case)
Version: 0.0.0.9000
Authors@R: person("First", "Last", email = "first.last@example.com", role = c("aut", "cre"))
Description: What the package does (one paragraph).
Depends: R (>= 3.4.0)
License: What license is it under?
Encoding: UTF-8
LazyData: true
RoxygenNote: 6.0.1
```

The line with "License: What license is it under?" is the cause of the warning. You must set it manually because **devtools** cannot choose license for you.  

Let's use MIT License for this project. For another option, see http://r-pkgs.had.co.nz/description.html#license and references therein. 


```r
use_mit_license("macroeconomics")
```

By this function call, 

* "License: What license is it under?" is changed to "License: MIT + file LICENSE",
* "LICENSE" file is created.

Open `LICENSE` file and write your name in "COPYRIGHT HOLDER" section. 

Then check again.


```r
check("macroeconomics")
```


```
Status: OK

R CMD check results
0 errors | 0 warnings | 0 notes
```

Now you're good to go!

Note: You might want to modify "Title",  "Authors@R" and "Description" sections of `DESCRIPTION` file too. Read http://r-pkgs.had.co.nz/description.html to understand what they are for.

## `devtools::install()`

Now let's install the package. It's as easy as


```r
install("macroeconomics")
```


```
...
** help
No man pages found in package  ‘macroeconomics’ 
*** installing help indices
** building package indices
** testing if installed package can be loaded
* DONE (macroeconomics)
Reloading installed macroeconomics
```

## `library()`

**Unload the package before moving on.** 


```r
unload("macroeconomics")
```

Or more simply, restart R session. 

Since your user library has your own package, the following code should work.


```r
library("macroeconomics")
```

Get no error? Congratulations!

## Exported and non-exported objects

Let's use `cobb_douglas_per_capita` then. Type in the function name and 
RStudio's auto completions should work .... Hmm, not really. 
Something went wrong. 

Let's check if **macroeconomics** really has the function. 


```r
macroeconomics::cobb_douglas_per_capita
```

```
## function (A, alpha) 
## {
##     function(k) A * k^alpha
## }
## <environment: namespace:macroeconomics>
```

The error message says it is not "exported" from the namespace, which 
means the function doesn't exist in the place (namespace) where users of 
the package have access to. This same error is thrown when you try to call
non-existent function with `::` operator but since you definitely defined 
the function (I'm sure you did!), you can use it with `:::` operator.


```r
macroeconomics:::cobb_douglas_per_capita
```

```
## function (A, alpha) 
## {
##     function(k) A * k^alpha
## }
## <environment: namespace:macroeconomics>
```

Remember this rule:

* `::` for exported objects, used by all users.
* `:::` for non-exported objects, internally used for package developers.

The package developers are supposed to provide stable API, meaning that  parameters and return values of exported functions won't change very often.  This doesn't apply to non-exported objects; when you use packages, don't rely on unexporeted objects too much.


## Exporting objects

* Open `production-function.R` 
* Put the cursor inside the definition of `cobb_douglas_per_capita`
* Click on "Code" and then "Insert Roxygen Skeleton" of RStudio's menu. 


```r
#' Title
#'
#' @param A 
#' @param alpha 
#'
#' @return
#' @export
#'
#' @examples
cobb_douglas_per_capita <- function(A, alpha) {
  function(k) A * k ^ alpha
}
```

Now your function has `@export` tag. Run the following command 


```r
document("macroeconomics")
```
```
Updating macroeconomics documentation
Loading macroeconomics
Warning: @return [production-function.R#7]: requires a value
Warning: @examples [production-function.R#10]: requires a value
Writing NAMESPACE
Writing cobb_douglas_per_capita.Rd
```

Open up `NAMESPACE` file, which now has the following contents.

```
# Generated by roxygen2: do not edit by hand

export(cobb_douglas_per_capita)
```

The line with `export(cobb_douglas_per_capite)` does the job of exporting 
the function. You don't need to manage `NAMESPACE` yourself because **roxygen2** and **devtools** packages do that for you.

Before install the updated package, let's `check()` again.


```r
check("macroeconomics")
```

```
R CMD check results
0 errors | 1 warning  | 0 notes
checking Rd contents ... WARNING
Argument items with no description in Rd object 'cobb_douglas_per_capita':
  ‘A’ ‘alpha’
```

You have another warning again, which says that there is missing
documentation. 

## Documentation

Since exported functions are made for other users, they must have 
documentation. Open `man/cobb_douglas_per_capita.Rd`, which was created
with `devtools::document()`. 

```
% Generated by roxygen2: do not edit by hand
% Please edit documentation in R/production-function.R
\name{cobb_douglas_per_capita}
\alias{cobb_douglas_per_capita}
\title{Title}
\usage{
cobb_douglas_per_capita(A, alpha)
}
\arguments{
\item{A}{}

\item{alpha}{}
}
\description{
Title
}
```

That warning was raised because `\item{A}{}` and `\item{alpha}{}` have empty
contents in the second braces. 


To fix this, open `R/production-function.R` again and write proper documents 
for the function.  For example,


```r
#' Per-capita Cobb-Douglas production function
#'
#' @param A double > 0, total factor productivity
#' @param alpha double in (0, 1), capital share
#'
#' @return Cobb-Douglas function in its per-capita form
#' @export
#'
#' @examples
cobb_douglas_per_capita <- function(A, alpha) {
  function(k) A * k ^ alpha
}
```

Let's check again (`devtools::document` is called by default).



```r
check("macroeconomics")
```

```
...
Status: OK

R CMD check results
0 errors | 0 warnings | 0 notes
```

You can now install. 

* Restart R session because this package is already loaded.
* Load **devtools**
* Install with `devtools::install()`


```r
library("devtools")
install("macroeconomics")
```

Then load the installed package.


```r
library("macroeconomics")
```

View help with 


```r
?cobb_douglas_per_capita
```


Congratulations! You can now create your own R-package!

To finish the homework, knit `solution.Rmd` in hw06 folder. (You don't get PDF for this assignment.)

