Pressure Profiles
================

Women's Football Data
---------------------
[FC Python](https://fcpython.com/tag/radar-chart)
Earlier this month [Statsbomb](https://statsbomb.com/) announced their [data product](https://statsbomb.com/data/) which looks to improve the current data offerings on the market. Interestingly, Statsbomb will be collecting data on women's football. The other day [Ted Knutson](https://twitter.com/mixedknuts) [tweeted](https://twitter.com/mixedknuts/status/997188477434425344):

"\#WhatIf we collect data on top flight women's football, on the same spec as the men, and give it away to clubs and fans alike, for free? Can we better support and help create the next generation of women's football coaches, player, writers, and fans?"

The [tweet](https://twitter.com/mixedknuts/status/997188477434425344) got a lot of traction and we hope Statsbomb follow through with the idea. To do my part I thought I would use the data distributed at their launch event in this tutorial.

An Idea
-------

I love myself a histogram as they are accessible ways of seeing patterns within the data. [Mara Averick](https://twitter.com/dataandme) [tweeted](https://twitter.com/dataandme/status/997297983472402437) about [Simon Jackson's](https://twitter.com/drsimonj) [work](https://drsimonj.svbtle.com/plotting-background-data-for-groups-with-ggplot2?utm_content=buffer2b686&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer) on histograms and how they can be layered to great effect.

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/example.jpg?raw=true)

I was searching for that could utilise this technique to good effect. I was intrigued to use the pressure metrics that are new to Statsbomb but I just couldn't wrestle an idea together in the time slot I had to make this tutorial. I therefore settled on our old friend passes...

After bouncing some ideas around I settled on showing the frequency of vertical origins of passes, this will allow us to see patterns at team level as well as overlay individual player stats.

Setting up the Packages
-----------------------

We will be using a few packages today so lets load them (make sure to install them if you haven't already).

``` r
## load packages 
require(dplyr) ## package for data tidying
require(formattable) ## package to make tables look nice
require(ggplot2) ## package for plotting
require(patchwork) ## package for organising our plots
require(plyr) ## for help with summaries
```

Loading & Prepping the Data
---------------------------

First we load the csv file that Statsbomb included in their free data drop at the event.

``` r
df <- read.csv("Files/Manchester City WFC_Chelsea LFC_7298.csv", stringsAsFactors = F)
```

Next we store a list of the participlated teams which we will use as variables to filter data later.

``` r
Teams <- unique(df$team_name)
```

Then we create a dataset for Team 1 (which in this case is Manchester City Women's FC) and filtering out everything but passes and filter out goal-kicks.

``` r
df_pass_T1 <- df %>% filter(event_type_name == "Pass", team_name == Teams[1], play_pattern_name != "From Goal Kick")
```

Seperately, I want to make a summary table of the top 3 passers for MCWFC which I will use as part of my player selection method.

``` r
PassTotals <- count(df_pass_T1, 'player_name')
PassTotals <- PassTotals %>% arrange(-freq) %>% head(3)
formattable(PassTotals)
```

<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
player\_name
</th>
<th style="text-align:right;">
freq
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
Demi Stokes
</td>
<td style="text-align:right;">
66
</td>
</tr>
<tr>
<td style="text-align:right;">
Esme Beth Morgan
</td>
<td style="text-align:right;">
54
</td>
</tr>
<tr>
<td style="text-align:right;">
Jennifer Beattie
</td>
<td style="text-align:right;">
53
</td>
</tr>
</tbody>
</table>
So Demi Stokes is the most frequent passer for MCWFC in this match against Chelsea Ladies FC, it will be Demi that we plot later.

Plotting the Team's Histogram
-----------------------------

Firstly, let's plot the histogram for the whole team, get the styling right and then overlay Demi's histogram at the end.

We will use ggplot2 package to help us with this, and we will build the plot layer by layer (using similar techniques that I used previously [here](https://github.com/FCrSTATS/Visualisations/blob/master/3.CreateAPitch.md) and [here](https://github.com/FCrSTATS/Visualisations/blob/master/2.BuildingARadar.md)).

``` r
## first we build the base plot, confirm the dataframe we will be using and the metric we will plot (location_x) which is vertical origin of the pass.  
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x))
```
![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown.png?raw=true)

Great, let's add the histogram via geom_histogram(). I set the binwidth to 5 and reduce the alpha (opacity) of the plot to 70% or 0.7, lastly I choose a faded pink for the fill.

``` r
## 
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
  geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F")
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-2.png?raw=true)

This provides us with a great base to work from, we know the data is plotting fine and all we need to do now is to get control of how the plot looks.

Eventually I want to be able to automatically generate the plots for a new game, this means we must lock the x and y axis so they are better used for comparison. We use ylim() and xlim() to achieve this.

``` r
## 
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125))
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-3.png?raw=true)

Although the plot hasn't changed much, the last step will really help us out later.

Next lets add a title to the plot and use theme\_void() to make it much more minimalistic.

``` r
## 
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
ggtitle(paste0(Teams[1],": Vertical Origin of Passes (x)")) +
theme_void()
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-4.png?raw=true)

Perfect, this is much cleaner and I am pretty happy with this as the base plot. Now let's add Demi Stokes data over the top.

First we make a subset of data and then simply re-use the geom\_histogram() function with a small change of the colour and the data which is used.

``` r
## create the player dataframe 
df_player_selectT1 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[1])

##
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
ggtitle(paste0(Teams[1],": Vertical Origin of Passes (x)")) +
theme_void() + 
geom_histogram(data = df_player_selectT1, aes(df_player_selectT1$location_x), binwidth = 5, alpha = 1, fill = "#E24F55")
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-5.png?raw=true)

Fantastic, we have quickly created a plot that we can use to see patterns of passing origins at team level whilst comparing player's individual contributions. If we didn't have to show the plot to anyone else we could make some conclusions, however a reader without any prior knowledge would not have a clue whats going on!

We need to improve readabilty of the plot.

First let's make sure that the reader knows which direction attacking play is, we acheive this by a simple arrow and text labels

``` r
## create the player dataframe 
df_player_selectT1 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[1])

##
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
ggtitle(paste0(Teams[1],": Vertical Origin of Passes (x)")) +
theme_void() + 
geom_histogram(data = df_player_selectT1, aes(df_player_selectT1$location_x), binwidth = 5, alpha = 1, fill = "#E24F55") + 
geom_segment(aes(x = -2.5, y = -2, xend = 25, yend = -2),colour = "#435366", arrow = arrow(length = unit(0.1, "cm"), type="closed")) + 
annotate("text", x = -2.5, y = -1, label = "Attacking Direction", colour = "#435366", fontface=2, size = 3, hjust = 0)
```
![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-5.png?raw=true)

Next we can add some information on the y-axis to provide frequency context.

``` r
## create the player dataframe 
df_player_selectT1 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[1])

## create the dataframe that we will use to plot the y-axis ticks and text
df_ticks <- data.frame(x = -4, y = seq(10,40,10), xend = -2.5, yend=seq(10,40,10), label_text = as.character(seq(10,40,10)), stringsAsFactors = F )
df_ticks$label_text <- paste0(df_ticks$label_text, " ")

##
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
ggtitle(paste0(Teams[1],": Vertical Origin of Passes (x)")) +
theme_void() + 
geom_histogram(data = df_player_selectT1, aes(df_player_selectT1$location_x), binwidth = 5, alpha = 1, fill = "#E24F55") + 
geom_segment(aes(x = -2.5, y = -2, xend = 25, yend = -2),colour = "#435366", arrow = arrow(length = unit(0.1, "cm"), type="closed")) + 
annotate("text", x = -2.5, y = -1, label = "Attacking Direction", colour = "#435366", fontface=2, size = 3, hjust = 0) + 
geom_segment(data = df_ticks, aes(x = x, y=y, xend = xend, yend =yend), colour = "#435366") + 
geom_text(data = df_ticks, aes(x = x, y=y,label = label_text), hjust=1, vjust=0.5, size = 3)
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-6.png?raw=true)

We don't know which player the plot refers to! So let's add the name of the player and a pass count in \[\].

``` r
## create the player dataframe 
df_player_selectT1 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[1])

## create the dataframe that we will use to plot the y-axis ticks and text
df_ticks <- data.frame(x = -4, y = seq(10,40,10), xend = -2.5, yend=seq(10,40,10), label_text = as.character(seq(10,40,10)), stringsAsFactors = F )
df_ticks$label_text <- paste0(df_ticks$label_text, " ")

##
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
ggtitle(paste0(Teams[1],": Vertical Origin of Passes (x)")) +
theme_void() + 
geom_histogram(data = df_player_selectT1, aes(df_player_selectT1$location_x), binwidth = 5, alpha = 1, fill = "#E24F55") + 
geom_segment(aes(x = -2.5, y = -2, xend = 25, yend = -2),colour = "#435366", arrow = arrow(length = unit(0.1, "cm"), type="closed")) + 
annotate("text", x = -2.5, y = -1, label = "Attacking Direction", colour = "#435366", fontface=2, size = 3, hjust = 0) + 
geom_segment(data = df_ticks, aes(x = x, y=y, xend = xend, yend =yend), colour = "#435366") + 
geom_text(data = df_ticks, aes(x = x, y=y,label = label_text), hjust=1, vjust=0.5, size = 3) +  annotate("text", x = 60, y = -4, label = paste0(PassTotals$player_name[1]," [",PassTotals$freq[1],"]"), colour = "#E24F55", fontface=2, size = 6) 
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-7.png?raw=true)

