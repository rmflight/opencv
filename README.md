# Vision and Graphics

> Some experiments combining vision and graphics.

[![Build Status](https://travis-ci.org/ropensci/opencv.svg?branch=master)](https://travis-ci.org/ropensci/opencv)
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/ropensci/opencv?branch=master&svg=true)](https://ci.appveyor.com/project/jeroen/opencv)


## Installation

On MacOS or Linux, first install opencv (not needed on Windows):

```sh
brew install opencv
```

On Ubuntu or Fedora you need [`libopencv-dev`](https://packages.debian.org/testing/libopencv-dev) or [`opencv-devel`](https://apps.fedoraproject.org/packages/opencv-devel/):

```sh
sudo apt-get install libopencv-dev
```

And then install the R bindings:

```r
devtools::install_github("ropensci/opencv")
library(opencv)
```

## Basic stuff:

Face recognition:

```r
unconf <- ocv_read('https://jeroen.github.io/images/unconf18.jpg')
faces <- ocv_face(unconf)
ocv_write(faces, 'faces.jpg')
```

Or get the face location data:

```r
facemask <- ocv_facemask(unconf)
attr(facemask, 'faces')
```

## Live Webcam Examples

Live face detection:

```r
library(opencv)
ocv_video(ocv_face)
```

Edge detection:

```r
library(opencv)
ocv_video(ocv_edges)
```

## Combine with Graphics

Replaces the background with a plot:

```r
library(opencv)
library(ggplot2)

# get webcam size
test <- ocv_camera()
bitmap <- ocv_bitmap(test)
width <- dim(bitmap)[2]
height <- dim(bitmap)[3]

png('bg.png', width = width, height = height)
par(ask=FALSE)
print(ggplot2::qplot(speed, dist, data = cars, geom = c("smooth", "point")))
dev.off()
bg <- ocv_read('bg.png')
unlink('pg.png')
ocv_video(function(input){
  mask <- ocv_mog2(input)
  return(ocv_copyto(input, bg, mask))
})
```

Put your face in the plot:

```r
# Overlay face filter
ocv_video(function(input){
  mask <- ocv_facemask(input)
  ocv_copyto(input, bg, mask)
})
```

## Live Face Survey

Go stand on the left if you're a tidier

```r
library(opencv)

# get webcam size
test <- ocv_camera()
bitmap <- ocv_bitmap(test)
width <- dim(bitmap)[2]
height <- dim(bitmap)[3]

# generates the plot
makeplot <- function(x){
  png('bg.png', width = width, height = height, res = 96)
  on.exit(unlink('bg.png'))
  groups <- seq(0, width, length.out = 4)
  left <- rep("left", sum(x < groups[2]))
  middle <- rep("middle", sum(x >= groups[2] & x < groups[3]))
  right <- rep("right", sum(x >= groups[3]))
  f <- factor(c(left, middle, right), levels = c('left', 'middle', 'right'),
              labels = c("Tidy!", "Whatever Works", "Base!"))
  color = I(c("#F1BB7B", "#FD6467", "#5B1A18"))
  plot(f, ylim = c(0, 5),
       main = "Are you a tidyer or baser?", col = color)
  dev.off()
  ocv_read('bg.png')
}

# overlays faces on the plot
ocv_video(function(input){
  mask <- ocv_facemask(input)
  faces <- attr(mask, 'faces')
  bg <- makeplot(faces$x)
  return(ocv_copyto(input, bg, mask))
})
```
