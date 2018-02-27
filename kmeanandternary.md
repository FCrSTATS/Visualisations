Ternary Plots and K-means Clustering
================

If a player gets possession of the ball there are only three onward outcomes, she passess, she shoots or she dribbles. We can present these as decision %'s ... i.e. She passes 65% of the time, whilst shooting 23% of the time and then dribbles 12% of the time.

Let's create some fake data for a league of players using the knowledge we learnt [here](https://github.com/FCrSTATS/R_basics/blob/master/5.WhileLoops.md) and [here](https://github.com/FCrSTATS/R_basics/blob/master/9.RandomExpectedGoals.md).

``` r
## Create an empty dataframe that we use to 'catch' the results from our while loop. 
Results <- data.frame(Club = character(), Player = character(), Pass = numeric(), Dribble = numeric(), Shoot = numeric(), stringsAsFactors = F)

## We define a few varibles here which will allow us to monitor our while loop as well as create club and player IDs
noPlayers <- 500 
noClubs <- 25 
PlayerCounter <- 1
ClubCounter <- 1
PlayersInTeamCounter <- 1

## Run a while loop for the number of players in our league
while(PlayerCounter <= noPlayers){

  Player <- paste0("Player",PlayerCounter) # create a playerID
  Club <- paste0("Club",ClubCounter) # create a club ID
  x <- runif(1,0,100) # create a random number
  y <- runif(1,0,(100-x)) # create another random number witbin the remaining range (100 - x)
  z <- 100 - (x+y) # calculate last number which is simply 100 - the first two numbers  

  # create a dataframe for this one player
  temp <- data.frame(Club = Club, Player = Player, Pass = x, Dribble = y, Shoot = z, stringsAsFactors = F)
  
  # bind the player results to the overall league results 
  Results <- rbind(Results, temp)
  
  # Add 1 to the player counter as we have processed another player
  PlayerCounter <- PlayerCounter + 1
  
  # Add 1 to the players in the team counter
  PlayersInTeamCounter <- PlayersInTeamCounter + 1
  
  # Work out it the team contains 25 players
  UpTeam <- if((PlayersInTeamCounter / noClubs)%%1==0){TRUE}else{FALSE}
  
  # Up date the players in the team counter 
  PlayersInTeamCounter <- if((PlayersInTeamCounter / noClubs)%%1==0){1}else{PlayersInTeamCounter}
  
  # If the players were more than 25 then update the club counter 
  ClubCounter <- if(UpTeam){ClubCounter + 1}else{ClubCounter}
}

## Print the top 5 rows to see if everything worked ok
head(Results)
```

    ##    Club  Player      Pass   Dribble      Shoot
    ## 1 Club1 Player1 83.989731 10.404360  5.6059098
    ## 2 Club1 Player2 72.140048 27.267614  0.5923382
    ## 3 Club1 Player3  8.271803 82.067260  9.6609370
    ## 4 Club1 Player4 96.656463  1.475396  1.8681409
    ## 5 Club1 Player5 47.617233 19.929625 32.4531419
    ## 6 Club1 Player6 34.791456 49.700572 15.5079716

But how do we plot this concisely? If we use two axis, i.e. Pass and Shoot, we will struggle to show the Dribble information. What we really want to do is to plot our data on three axis... enter Ternary Plots.

Ternary Plots allow us to plot information on three axis with ease, they take a little time to get use to reading but once you get your eye in they are very useful.

With many things that you want to achieve in R, there is often a package. ggtern provides us with a massive helping hand, make sure it's installed and let's plot our data.

``` r
require(ggtern)
```

    ## Loading required package: ggtern

    ## Loading required package: ggplot2

    ## --
    ## Consider donating at: http://ggtern.com
    ## Even small amounts (say $10-50) are very much appreciated!
    ## Remember to cite, run citation(package = 'ggtern') for further info.
    ## --

    ## 
    ## Attaching package: 'ggtern'

    ## The following objects are masked from 'package:ggplot2':
    ## 
    ##     %+%, aes, annotate, calc_element, ggplot, ggplot_build,
    ##     ggplot_gtable, ggplotGrob, ggsave, layer_data, theme,
    ##     theme_bw, theme_classic, theme_dark, theme_gray, theme_light,
    ##     theme_linedraw, theme_minimal, theme_void