This is pretty good and essentially we could leave it there but I feel it would be much more intuitive if we overlay the outline of a pitch on top of the plot.

``` r
## create the player dataframe 
df_player_selectT1 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[1])

## create the dataframe that we will use to plot the y-axis ticks and text
df_ticks <- data.frame(x = -4, y = seq(10,40,10), xend = -2.5, yend=seq(10,40,10), label_text = as.character(seq(10,40,10)), stringsAsFactors = F )
df_ticks$label_text <- paste0(df_ticks$label_text, " ")

##
ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
ggtitle(paste0(Teams[1],": Vertical Origin of Passes (x)")) +
theme_void() + 
geom_histogram(data = df_player_selectT1, aes(df_player_selectT1$location_x), binwidth = 5, alpha = 1, fill = "#E24F55") + 
geom_segment(aes(x = -2.5, y = -2, xend = 25, yend = -2),colour = "#435366", arrow = arrow(length = unit(0.1, "cm"), type="closed")) + 
annotate("text", x = -2.5, y = -1, label = "Attacking Direction", colour = "#435366", fontface=2, size = 3, hjust = 0) + 
geom_segment(data = df_ticks, aes(x = x, y=y, xend = xend, yend =yend), colour = "#435366") + 
geom_text(data = df_ticks, aes(x = x, y=y,label = label_text), hjust=1, vjust=0.5, size = 3) +  annotate("text", x = 60, y = -4, label = paste0(PassTotals$player_name[1]," [",PassTotals$freq[1],"]"), colour = "#E24F55", fontface=2, size = 6) + 
# add pitch outline
geom_rect(aes(xmin=-2.5, xmax=122.5, ymin=0, ymax=50), fill = NA, colour = "#435366") + 
# left hand 18 yard 
geom_rect(aes(xmin=-2.5, xmax=16.25, ymin=11, ymax=39), fill = NA, colour = "#435366") + 
# add right hand 18 yard 
geom_rect(aes(xmin=103.75, xmax=122.5, ymin=11, ymax=39), fill = NA, colour = "#435366") +
# add half way line
geom_segment(aes(x = 60, y = 0, xend = 60, yend = 50),colour = "#435366") 
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-8.png?raw=true)

Perfect, I am pretty happy with this! It's much better for the reader now and gives a good indication of the team patterns and individual contribution. Without watching much footage of MCWFC or Demi Stokes, I can already see that the left-back is extremely important to their offensive play.

Patchwork Plots
---------------

We could leave it here but I would quiet like to plot the top 3 passers in 3 seperate plots but then displayed side-by-side. The patchwork package is ideal for this job. Have a look at the following code to see if you can follow along.

``` r
## create the player dataframe 
df_player_selectT1P1 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[1])

