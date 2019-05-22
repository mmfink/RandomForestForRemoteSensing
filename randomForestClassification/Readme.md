randomForest Classification
================

## Table of Contents

  - [Project Summary](#project-summary)
  - [Source History](#source-history)
  - [To Use](#to-use)
      - [Environment](#environment)
      - [Required inputs](#required-inputs)
      - [Setting variables and output
        options](#setting-variables-and-output-options)
      - [Training data assessment](#training-data-assessment)
      - [Classification outputs](#classification-outputs)
  - [Disclaimer](#disclaimer)

## Project Summary

[Top](#table-of-contents)  
This is a tool (R script) to do image classification such as land cover
classification using a random forests classifier. This is a work in
progress and the intent is to provide robust methods that can be used by
people with minimal remote sensing experience.

The script reads an ESRI shapefile with training polygons and then
either selects all pixels or randomly selects a user-determined number
of samples for each “land cover type”. A multilayer image that contains
spectral, other continuous data, or categorical data is used as the
“land cover”. For each randomly selected sample the data values for
that pixel are determined and these data are used to run the random
forest model.

The model is then applied to all pixels in the multilayer image, and
outputs two GeoTIFF rasters:

  - **classImage:** classifies all of the pixels according to the
    classes assigned during training.
  - **probImage:** outputs the class probability of the class that got
    the most votes (i.e., the class that was selected for the classImage
    layer).

A *variable importance plot* is displayed to provide information about
the influence of each variable. An *error rate estimate* and *confusion
matrix* are also printed to provide information about classification
accuracy. A point shapefile containing the *margin* of error (the
proportion of votes for the correct class minus maximum proportion of
votes for the other classes for that area) can be optionally created to
help with iterative adjustment of the training data. Finally, there is
also an option to output a *feature space plot* using two bands of your
choice to help locate pixels in the image that are not well represented
in the training data.

## Source History

[Top](#table-of-contents)  
This is a fork of Ned Horning’s [randomForest
Classification](https://bitbucket.org/rsbiodiv/randomforestclassification/src)
repository. The original script and instructions for use were written by
Ned Horning \[<horning@amnh.org>\], of the American Museum of Natural
History, Center for Biodiversity and Conservation. Original support for
writing and maintaining this script came from The John D. and Catherine
T. MacArthur Foundation and Google.org. The original code and guide can
be found in the **/Original\_work** folder of this repository, and also
at its original location in the link given above.

Modifications to the original work are by Michelle M. Fink, Colorado
Natural Heritage Program, Colorado State University
\[<michelle.fink@colostate.edu>\]. Contributions for further
improvements are welcome and all contributors will be credited.

The original work by Ned Horning was licensed under the GNU General
Public License version 2 (GPL2,
<https://www.gnu.org/licenses/old-licenses/gpl-2.0.html>). It is being
redistributed with modifications under GPL3
(<https://www.gnu.org/licenses/gpl.html>), as allowed by the original
license. For a list of modifications, see the git commit history. See
the [License](LICENSE) file in this repository for full text of the GPL3
license.

Parts of this Readme file are based on the original user guide, which is
licensed under a [Creative Commons Attribution-Share Alike 3.0
License](https://creativecommons.org/licenses/by-sa/3.0/us/) (CC BY-SA
3.0 US). This Readme is therefore likewise licensed under CC BY-SA 3.0
US. You are free to alter the work, copy, distribute, and transmit the
document under the following conditions:

  - You must attribute the work in the manner specified by the author or
    licensor (but not in any way that suggests that they endorse you or
    your use of the work).
  - If you alter, transform, or build upon this work, you may distribute
    the resulting work only under the same, similar or a compatible
    license.

The citation for the original user guide is: Horning, N. 2013. Training
Guide for Using Random Forests to Classify Satellite Images - v9.
American Museum of Natural History, Center for Biodiversity and
Conservation. Available from <http://biodiversityinformatics.amnh.org/>.

## To Use

### Environment

[Top](#table-of-contents)  
The script was built on R 3.4.4 using the following required packages:

  - sf 0.7.3
  - raster 2.8.19
  - rgdal 1.4.3
  - randomForest 4.6.14
  - doSNOW 1.0.16

As of 2019-05-22 the package sp 1.3.1 is also required, though the
intention is to transition entirely to sf.

### Required inputs

[Top](#table-of-contents)

  - Training polygons, in
    [shapefile](https://en.wikipedia.org/wiki/Shapefile) format.
  - Multi-band raster of inputs to classify, in
    [GeoTIFF](https://en.wikipedia.org/wiki/GeoTIFF) format.

These spatial layers must be in the same geospatial coordinate system.
The training polygons are used to tell the program what to look for when
classyfing the multi-band raster into discrete “land cover” types. I use
“land cover” in quotes, because the multi-band raster can contain any
manner of continuous or categorical data. These can include satellite
spectral imagery, aerial photography, climate models, geology or soil
types, or anything else that can help inform your intended
classification goals, whether or not you are trying to identify actual
land cover or something else entirely. But for simplicity, the term
“land cover” will be used here.

Draw the training polygons around one or more examples of each known
“land cover” type. The random forest algorithm is non-parametric so it
is not necessary to keep training areas homogeneous. For example, you
can have a “cloud and shadow” class with both clouds and shadows in it.
You can have as many polygons that you want for any class. Identify what
class each polygon belongs to with an integer *class type*. Any other
field in the attribute table will be ignored by the program, but for
your own use, it is helpful to have a descriptive text field identifying
the classes. For example:

| id | class |    cover    |
| :-: | :---: | :---------: |
| 0  |   3   |    Water    |
| 1  |   3   |    Water    |
| 2  |   4   |    Grass    |
| 3  |   4   |    Grass    |
| 4  |   4   |    Grass    |
| 5  |   1   | Tall shrub  |
| 6  |   1   | Tall shrub  |
| 7  |   2   | Short shrub |

Here, “id” is the polygon identifier, “class” is the required integer
class type, and “cover” is a description of the class type. You must
have at least two class types defined in the training shapefile, even if
they just represent “my class” and “not my class”.

### Setting variables and output options

[Top](#table-of-contents)  
The variables that need to be set within the script, under the **“SET
VARIABLES HERE”** section near the top, are detailed below.

#### setwd - Working Directory

**Required** The path where the input layers are and where the outputs
will be written. If the input layers are not actually in this directory,
you will need to specify the full path to them in the variables below.
If you are running the script on Microsoft Windows, keep in mind that R
uses forward slashes for paths.

``` r
# example
setwd("C:/GIS_Projects/My_project")
```

#### shapefile - Training polygons

**Required** Include the .shp extension in the name.

``` r
# example
shapefile <- "training_1.shp"

# or, if not in the working directory
shapefile <- "C:/inputs/training_1.shp"
```

#### classNums - Class Types

**Required** The class types identified in the training shapefile that
you want to select training samples from. This must be specified as an R
vector.

``` r
# example
classNums <- c(1, 2, 3)
```

#### classSampNums - Sample selection

**Required** Training points are selected within the provided training
polygons. You have the option to either use all pixels covered by the
polygons or to randomly select a user-defined number of samples under
the polygons. One thing to keep in mind when deciding which option to
use is that the more samples you use the longer the script will take to
run.

The number of classSampNums *must* match the number of classNums, and
the order in which they are given will be matched to the order that the
classNums were written.

``` r
# example
classSampNums <- c(500, 500, 300) #classes 1 & 2 have 500 samples, class 3 has 300

# or if you want the entire training polygon(s) to be used
classSampNums <- c(0, 0, 0)
```

#### attName - Class attribute name

**Required** The name of the field in the training shapefile attribute
table that contains the integer class type.

``` r
# example
attName <- "class"
```

#### nd - NoData value

**Optional** If the input multi-band raster contains numeric values that
should be treated as NoData, specify what that value is here, so that
the script can filter those pixels out. This assumes that each band has
the same NoData value. If the raster does *not* contain values to be
filtered out, the nd variable still needs some numeric value, just make
sure it is a value that does not occur in the raster. Actual Null values
in the raster will be filtered out regardless.

``` r
# example
nd <- -99999
```

#### inImageName - Multi-band raster

**Required** Include the extension in the name, and the path if it is
different from setwd. Note that GeoTiff is known to work and is
recommended. Other raster file types may work, but this has not been
tested nor will it be supported.

``` r
# example
inImageName <-"environment_5band.tif"

# or, if not in the working directory
inImageName <- "C:/inputs/environment_5band.tif"
```

#### outMarginFile - Margin of error

**Optional** Name and path for the output margin shapefile (more
explanation in the next section). If this output is not needed, enter an
empty string. Because this file is useful during iterative assessment
and refinement of the classification, you are not allowed to (presumably
accidentally) overwrite an existing file of the same name. So make sure
you change the file name each time you run the script.

``` r
# example
outMarginFile <- "C:/outputs/margin_1.shp"

# or, if you do not want this output
outMarginFile <- ""
```

#### xBand and yBand - Feature space plot

**Optional** The band numbers to use for the X and Y axes of a
2-dimensional feature space plot (more explanation in the next section).
The first band in a multi-band raster is 1 (i.e., not zero indexed). If
you do not want to use this feature, assign 0 (zero) to both variables.

``` r
# example, to create a plot using bands 3 and 4
xBand <- 3
yBand <- 4

# or, if you do not want a feature space plot
xBand <- 0
yBand <- 0
```

### Training data assessment

[Top](#table-of-contents)  
The option to create a *feature space plot* is to visualize how your
training data are distributed across feature space. The tools allows you
to select two bands for the feature space plot axes. After the plot is
displayed a dialog is printed in the R console that gives you the option
to use other bands, define a rectangle to locate gaps in feature space,
cancel the script, or continue on to the random forest model creation.
If the option to define a rectangle is selected then you need to click
on the feature space plot to define the upper left and lower right
corners of a rectangle that falls within a gap in the training data
plotted on the feature space plot. After the rectangle is selected your
image will be plotted using the first three bands and all pixels that
were selected by the rectangle will be displayed in white. After the
plot is displayed the script will stop so you can add more training data
to cover the highlighted pixels.

The option to assess training data quality using a metric called the
*margin* can be used to evaluate how well the classifiction performed.
The margin of a training sample is the *proportion of votes that sample
got for the correct class minus maximum proportion of votes for the
other classes for that sample*. Margin values range from 1 to -1. If
margin is zero, it means your training class, and another class received
equal votes for that sample. If it is positive, your training class was
voted higher than another class which is what you want. If the margin is
negative, your training class received fewer votes than another class.

The margin data are written to a point shapefile so they can be overlaid
on the image and training polygons to assess which polygons may need to
be removed, relabeled, or modified to improve the outcome. It can also
help determine which classes may need additional training points.

### Classification outputs

[Top](#table-of-contents)  
*In Work*

## Disclaimer

[Top](#table-of-contents)  
The scripts in this repository are free software: you can redistribute
and/or modify them under the terms of the GNU General Public License as
published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.

The scripts are distributed in the hope that they will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General
Public [License](LICENSE) file for more details.