``` r
#Create the plot and store
plot <- ggtern(data = Results, aes(x = Pass, y = Dribble, z = Shoot)) + 
               geom_point(size = 2, 
                          shape = 21, 
                          color = "black",
                          fill="black")
 
#Render
plot
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/ternary1.png?raw=true)


Fantastic, we have each of our players plotted and this provides us an overview of the players in our fake league. Maybe, we want to see how one specific player lays within the dataset... let's see where Player12 sits.

``` r
PlayerSelect <- "Player27"
playerHightlight <- Results[which(Results$Player == PlayerSelect),]

require(ggtern)
#Create the plot and store
plot <- ggtern(data = Results, aes(x = Pass, y = Dribble, z = Shoot)) + 
               geom_point(size = 2, 
                          shape = 21, 
                          color = "black",
                          fill= "black") + 
              geom_point(data = playerHightlight, aes(x = Pass, y = Dribble, z = Shoot), 
                          size = 2, 
                          shape = 21, 
                          color = "black",
                          fill= "red")
```

    ## Warning: Ignoring unknown aesthetics: z

``` r
#Render
plot
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/ternary2.png?raw=true)

This is a little difficult to see so let's make the other datapoints grey and not black.

``` r
#Create the plot and store
plot <- ggtern(data = Results, aes(x = Pass, y = Dribble, z = Shoot)) + 
               geom_point(size = 2, 
                          shape = 21, 
                          color = "grey",
                          fill= "grey") + 
              geom_point(data = playerHightlight, aes(x = Pass, y = Dribble, z = Shoot), 
                          size = 2, 
                          shape = 21, 
                          color = "black",
                          fill= "red")
```

    ## Warning: Ignoring unknown aesthetics: z

``` r
#Render
plot
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/ternary3.png?raw=true)

This is much better and it's easy to see how the player fits within the overal dataset. Maybe we would want to see how a full team of players fit within the dataset.

``` r
ClubSelect <- "Club5"
ClubHightlight <- Results[which(Results$Club == ClubSelect),]

require(ggtern)
#Create the plot and store
plot <- ggtern(data = Results, aes(x = Pass, y = Dribble, z = Shoot)) + 
               geom_point(size = 2, 
                          shape = 21, 
                          color = "grey",
                          fill= "grey") + 
              geom_point(data = ClubHightlight, aes(x = Pass, y = Dribble, z = Shoot), 
                          size = 2, 
                          shape = 21, 
                          color = "black",
                          fill= "red")
```

    ## Warning: Ignoring unknown aesthetics: z

``` r
#Render
plot
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/ternary4.png?raw=true)

Fantastic, I think this works pretty well, have a play around with the code and further feautures of the ggtern package and see if you can make it better.

K-Means Clustering
------------------

K-means Clustering is a way of dividing datapoints into groups or 'clusters' depending on how 'similar' they. A quick google will give you much more details, but let's see it in action with out fake data.

``` r
# create a new dataframe of just the passing, shooting and dribbling data. 
ResultsKmean <- Results[3:5]

# use the kmeans() function to perform the analysis, we chose to devide the data into 6 'clusters' 
kmeans <- kmeans(ResultsKmean, 6)

# We want to bind the kMeans Clusters to the original dataframe 
Results$Cluster <- as.character(kmeans$cluster)

## print the top 5 rows to check we are on the right path 
head(Results)
```

    ##    Club  Player      Pass   Dribble      Shoot Cluster
    ## 1 Club1 Player1 83.989731 10.404360  5.6059098       1
    ## 2 Club1 Player2 72.140048 27.267614  0.5923382       6
    ## 3 Club1 Player3  8.271803 82.067260  9.6609370       4
    ## 4 Club1 Player4 96.656463  1.475396  1.8681409       1
    ## 5 Club1 Player5 47.617233 19.929625 32.4531419       2
    ## 6 Club1 Player6 34.791456 49.700572 15.5079716       3

Great now we have all the data and the clusters of our players. Let's plot the full dataset again but colour it by cluster.

``` r
#Create the plot and store
plot <- ggtern(data = Results, aes(x = Pass, y = Dribble, z = Shoot)) + 
               geom_point(aes(fill = Cluster), size = 2, 
                          shape = 21, 
                          color = "black")
 
#Render
plot
```

![](https://github.com/FCrSTATS/Visualisations/blob/master/Images/ternary5.png?raw=true)

We can see that players within Cluster 3 have a high tendancy to Pass with over 80& of their actions being passes. Maybe we are about to sell our best midfielder who dictates and maintains our possessions. If we were looking across the league for a replacement we could focus our efforts on players that also reside in cluster 3.

Granted we are using fake data but this article may spark a thought about how you could use ternary plots and/or k-means clustering with other data.