## create the dataframe that we will use to plot the y-axis ticks and text
df_ticks <- data.frame(x = -4, y = seq(10,40,10), xend = -2.5, yend=seq(10,40,10), label_text = as.character(seq(10,40,10)), stringsAsFactors = F )
df_ticks$label_text <- paste0(df_ticks$label_text, " ")

## plot 1
p1 <- ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
ggtitle(paste0(Teams[1],": Vertical Origin of Passes (x)")) +
theme_void() + 
geom_histogram(data = df_player_selectT1P1, aes(df_player_selectT1P1$location_x), binwidth = 5, alpha = 1, fill = "#E24F55") + 
geom_segment(aes(x = -2.5, y = -2, xend = 25, yend = -2),colour = "#435366", arrow = arrow(length = unit(0.1, "cm"), type="closed")) + 
annotate("text", x = -2.5, y = -1, label = "Attacking Direction", colour = "#435366", fontface=2, size = 2, hjust = 0) + 
geom_segment(data = df_ticks, aes(x = x, y=y, xend = xend, yend =yend), colour = "#435366") + 
geom_text(data = df_ticks, aes(x = x, y=y,label = label_text), hjust=1, vjust=0.5, size = 3) +  annotate("text", x = 60, y = -4, label = paste0(PassTotals$player_name[1]," [",PassTotals$freq[1],"]"), colour = "#E24F55", fontface=2, size = 6) + 
# add pitch outline
geom_rect(aes(xmin=-2.5, xmax=122.5, ymin=0, ymax=50), fill = NA, colour = "#435366") + 
# left hand 18 yard 
geom_rect(aes(xmin=-2.5, xmax=16.25, ymin=11, ymax=39), fill = NA, colour = "#435366") + 
# add right hand 18 yard 
geom_rect(aes(xmin=103.75, xmax=122.5, ymin=11, ymax=39), fill = NA, colour = "#435366") +
# add half way line
geom_segment(aes(x = 60, y = 0, xend = 60, yend = 50),colour = "#435366") 
  
