---
layout: post
title:  "Introduction to Developing an R Package"
date:   2018-01-06
excerpt: "As a data scientist, we might have solved a problem and want to distribute our solution so that everyone else can use it. In this post, we will cover the basics about R package."
image: "/images/Rlogo.png"
author: [Shivayogi Biradar, Bagus Trihatmaja]
---

Distributing our code as an R package is a good way to share our work. By creating R package, we make our code reusable, not only for us, but also
for everyone else. We don't reinvent the wheel. If someone has solved the problem, then we can use it. That way we minimise the development time.

# The tools

There are tools available that will make our live easier when developing R package:
1. [devtools](https://cran.r-project.org/web/packages/devtools/index.html) helps with building and installing packages
2. [roxygen2](https://cran.r-project.org/web/packages/roxygen2/index.html) for documentation streamline
3. [testthat](https://cran.rstudio.com/web/packages/testthat/index.html) as a unit test framework

# R package structure

R package is actually a directory with required files and directories. Inside R package, we will see this structure:

| Name | Type | Is Required | Note |
|-------|-----|----------|-------|
|DESCRIPTION | File | Yes | A structured  description of the package |
|NAMESPACE | File | Yes | Imports package dependencies and exports our functions and classes |
|R | Folder | Yes | This is where we put all of our sources|
|man | Folder | Yes | The documentation folder |
|src | Folder | No | If we use other language such as C, C++, etc., then we put the compiled files here |
|inst| Folder | No | Misc. files to move to the package's root directory when installed |
|data| Folder | No | Contains .RData or .rda files |
|tests| Folder | No | Contains test files |
|vignettes| Folder | No | Vignettes illustrating the package's use |
|NEWS| File| No| Changelog file |

# How does a package work?

There are five states of a package:

## 1. Source package

This contains just a mandatory directories and files we have discussed above

## 2. Bundled package

In this state, the package has been built and compressed into `tar.gz`. If we extract the built, then we will see the same source package as number 1.

## 3. Binary package

In this state, the package is built as binary for specific OS architechture. Right now, only the most common operating systems are supported (Windows, and MacOS).

## 4. Installed package

This is the state after we do `install.packages()` command. The package is downlaoded and extracted to the package library.

## 5. In memory package

After we call, `library(ourpackage)`, then the package will be loading into library. We can also load the package into memory by calling the function with namespace.
For example, `package::function()`. The difference is the latter won't be included in the search path. So we need to define the namespace every time we call the function.

# The steps

## Create the initial package structure

We can do this by using `devtools::create("package_name")`.

## Fill the DESCRIPTION file

Example of the DESCRIPTION file

```
Package: solution10
Version: 0.0.0.9000
Title: Plot MSI image
Description: Solution of number 8 and 9 as a package.
Authors@R: person("Anak Agung Ngurah Bagus", "Trihatmaja", ,"trihatmaja.a@husky.neu.edu", c("aut", "cre"))
License: Artistic-2.0
Encoding: UTF-8
LazyData: true
ByteCompile: true
Imports: 
    ggplot2,
    tibble
RoxygenNote: 6.0.1
```

The fields are mostly self-explanatory. One thing we should notice is `Imports`. The `Imports` is the list of package we need for our package to be running but won't be attached to the search path.
There is also `Suggests`, which is the list of packages we use but does not necessarily needed for the user to install. 

## Implement the source codes

There are other steps we need to perform, not only by putting our source codes in the `R` folder. 

To make our package usable, we need to create a proper documentation. We use special comment `#'` with special code, so that Roxygen2 can autogenerate the documentation for us.
The complete explanation, you can read it [here](http://r-pkgs.had.co.nz/man.html).

```r
#' Mass Spectrometry Inspector
#'
#' \code{msi} build MSI
#'
#' @param mz The mz data we load
#' @param intensity The intensity 
#' @param coord_x X coordinate of the image
#' @param coord_y Y coordinate of the image
#'
#' @return An object of class \code{msi}.
#'
#' @examples
#' library(msipotter)
#' msi(mz, intensity, coord_x, coord_y)
#'
#' @export
msi <- function(mz, intensity, coord_x, coord_y) structure(list(
      mz = mz,
      intensity = simplify2array(intensity),
      coord = tibble(x = coord_x, y = coord_y)
    ),
    class = "MSI")
```

Notice the `@export` above. That comment is important for Roxygen2 to export our method to public in `NAMESPACE` file.

If we build the package, you will see this in your NAMESPACE file.

```
export(msi)
```
You can put the special comment according to your needs

## Write the tests

Test is an important aspect in developing a package but often underestimated. Writing test seems tedious at the beginning but it saves a lot of time in the future. For example, let's say today you write a function A that depends on another function B.
One year later, you want to change function B. Without unit test, you might not realize that you might have broken function A. Tests can also be a proof, that a bug does not appear again in our package.
Imagine in the future without unit tests, you have to test manually all the bug that has appeared. Now, which one is more tedious?

Actually, it is better to write the test first. We are not going to discuss why. If you are curious about the motivation, you can see it [here](https://softwareengineering.stackexchange.com/questions/41409/why-does-tdd-work).
Here, let's put that we write the test after implementing the code.

Here is a simple example of a test.

```r
test_that("prepareCompanyInfo must return at least one value", {
  expect_gt(nrow(prepareCompanyInfo(fundamental_and_prices_2016)), 0)
  expect_s3_class(getAllCompanyInfo(), "tbl")
  })
```

`TestThat` has a convention on how we should name the test files. Inside test folder, there will be another folder for testthat. Inside the testthat folder, we should name our file according to the file name in R folder.
For example, if the filename in our R folder is `companies.R` then we have to name our test file as `test-companies.R`. This way `testthat` will know which R file does the test belong to.

## Build the package

To build the package, we can use devtools command.

```r
devtools::build("package_name")
```
Or simply

```r
devtools::build()
```

To take a source package as an input.

## Test the package

If we want to submit our package to CRAN or bioconductor, we have to make sure all the tests are passed without errors, warnings, or notes.

To check the package, we can use:

```r
devtools::check()
```

## Install the package

To install our package, we can use:

```r
install.packages("package_name_0.0.1.tar.gz", repos=NULL, type="source")
```


Resources:

[1] Wickham, Hadley. R Packages. http://r-pkgs.had.co.nz

