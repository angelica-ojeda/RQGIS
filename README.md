
<!-- README.md is generated from README.Rmd. Please edit that file -->

#### General

<!-- badges: start -->

[![Lifecycle:
retired](https://img.shields.io/badge/lifecycle-retired-orange.svg)](https://www.tidyverse.org/lifecycle/#retired)
<!-- badges: end --> [![minimal R
version](https://img.shields.io/badge/R%3E%3D-3.2.0-6666ff.svg)](https://cran.r-project.org/)
[![Last-changedate](https://img.shields.io/badge/last%20change-2019--10--07-yellowgreen.svg)](/commits/master)

#### CRAN

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/RQGIS)](http://cran.r-project.org/package=RQGIS)
[![Downloads](http://cranlogs.r-pkg.org/badges/RQGIS?color=brightgreen)](http://www.r-pkg.org/pkg/RQGIS)
![](http://cranlogs.r-pkg.org/badges/grand-total/RQGIS)

<!-- C:\OSGeo4W64\bin\python-qgis -> opens Python!!
/usr/share/qgis/python/plugins/processing-->

# Attention RQGIS users

**RQGIS** only supports QGIS**2** (latest LTR version: 2.14; latest dev
version: 2.18), which is no longer actively developed by the QGIS
community. If you would like to use QGIS**3** from within R, please use
[RQGIS3](https://github.com/jannes-m/RQGIS3).

# Description

**RQGIS** establishes an interface between R and QGIS, i.e., it allows
the user to access QGIS functionalities from within R. It achieves this
by establishing a tunnel to the Python QGIS API via the
[reticulate-package](https://github.com/rstudio/reticulate). This
provides the user with an extensive suite of GIS functions, since QGIS
allows you to call native as well as third-party algorithms via its
processing framework (see also
<https://docs.qgis.org/2.14/en/docs/user_manual/processing/index.html>).
Third-party providers include among others GDAL, GRASS GIS, SAGA GIS,
the Orfeo Toolbox, TauDEM and tools for LiDAR data. **RQGIS** brings you
this incredibly powerful geoprocessing environment to the R console.

Please check also out our paper presenting **RQGIS** in
detail:

<div style="text-align:center">

<a href = "https://rjournal.github.io/archive/2017/RJ-2017-067/RJ-2017-067.pdf">https://rjournal.github.io/archive/2017/RJ-2017-067/RJ-2017-067.pdf</a>

</div>

<p align="center">

<img src="https://raw.githubusercontent.com/jannes-m/RQGIS/master/figures/r_qgis_puzzle.png" width="40%"/>

</p>

The main advantages of **RQGIS** are:

1.  It provides access to QGIS functionalities. Thereby, it calls Python
    QGIS API but R users can stay in their programming environment of
    choice without having to touch Python.
2.  It offers a broad suite of geoalgorithms making it possible to solve
    most GIS problems.
3.  R users can use just one package (**RQGIS**) instead of using
    **RSAGA** and **rgrass7** to access SAGA and GRASS functions. This,
    however, does not mean that **RSAGA** and **rgrass7** are obsolete
    since both packages offer various other advantages. For instance,
    **RSAGA** provides many user-friendly and ready-to-use GIS functions
    such as `rsaga.slope.asp.curv()` and `multi.focal.function()`.

# Package installation

In order to run **RQGIS** properly, you need to download various
third-party software packages. Our vignette should help you with the
download and installation procedures on various platforms (Windows,
Linux, Mac OSX). To access it, use `vignette("install_guide", package =
"RQGIS")`.

You can install:

  - the latest released version from CRAN with:

<!-- end list -->

``` r
install.packages("RQGIS")
```

  - the latest **RQGIS** development version from Github with:

<!-- end list -->

``` r
devtools::install_github("r-spatial/RQGIS")
```

# RQGIS usage

Subsequently, we will show you a typical workflow of how to use
**RQGIS**. Basically, we will follow the steps also described in the
[QGIS
documentation](https://docs.qgis.org/2.14/en/docs/user_manual/processing/console.html).
In our first and very simple example we simply would like to retrieve
the centroid coordinates of a spatial polygon object. First off, we will
download the administrative areas of Germany using the raster package.

``` r
# attach packages
library("raster")
library("rgdal")

# download German administrative areas 
ger = getData(name = "GADM", country = "DEU", level = 1)
# ger is of class "SpatialPolygonsDataFrame"
```

Now that we have a spatial object, we can move on to using **RQGIS**.
First of all, we need to specify all the paths necessary to run the
QGIS-API. Fortunately, `set_env()` does this for us (assuming that QGIS
and all necessary dependencies were installed correctly). The only thing
we need to do is: specify the root path to the QGIS-installation. If you
do not specify a path, `set_env()` tries to find the
OSGeo4W-installation first in the ‘C:/OSGeo4W’-folders. If this is
unsuccessful, it will search your C: drive though this might take a
while. If you are running **RQGIS** under Linux or on a Mac, `set_env()`
assumes that your root path is “/usr” and
“/applications/QGIS.app/Contents”, respectively. Please note, that
most of the **RQGIS** functions, you are likely to work with (such as
`find_algorithms()`, `get_args_man()` and `run_qgis`), require the
output list (as returned by `set_env()`) containing the paths to the
various installations necessary to run QGIS from within R. This is why,
`set_env()` caches its result in a temporary folder, and loads it back
into R when called again (to overwrite an existing cache, set parameter
`new` to `TRUE`).

``` r
# attach RQGIS
library("RQGIS")

# set the environment, i.e. specify all the paths necessary to run QGIS from 
# within R
set_env()
# under Windows set_env would be much faster if you specify the root path:
# set_env("C:/OSGeo4W~1")


## $root
## [1] "C:\\OSGeo4W64"
##
## $qgis_prefix_path
## [1] "C:\\OSGeo4W64\\apps\\qgis-ltr"
##
## $python_plugins
## [1] "C:\\OSGeo4W64\\apps\\qgis-ltr\\python\\plugins"
```

Secondly, we would like to find out how the function in QGIS is called
which gives us the centroids of a polygon shapefile. To do so, we use
`find_algorithms()`. Here, we look for a geoalgorithm that contains the
words “polygon” and “centroid”.  
Note that you can use regular expressions.

``` r
library("RQGIS")
find_algorithms(search_term = "([Pp]olygon)(centroid)")

## [1] "Polygon centroids------------------------------------>qgis:polygoncentroids"
## [2] "Polygon centroids------------------------------------>saga:polygoncentroids"
```

This gives us two functions we could use. Here, we’ll choose the QGIS
function named `qgis:polygoncentroids`. Subsequently, we would like to
know how we can use it, i.e., which function parameters we need to
specify.

``` r
get_usage(alg = "qgis:polygoncentroids")

## ALGORITHM: Polygon centroids
##  INPUT_LAYER <ParameterVector>
##  OUTPUT_LAYER <OutputVector>
```

Consequently `qgis:polygoncentroids` only expects a parameter called
`INPUT_LAYER`, i.e., the path to a spatial polygon object whose centroid
coordinates we wish to extract, and a parameter called `OUTPUT_LAYER`,
i.e., the path where we would like to store the output object. Since it
would be tedious to specify manually each and every function argument,
especially if a function has more than two or three arguments, we have
written a convenience function, named `get_args_man()`, that retrieves
all function arguments and respective default values for a given GIS
function. It returns these values in the form of a list. If a function
argument lets you choose between several options (drop-down menu in a
GUI), setting `get_arg_man()`’s `options`-argument to `TRUE` makes sure
that the first option will be selected (QGIS GUI behavior). For example,
`qgis:addfieldtoattributestable` has three options for the
`FIELD_TYPE`-parameter, namely integer, float and string. Setting
`options` to `TRUE` means that the field type of your new column will be
of type integer.

``` r
params = get_args_man(alg = "qgis:polygoncentroids")
params
## $INPUT_LAYER
## [1] "None"
## 
## $OUTPUT_LAYER
## [1] "None"
```

In our case, `qgis:polygoncentroids` has only two function arguments and
no default values. Naturally, we need to specify manually our input and
output layer. We can do so in two ways. Either we use directly our
parameter-argument list…

``` r
params$INPUT_LAYER  = ger
params$OUTPUT_LAYER = file.path(tempdir(), "ger_coords.shp")
out = run_qgis(alg = "qgis:polygoncentroids",
               params = params,
               load_output = TRUE)
## $OUTPUT_LAYER
## [1] "C:\\Users\\pi37pat\\AppData\\Local\\Temp\\Rtmpeil4bS/ger_coords.shp"
```

… or we can use R named arguments in `run_qgis()`

``` r
out = run_qgis(alg = "qgis:polygoncentroids",
               INPUT_LAYER = ger,
               OUTPUT_LAYER = file.path(tempdir(), "ger_coords.shp"),
               load_output = TRUE)
## $OUTPUT_LAYER
## [1] "C:\\Users\\pi37pat\\AppData\\Local\\Temp\\Rtmpeil5bS/ger_coords.shp"
```

Please note that our `INPUT_LAYER` is a spatial object residing in R’s
global environment. Of course, you can also use a path to specify
`INPUT_LAYER` (e.g. “ger.shp”) which is the better option if your data
is somewhere stored on your hard drive. Finally, `run_qgis()` calls the
QGIS API to run the specified geoalgorithm with the corresponding
function arguments. Since we set `load_output` to `TRUE`, `run_qgis()`
automatically loads the QGIS output back into R (an `sf`-object in the
case of vector data and a `raster`-object in the case of raster data).
Naturally, we would like to check if the result meets our expectations.

``` r
# first, plot the federal states of Germany
plot(ger)
# next plot the centroids created by QGIS
plot(out$geometry, pch = 21, add = TRUE, bg = "lightblue", col = "black")
```

<p align="center">

<img src="https://raw.githubusercontent.com/jannes-m/RQGIS/master/https://raw.githubusercontent.com/jannes-m/RQGIS/master/figures/10_plot_ger.png" width="60%"/>

</p>

Of course, this is a very simple example. We could have achieved the
same using `sp::coordinates()`. For a more detailed introduction to
**RQGIS** and more complex examples have a look at our
paper:

<div style="text-align:center">

<a href = "https://rjournal.github.io/archive/2017/RJ-2017-067/RJ-2017-067.pdf">https://rjournal.github.io/archive/2017/RJ-2017-067/RJ-2017-067.pdf</a>

</div>

# Advanced topics

## Calling the QGIS Python API

Internally, `open_app()` first sets all necessary paths (among others
the path to the QGIS Python binary) to run QGIS, and secondly opens a
QGIS application with the help of
[reticulate](https://github.com/rstudio/reticulate). Please note that
`open_app()` establishes a tunnel to the QGIS Pyton API which can only
be closed by starting a new R session (see
<https://github.com/rstudio/reticulate/issues/27>). On the one hand,
this means that we only have to set up the Python environment once and
consequently subsequent processing is faster. Additionally, you can use
your own Python commands to customize **RQGIS** in accordance with your
needs. On the other hand, it also means that once you have run the first
**RQGIS** command interacting with the QGIS Python API (e.g.,
`find_algorithms()`, `get_usage()`, etc.), you have to stay with the
chosen QGIS version for this session. For instance, if you are using
QGIS 2.14.14 (`qgis_session_info()`), and you would like to use the
developer version QGIS 2.18.7, you have to restart R. Then you can call
`set_env(dev = TRUE)` and `open_app()` to use the QGIS developer
version.

<!--
## (R)QGIS modifications (v. 2.16-2.18.1)

If you would like to use QGIS versions 2.16-2.18.1, you need to fix manually a Processing error in order to make **RQGIS** work.
First, add one `import` statement (SilentProgress) to `../processing/gui/AlgorithmExecutor.py`. 
Secondly replace `python alg.execute(progress)` by `python alg.execute(progress or SilentProgress())`:

<p align="center">
  <img src="https://raw.githubusercontent.com/jannes-m/RQGIS/master/figures/rewrite_algexecutor.PNG" width="80%"/>
</p>

The QGIS core team fixed this bug, and starting with QGIS 2.18.2 this manual adjustment is no longer necessary (see also this [post](http://gis.stackexchange.com/questions/204321/qgis-2-16-processing-runalg-fails-when-run-outside-of-qgis-in-a-custom-applicat)). 
However, we would strongly recommend to either use the QGIS LTR (2.14) or QGIS >= 2.18.2.
-->