## create the player dataframe 
df_player_selectT1P2 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[2])

## plot 2
p2 <- ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
theme_void() + 
geom_histogram(data = df_player_selectT1P2, aes(df_player_selectT1P2$location_x), binwidth = 5, alpha = 1, fill = "#E24F55") + 
geom_segment(aes(x = -2.5, y = -2, xend = 25, yend = -2),colour = "#435366", arrow = arrow(length = unit(0.1, "cm"), type="closed")) + 
annotate("text", x = -2.5, y = -1, label = "Attacking Direction", colour = "#435366", fontface=2, size = 2, hjust = 0) + 
geom_segment(data = df_ticks, aes(x = x, y=y, xend = xend, yend =yend), colour = "#435366") + 
geom_text(data = df_ticks, aes(x = x, y=y,label = label_text), hjust=1, vjust=0.5, size = 3) +  annotate("text", x = 60, y = -4, label = paste0(PassTotals$player_name[2]," [",PassTotals$freq[2],"]"), colour = "#E24F55", fontface=2, size = 6) + 
# add pitch outline
geom_rect(aes(xmin=-2.5, xmax=122.5, ymin=0, ymax=50), fill = NA, colour = "#435366") + 
# left hand 18 yard 
geom_rect(aes(xmin=-2.5, xmax=16.25, ymin=11, ymax=39), fill = NA, colour = "#435366") + 
# add right hand 18 yard 
geom_rect(aes(xmin=103.75, xmax=122.5, ymin=11, ymax=39), fill = NA, colour = "#435366") +
# add half way line
geom_segment(aes(x = 60, y = 0, xend = 60, yend = 50),colour = "#435366") 
  
## create the player dataframe 
df_player_selectT1P3 <- df_pass_T1 %>% filter(player_name == PassTotals$player_name[3])

## plot 2
p3 <- ggplot(data=df_pass_T1, aes(df_pass_T1$location_x)) +
geom_histogram(binwidth = 5, alpha = .7, fill = "#EDA09F") + 
ylim(c(-5,50)) + 
xlim(c(-5,125)) + 
theme_void() + 
geom_histogram(data = df_player_selectT1P3, aes(df_player_selectT1P3$location_x), binwidth = 5, alpha = 1, fill = "#E24F55") + 
geom_segment(aes(x = -2.5, y = -2, xend = 25, yend = -2),colour = "#435366", arrow = arrow(length = unit(0.1, "cm"), type="closed")) + 
annotate("text", x = -2.5, y = -1, label = "Attacking Direction", colour = "#435366", fontface=2, size = 2, hjust = 0) + 
geom_segment(data = df_ticks, aes(x = x, y=y, xend = xend, yend =yend), colour = "#435366") + 
geom_text(data = df_ticks, aes(x = x, y=y,label = label_text), hjust=1, vjust=0.5, size = 3) +  annotate("text", x = 60, y = -4, label = paste0(PassTotals$player_name[3]," [",PassTotals$freq[3],"]"), colour = "#E24F55", fontface=2, size = 6) + 
# add pitch outline
geom_rect(aes(xmin=-2.5, xmax=122.5, ymin=0, ymax=50), fill = NA, colour = "#435366") + 
# left hand 18 yard 
geom_rect(aes(xmin=-2.5, xmax=16.25, ymin=11, ymax=39), fill = NA, colour = "#435366") + 
# add right hand 18 yard 
geom_rect(aes(xmin=103.75, xmax=122.5, ymin=11, ymax=39), fill = NA, colour = "#435366") +
# add half way line
geom_segment(aes(x = 60, y = 0, xend = 60, yend = 50),colour = "#435366") 

p1 | p2 | p3
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/Unknown-9.png?raw=true)
