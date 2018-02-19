Building a OPTA ready Pitch in R with ggplot2
================

This post is a R conversion of a previous [FC Python's article](https://fcpython.com/tag/radar-chart), follow them on [twitter](www.twitter.com/FC_Python) if you have a desire to learn Python then they are a fantastic resource!

For this activity we are going to use the pitch plot function from the last tutorial. We could copy and paste the code into this file, this would work but our new script would be a bit messy. A better way to achieve this is to put the function script in a different file and the load it.

I have saved the function in a file called PitchCreate.r and stored it in my working directory. It's easy to load:

``` r
source("PitchCreate.R")
options(warn=-1) # this turns warnings off, don't get into a habit of doing this
require(ggplot2)
```

    ## Loading required package: ggplot2

Now it's loaded lets feed the function some variables to get the base of our plot:

``` r
grass_colour <- "#202020"
line_colour <- "#797876"
background_colour <- "#202020"
goal_colour <- "#131313"
jp_colour <- "#F76065"
jp_include <- TRUE
ymax <- 7040
xmax <- 10600
jp_alpha <- 0.1

pitchBase <- createPitchJP(xmax, ymax, grass_colour, line_colour, background_colour, goal_colour, jp_colour, jp_include, jp_alpha)

pitchBase
```

![](MappingTouches_files/figure-markdown_github/unnamed-chunk-2-1.png)

Our base pitch map is ready! We are going to map the locations of every time Mesut Ozil recevies the ball. We will do this by producing some fake data as we don't have access to OPTA data. Let's say he recevied the ball 50 times in a match, we use the knowledge we developed in our random number tutorial:

``` r
## let's create a function to generate some fake locations 
x <- runif(50,0,100)
y <- runif(50,0,100)

# now we have the OPTA style x,y which is displayed as a % of the x and a % of the y of a pitch. But we want to convert these to use on our pitch.
x2 <- (x/100) * xmax
y2 <- (y/100) * ymax
```

Now let's plot the touches using geom\_point(). We need to feed it the x2 and y2 data and then tell it which colour the dots should be:

``` r
pitchBase + geom_point(aes(x=x2,y=y2),colour = "blue")
```

![](MappingTouches_files/figure-markdown_github/unnamed-chunk-4-1.png)

Ozil is attacking from left to right. Let's try and make the fake data more realistic for his position. We do this by generating a random number centered around a different mean and adjusting the standard deviation.

First we setup a function that takes the variables: n - number of numbers you want to generate mean - the mean of the probabilty distribution sd - standard deviation of the sample

Below we create 50 numbers for our x values, we put a mean of 70 which means our locations are more likely to be in the opposition half. Our y mean at 60 to give it a slight bias to being on the left of the pitch. The standard deviation we set at 25.

Let's see the results

``` r
## 

give_a_bias_number <- function(n,mean,sd) { mean+sd*scale(rnorm(n)) }

x <- give_a_bias_number(50,70,25)
y <- give_a_bias_number(50,60,25)

# now we have the OPTA style x,y which is displayed as a % of the x and a % of the y of a pitch. But we want to convert these to use on our pitch.
x2 <- (x/100) * xmax
y2 <- (y/100) * ymax

pitchBase + geom_point(aes(x=x2,y=y2),colour = "blue")
```

![](MappingTouches_files/figure-markdown_github/unnamed-chunk-5-1.png)

This is much more realistic! Try playing with the variables of our give\_a\_bias\_number() function. Play with the variables to try and create realistic receiving coorindates for other positions.

``` r
give_a_bias_number <- function(n,mean,sd) { mean+sd*scale(rnorm(n)) }

x <- give_a_bias_number(50,35,30)
y <- give_a_bias_number(50,20,10)

# now we have the OPTA style x,y which is displayed as a % of the x and a % of the y of a pitch. But we want to convert these to use on our pitch.
x2 <- (x/100) * xmax
y2 <- (y/100) * ymax

pitchBase + geom_point(aes(x=x2,y=y2),colour = "blue")
```

![](MappingTouches_files/figure-markdown_github/unnamed-chunk-6-1.png)
