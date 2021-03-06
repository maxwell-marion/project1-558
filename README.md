# Using the PokeAPI: Examining and creating functions for the pokemon, habitat, color, egg-group, and berry endpoints

Maxwell Marion-Spencer
2022-06-26

-   [Requirements](#requirements)
-   [Functions](#functions)
    -   [`pokemon`](#pokemon)
    -   [`pokemon_df`](#pokemon_df)
    -   [`habitat`](#habitat)
    -   [`color`](#color)
    -   [`egg_group`](#egg_group)
    -   [`berry`](#berry)
-   [Exploratory Data Analysis](#exploratory-data-analysis)
    -   [Pokémon Analysis](#pokémon-analysis)
    -   [Berry Analysis](#berry-analysis)
    -   [Conclusion](#conclusion)

# Requirements

List of required packages:

-   `tidyverse`
-   `httr`
-   `jsonlite`

This vignette will highlight some of the many features of the
[PokeAPI](https://pokeapi.co/), while also providing a few functions
that may be useful for pulling and analyzing data from this API.

Additionally, I will examine some of the trends and relationships across
Pokémon from the first two generations, as well as taking a look at the
numerous berries that can be found in the game.

# Functions

PokeAPI is a RESTful API that covers “everything from Pokémon to Berry
Flavors.” Fittingly, our functions will begin with the `pokemon`
endpoint, and conclude with the `berry` endpoint.

## `pokemon`

The `pokemon` function allows you to bring in information about a
particular Pokémon by using the `pokemon` endpoint. The user has the
option to specify whether or not they would also like to receive the
Pokémon’s statistics, which characterize how effective a Pokémon is in
battle. This `stats` option is included by default. When this option is
not specified only the height, weight, and type(s) are included.

``` r
# Building 'Pokemon' Query function
pokemon <- function(pokemon, stats = TRUE){
  
  ###
  # Pulls in height, weight, type and base stats (hp, attack, defense, 
  #  special attack, special defense, speed)
  ###
  
  # Fixing case, if last is entered as a specific pokemon
  if(is.character(pokemon == TRUE)){
     pokemon <- tolower(pokemon)
  }
  
  # Accounting for the user entering a number rather than a specific Pokémon
  if(is.numeric(pokemon) == TRUE){
    last_test <- fromJSON(paste0('https://pokeapi.co/api/v2/pokemon/',last,
                                 collapse =","))
    pokemon <- last_test$name
  }
  
  # Getting API output
  output <- fromJSON(paste0('https://pokeapi.co/api/v2/pokemon/',pokemon,
                            collapse =","))
  
  # Output if user DOES want to see the battle stat attributes of the pokemon
  if(stats == TRUE){
    
  # Loading the data into a vector
  output_row <- c(pokemon,  output$types$type$name[1], output$types$type$name[2],
                  output$height,output$weight,output$stats$base_stat[1],
                  output$stats$base_stat[2],output$stats$base_stat[3],
                  output$stats$base_stat[4],output$stats$base_stat[5],
                  output$stats$base_stat[6])
  
  # Coercing into a data frame
  output_frame <- as.data.frame(t(output_row))
  colnames(output_frame) <- c("name","type_1","type_2","height","weight","hp",
                              "attack","defense","spec_attack","spec_defense",
                              "speed")
  
  # Making the numeric variables properly numeric
  output_frame <- transform(output_frame, height = as.numeric(height),
                            weight = as.numeric(weight),hp = as.numeric(hp),
                            attack = as.numeric(attack), 
                            defense = as.numeric(defense), 
                            spec_attack = as.numeric(spec_attack),
                            spec_defense = as.numeric(spec_defense), 
                            speed = as.numeric(speed))
  
  # Returning a nice data frame
  return(output_frame)
  }
  
  # Output if user does NOT want to see the battle stat attributes of the pokemon
  if(stats == FALSE){
    
  # Loading the data into a vector
  output_row <- c(pokemon, output$types$type$name[1], output$types$type$name[2],
                  output$height, output$weight)
  
  # Coercing into a data frame
  output_frame <- as.data.frame(t(output_row))
  colnames(output_frame) <- c("name","type_1","type_2","height","weight")
  
  # Making the numeric variables properly numeric
  output_frame <- transform(output_frame, height = as.numeric(height), 
                            weight = as.numeric(weight))
  
  # Returning a nice data frame
  return(output_frame)
  }
}
```

## `pokemon_df`

Progressing logically, the `pokemon_df` function also uses the `pokemon`
endpoint and allows you to procure a data frame containing either all of
the pokemon (using the `all = TRUE` option) or simply a list from
Pokémon \#1, to the desired last Pokémon to be included.

(e.g): If you wanted to pull in from Bulbasaur (#1) to Charizard (#6),
you would use: pokemon_df(last = “Charizard”) or pokemon_df(last = 6)

The user also has the option of returning a data frame with information
from all Pokémon (up to \#251, or Gen 2) by utilizing the `all = TRUE`
argument. If desired, you could adjust the function to make the search
unlimited (to Pokémon \#1126) but this would take quite some time!

``` r
# Building a function for pokemon that allows you to pull a df
pokemon_df <- function(all = FALSE, last = NULL){
  
  ###
  # Pulls in height, weight, type and base stats (hp, attack, defense, 
  #  special attack, special defense, speed) for up to a specific pokemon, 
  #  or all pokemon. Note that 'last' can be entered as a number, or as a 
  #  specific pokemon.
  ###
  
  # Fixing case, if last is entered as a specific pokemon
  if(is.character(last) == TRUE){
    last <- tolower(last)
  }
  
  # Accounting for the user entering a number rather than a specific Pokémon
  if(is.numeric(last) == TRUE){
     last_test <- fromJSON(paste0('https://pokeapi.co/api/v2/pokemon/',
                                  last,collapse =","))
     last <- last_test$name
  }
  
  # If all = TRUE, returning a df of all pokemon from Generation 1 + 2 (to 251)
  # NOTE: This is done to limit the burden on the APi
  if(all == TRUE){
    last <- 251
  }
  
  # Establishing empty df to merge things to
  output_df <- rep("a",11)
  
  # Running for loop to populate df
  for(n in 1:last){
  
    # Getting API output
    output <- fromJSON(paste0('https://pokeapi.co/api/v2/pokemon/',n,
                              collapse =","))
    
    # Loading the data into a vector
    output_row <- c(output$name,  output$types$type$name[1], 
                    output$types$type$name[2],output$height,
                    output$weight,output$stats$base_stat[1],
                    output$stats$base_stat[2],output$stats$base_stat[3],
                    output$stats$base_stat[4],output$stats$base_stat[5],
                    output$stats$base_stat[6])
  
    # Coercing into a data frame
    output_frame <- as.data.frame(t(output_row))
    colnames(output_frame) <- c("name","type_1","type_2","height","weight",
                                "hp","attack","defense","spec_attack",
                                "spec_defense","speed")
  
  # Adding the stats to the output df
  output_df <- rbind(output_df,output_frame)
  
  }
  
  # Making the numeric variables properly numeric
  output_df <- transform(output_df, height = as.numeric(height),
                              weight = as.numeric(weight),hp = as.numeric(hp),
                              attack = as.numeric(attack), 
                              defense = as.numeric(defense), 
                              spec_attack = as.numeric(spec_attack),
                              spec_defense = as.numeric(spec_defense), 
                              speed = as.numeric(speed))
  
  # Fixing row names and removing extraneous 'starter' row
  output_df <- output_df[-1,]
  rownames(output_df) <- NULL
  
  # Returning our desired data frame
  return(output_df)
  
}
```

## `habitat`

The `habitat` function makes use of the `pokemon-habitat` endpoint of
the PokeAPI, and pulls in a list of Pokémon that occupy the specified
habitat. This function also checks the user’s input against a list of
acceptable habitats, and returns an message if the habitat the user has
specified is not appropriate for querying the API, along with a list of
appropriate habitats.

(e.g): \`habitat(“cave”) returns a data frame of cave: zubat, diglett,
(…), registeel.

``` r
habitat <- function(habitat){
  ###
  # Asks for habitat as an argument, returns a data frame of the pokemon who can
  # be found in that habitat.
  # 
  # NOTE: Habitats are: cave, forest, grassland, mountain, rare, rough-terrain,
  # sea, urban, and waters-edge.
  ###
  
  # Establishing a habitat list for input checking
  habitat_list <- c("cave", "forest", "grassland", "mountain", "rare", 
                    "rough-terrain", "sea", "urban", "waters-edge")
  
  # Implementing a check and error message
  if(!(habitat %in% habitat_list)){
    stop("Please pick one of cave, forest, grassland, mountain, rare, 
    rough-terrain, sea, urban, or waters-edge.")
  }
  
  # Building output
  output <- fromJSON(paste0('https://pokeapi.co/api/v2/pokemon-habitat/',
                            habitat,collapse =","))
  
  # Building output df and naming
  output_df <- data.frame(output$pokemon_species$name)
  colnames(output_df) <- c(habitat)
  
  # Returning results
  return(output_df)
}
```

## `color`

Similarly, the `color` function returns a data frame of all Pokémon that
match the user-specified color. This function also checks the user’s
input against a list of acceptable colors, and returns an message if the
color the user has specified is not appropriate for querying the API,
along with a list of appropriate colors. One small quality of life
addition is if the user enters the color ‘grey’ it is adjusted to
‘gray.’

(e.g): color(‘yellow’) returns a data frame of yellow: sandshrew,
meowth, (…), cofagrigus.

``` r
color <- function(color){
  ###
  # Asks for color as an argument, returns a data frame of all pokemon of that
  # color.
  #
  # NOTE: Colors are - black, blue, brown, gray, green, pink, purple, red, white
  # and yellow.
  ###
  
  # Establishing a color list for input checking
  color_list <- c("black","blue","brown","gray","grey","green","pink", 
                  "purple","red","white","yellow")
  
  # Implementing a check and error message
  if(!(color %in% color_list)){
    stop("Please pick one of black, blue, brown, gray, green, pink, purple, red,
    white, or yellow ")
  }
  
  # Troubleshooting grey != gray
  if(color == 'grey'){
    color <- 'gray'
  }
  
  # Building output
  output <- fromJSON(paste0('https://pokeapi.co/api/v2/pokemon-color/',
                            color,collapse =","))
  
  output_df <- data.frame(output$pokemon_species$name)
  colnames(output_df) <- c(color)
  return(output_df)
}
```

## `egg_group`

The `egg_group` function returns a data frame of all Pokémon that fall
under a user-specified egg group. This function also checks the user’s
input against a list of acceptable groups, and returns an message if the
group the user has specified is not appropriate for querying the API,
along with a list of appropriate groups.

(e.g): egg_group(‘dragon’) returns a data frame of dragon:
charmander,charmeleon, (…), dragapult.

``` r
egg_group <- function(group){
  ###
  # Asks for egg group as an argument and returns all pokemon from that egg 
  # group
  #
  # NOTE: The egg groups are: monster, water1, bug, flying, ground, fairy, plant,
  #       humanshape, water3, mineral, indeterminate, water2, ditto, dragon, 
  #       and no-eggs
  ###
  
  # Establishing a group list for input checking
  egg_list <- c("monster", "water1", "bug", "flying", "ground", "fairy", "plant",
        "humanshape", "water3", "mineral", "indeterminate", "water2", "ditto",
        "dragon", "no-eggs")
  
  # Implementing a check and error message
  if(!(group %in% egg_list)){
    stop("Please pick one of: monster, water1, bug, flying, ground, fairy, plant,
         humanshape, water3, mineral, indeterminate, water2, ditto, dragon, 
         or no-eggs")
  }
  
  # Building output
  output <- fromJSON(paste0('https://pokeapi.co/api/v2/egg-group/',
                            group,collapse =","))
  
  # Correcting output
  output_df <- data.frame(output$pokemon_species$name)
  colnames(output_df) <- c(output$name)
  
  # Returning output_df
  return(output_df)
  
}
```

## `berry`

Our final function `berry` allows users to specify either a particular
berry (with the argument `berry =`) that they wish to learn about, or a
data frame of all berries and their information (with the argument
`all =`) by utilizing the `berry` endpoint. The output is then adjusted
to properly reflect the type of variables, including adjusting
`firmness` to be a factor with multiple levels.

Like with the previous functions, a user’s input is checked against a
list of berries that are appropriate options.

``` r
berry <- function(all = NULL, berry = NULL){
  ###
  # Asks for berry as an argument, returns growth_time, max_harvest, size,
  # smoothness, soil_dryness, firmness, and flavor
  #
  # NOTE: The berries are: cheri, chesto, pecha, rawst, aspear, leppa, oran, 
  # persim, lum, sitrus, figy, wiki, mago, aguav, iapapa, razz, bluk, nanab,
  # wepear, pinap
  ###
  
  # Building index df to convert user input into numeric 
  index <- rep(1:20)
  berry_list <- c("cheri", "chesto", "pecha", "rawst", "aspear", "leppa", "oran",
                  "persim", "lum", "sitrus", "figy", "wiki", "mago", "aguav",
                  "iapapa", "razz", "bluk", "nanab", "wepear", "pinap")
  index_df <- data.frame(berry_list, index)
  
  # Switching berry user input to a number if necessary
  if(is.numeric(berry) == FALSE){
    berry <- which(index_df$berry_list == berry)
  }
  
  # If user selects all berries
  if(all == TRUE){
  
    # Establishing empty df to merge things to
    output_df <- rep("a",12)
  
    for(n in 1:20){
      
      # Getting API output
      output <- fromJSON(paste0('https://pokeapi.co/api/v2/berry/',n,collapse =","))
    
      # Loading the data into a vector
      output_row <- c(output$name, output$growth_time, output$max_harvest, 
                      output$size,output$smoothness, output$soil_dryness, 
                      output$firmness$name,output$flavors$potency[1], 
                      output$flavors$potency[2],output$flavors$potency[3],
                      output$flavors$potency[4],output$flavors$potency[5])
  
      # Coercing into a data frame
      output_frame <- as.data.frame(t(output_row))
      colnames(output_frame) <- c("name", "growth_time", "max_harvest",
                                  "size", "smoothness","soil_dryness", 
                                  "firmness", "spicy","dry","sweet","bitter",
                                  "sour")
  
      # Adding the stats to the output df
      output_df <- rbind(output_df,output_frame)
      
    }
    
      # Fixing row names and removing extraneous 'starter' row
      output_df <- output_df[-1,]
      rownames(output_df) <- NULL
      
      
      # Fixing types of df
      output_df <- transform(output_df, growth_time = as.numeric(growth_time),
                             max_harvest = as.numeric(max_harvest), 
                             size = as.numeric(size),
                             smoothness = as.numeric(smoothness),
                             soil_dryness = as.numeric(soil_dryness),
                             spicy = as.numeric(spicy), dry = as.numeric(dry),
                             sweet = as.numeric(sweet), 
                             bitter = as.numeric(bitter),
                             sour = as.numeric(sour))
  
      # Setting firmness as a factor
      output_df$firmness <- as.factor(output_df$firmness)
      levels(output_df$firmness) <- c('very-soft','soft',
                                      'hard','very-hard','super-hard')
      
      
      # Returning our desired data frame
      return(output_df)
  }
  
  # Building single output
  output <- fromJSON(paste0('https://pokeapi.co/api/v2/berry/',berry,
                            collapse =","))
  
  output_row <- c(output$name, output$growth_time, output$max_harvest, 
                  output$size, output$smoothness, output$soil_dryness,
                  output$firmness$name, output$flavors$potency[1], 
                  output$flavors$potency[2],output$flavors$potency[3], 
                  output$flavors$potency[4],output$flavors$potency[5])
  
  output_names <- c("name", "growth_time", "max_harvest", "size", "smoothness", 
                    "soil_dryness", "firmness", "spicy","dry","sweet","bitter",
                    "sour")
  
  output_df <- as.data.frame(output_row, output_names)
  
  # Fixing types of df
  output_df <- transform(output_df, growth_time = as.numeric(growth_time),
                 max_harvest = as.numeric(max_harvest), size = as.numeric(size),
                 smoothness = as.numeric(smoothness), 
                 soil_dryness = as.numeric(soil_dryness),
                 spicy = as.numeric(spicy), dry = as.numeric(dry),
                 sweet = as.numeric(sweet), bitter = as.numeric(bitter), 
                 sour = as.numeric(sour))
  
  # Setting firmness as a factor
  output_df$firmness <- as.factor(output_df$firmness)
  levels(output_df$firmness) <- c('very-soft','soft','hard','very-hard',
                                  'super-hard')
  
  colnames(output_df) <- output$name
  
  return(output_df)

}
```

# Exploratory Data Analysis

## Pokémon Analysis

Where should would we start with our exploration of data from the
PokeApi? with Pokémon of course!

To begin, we can bring in all 251 Pokémon from the first and second
generation of Pokémon, using: `pokemon_df(all = TRUE)`

``` r
all_pokemon <- pokemon_df(all = TRUE)
```

Let’s examine the rough distribution of primary types across these
Pokémon.

``` r
# A one-way contigency table for primary type of Generation I and Generation II pokemon
all_pokemon %>% group_by(type_1) %>% 
  summarize(n())
```

    ## # A tibble: 17 x 2
    ##    type_1   `n()`
    ##    <chr>    <int>
    ##  1 bug         22
    ##  2 dark         5
    ##  3 dragon       3
    ##  4 electric    15
    ##  5 fairy        7
    ##  6 fighting     9
    ##  7 fire        20
    ##  8 ghost        4
    ##  9 grass       21
    ## 10 ground      11
    ## 11 ice          6
    ## 12 normal      37
    ## 13 poison      15
    ## 14 psychic     15
    ## 15 rock        13
    ## 16 steel        2
    ## 17 water       46

We can see that Water and Normal type Pokémon seem to be well
represented, at 46 and 37 respectively. On the other side of things,
there are only 2 pokemon whose primary type is Steel, and 3 who are
primarily Dragon type.

Continuing on, we can create a table of secondary types.

``` r
# A table for secondary types of pokemon
all_pokemon %>% group_by(type_2) %>% 
  summarize(n())
```

    ## # A tibble: 16 x 2
    ##    type_2   `n()`
    ##    <chr>    <int>
    ##  1 dark         1
    ##  2 dragon       1
    ##  3 electric     2
    ##  4 fairy        6
    ##  5 fighting     2
    ##  6 fire         2
    ##  7 flying      38
    ##  8 grass        3
    ##  9 ground      13
    ## 10 ice          4
    ## 11 poison      22
    ## 12 psychic      9
    ## 13 rock         5
    ## 14 steel        4
    ## 15 water        4
    ## 16 <NA>       135

From this table we are able to see that by far the largest category of
secondary type is None! Flying, Poison, and Ground are the standsouts
otherwise (38,22,13), with every other type being relatively negligibly
represented.

Creating a two-way table of primary vs. secondary types, we can
represent this information another way, and we are able to see that a
total of 116 pokemon have secondary types, while 135 (a majority) have
no secondary type.

``` r
types_table <- addmargins(table(all_pokemon$type_1, all_pokemon$type_2))
types_table
```

    ##           
    ##            dark dragon electric fairy fighting fire flying grass ground ice poison psychic rock steel water Sum
    ##   bug         0      0        0     0        1    0      5     2      0   0      7       0    1     2     0  18
    ##   dark        0      0        0     0        0    2      1     0      0   1      0       0    0     0     0   4
    ##   dragon      0      0        0     0        0    0      1     0      0   0      0       0    0     0     0   1
    ##   electric    0      0        0     0        0    0      1     0      0   0      0       0    0     2     0   3
    ##   fairy       0      0        0     0        0    0      1     0      0   0      0       0    0     0     0   1
    ##   fighting    0      0        0     0        0    0      0     0      0   0      0       0    0     0     0   0
    ##   fire        0      0        0     0        0    0      3     0      0   0      0       0    1     0     0   4
    ##   ghost       0      0        0     0        0    0      0     0      0   0      3       0    0     0     0   3
    ##   grass       0      0        0     0        0    0      3     0      0   0      9       2    0     0     0  14
    ##   ground      0      0        0     0        0    0      1     0      0   0      0       0    2     0     0   3
    ##   ice         0      0        0     0        0    0      2     0      2   0      0       2    0     0     0   6
    ##   normal      0      0        0     3        0    0     10     0      0   0      0       1    0     0     0  14
    ##   poison      0      0        0     0        0    0      3     0      2   0      0       0    0     0     0   5
    ##   psychic     0      0        0     1        0    0      3     1      0   0      0       0    0     0     0   5
    ##   rock        1      0        0     0        0    0      1     0      6   0      0       0    0     0     4  12
    ##   steel       0      0        0     0        0    0      1     0      1   0      0       0    0     0     0   2
    ##   water       0      1        2     2        1    0      2     0      2   3      3       4    1     0     0  21
    ##   Sum         1      1        2     6        2    2     38     3     13   4     22       9    5     4     4 116

Let’s examine this information in graph form.

``` r
type_bar <- ggplot(data = all_pokemon, aes( x = type_1))
type_bar + geom_bar(aes(fill = type_2), position = "dodge") + 
  xlab("1st Type") + ylab("Count") + 
  labs(title = "Barplot of a Pokemon's Primary Type by Secondary Type") +
  scale_fill_discrete(name = "2nd Type") + theme(axis.text.x = 
                                                   element_text(angle = 45))
```

![](README_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

From this bar plot we can see that Fighting type Pokémon have no
individuals with secondary types, while Water, Rock, and Bug types are
fairly diverse. We are also able to see that what we learned before -
that the majority of Pokémon have no secondary type seems to bear out
quiet clearly, with the highest bars in many cases representing the ‘NA’
secondary type.

If we are curious about other physical characteristics of Pokémon, we
can take a look at weight, or height - like in the table below:

``` r
# One way table of the frequency of pokemon's height
table(all_pokemon$height)
```

    ## 
    ##  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 25 35 38 40 52 65 88 92 
    ##  2 15 16 14 23 11 17 12 22 14 16 11 13 14  9  8  9  3  8  3  2  1  1  1  1  1  1  1  1  1

This table shows us that the heights of Pokémon appear to be strongly
skewed right, with quite a long tail.

Similarly, a histogram confirms that the case with weight is very
similar.

``` r
type_hist <- ggplot(data = all_pokemon, aes(x = weight))
type_hist + geom_histogram(fill = "red", bins = 100) + 
  labs(title = "Histogram of Pokemon Weight\n(Pokemon no.1-251)") + xlab("Weight") + 
  ylab("Count")
```

![](README_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

But what about how these two variables relate?

``` r
pokemon_scatter <- ggplot(all_pokemon, aes(x = weight, y = height))
pokemon_scatter + geom_point(aes(color = type_1)) + geom_smooth(method = lm) +
  labs(title = "Relationship between Weight and Height of Pokemon\n(Gen. 1 and 2)",
       x = 'Weight', y = 'Height', 
       color = 'Primary Type')
```

![](README_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

We can see that up until about the 2000 mark for Weight, the correlation
is very linear! After that, those pesky Rock and Steel type pokemon seem
to throw things off.

In order to relate height and weight in a quick way, we can create a
`BMI` variable (weight/height^2) using the `mutate()` function from the
`tidyverse`.

While we are at it, we can examine the mean, median, and standard
deviation of `weight`, `height`, and our new variable `BMI` across each
primary type of Pokémon.

``` r
# Numerical summary for average weight of pokemon by type, creating a
#new variable for 'BMI"
all_pokemon %>% group_by(type_1) %>% select(type_1, weight, height) %>% 
  mutate(bmi = weight/(height^2)) %>%
  summarise(AvgWeight = mean(weight), SdWeight = sd(weight), 
            MedianWeight = median(weight),AvgHeight = mean(height),
            SdHeight = sd(height), MedianHeight = median(height),
            AvgBMI = mean(bmi), SdBMI = sd(bmi), MedianBMI = median(bmi))
```

    ## # A tibble: 17 x 10
    ##    type_1   AvgWeight SdWeight MedianWeight AvgHeight SdHeight MedianHeight AvgBMI SdBMI MedianBMI
    ##    <chr>        <dbl>    <dbl>        <dbl>     <dbl>    <dbl>        <dbl>  <dbl> <dbl>     <dbl>
    ##  1 bug           331.     333.        295        9.86     4.46         10    3.13  1.75      2.77 
    ##  2 dark          206.     136.        270        8.8      3.56          9    2.36  1.05      2.7  
    ##  3 dragon        766     1157.        165       26.7     11.7          22    1.51  2.45      0.103
    ##  4 electric      381.     446.        245        8.87     4.79          8    3.95  1.61      3.83 
    ##  5 fairy         160.     197.         75        7.29     4.46          6    2.14  0.751     2.17 
    ##  6 fighting      499.     343.        480       11.6      4.10         14    4.13  2.80      3.13 
    ##  7 fire          596.     602.        325       12.9      7.83         10.5  3.37  1.81      3.04 
    ##  8 ghost         104.     201.          5.5     12.8      4.03         14    0.503 0.869     0.105
    ##  9 grass         228.     362.         69        9.62     5.23          8    1.57  1.04      1.33 
    ## 10 ground        528.     457.        335        8.64     4.61         10    6.10  3.83      4.5  
    ## 11 ice           300.     234.        283        9.83     5.27         10    3.06  1.21      2.91 
    ## 12 normal        456.     761.        325       10.5      5.45         11    3.24  2.15      2.62 
    ## 13 poison        305.     257.        200       12.6      7.76         12    2.22  1.38      2.31 
    ## 14 psychic       474.     569.        285       13.5     11.8          13    2.51  1.12      2.5  
    ## 15 rock          963.     922.        590       16.6     22.0          12    7.22  6.00      4.69 
    ## 16 steel        2252.    2471.       2252.      54.5     53.0          54.5  1.11  0.901     1.11 
    ## 17 water         570.     614.        314.      12.3      9.67         11    3.49  1.53      3.39

There are a ton of great tidbits here!

Dragon types appear quite heavy on average, but the median dragon is
actually quite light.

Ghost types similarly appear to have a single ghost who is quite heavy,
as the median weight is only 5.5, while the mean is a still very light
104.25.

If you were scouting for a basketball team comprised of Pokémon, the
median height column might be of use - and Dragon, Fighting, and Ghost
types would be at the top of your list.

Similarly, Ground, Dragon, and Fighting type Pokémon appear to be very
dense according to our `BMI` variable. While Ghost types are clock in at
a negligible median BMI of 0.105, no surprise there I suppose!

Were Pokémon denser in the first or second generation of Pokémon? What
type of Pokémon are the outliers for `BMI`?

``` r
# Creating plotting item and generating BMI
bmi_df <- all_pokemon %>% select(name, type_1, weight, height) %>% 
  mutate(bmi = weight/(height^2)) %>% mutate(gen = NA)

# Establishing gen values
bmi_df$gen[1:151] <- "1"
bmi_df$gen[152:251] <- "2"

bmi_box <- ggplot(bmi_df, aes(x = gen, y = bmi))
bmi_box + geom_boxplot() + geom_jitter(aes(color = type_1), width = 0.3) + 
  labs(title = "Boxplot of Pokemon BMI by Generation\n(Gen. 1 and 2)",
       x = "Generation", y = "BMI", color = "Primary Type")
```

![](README_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

Our box plot shows that the BMI’s for Generation 1 and Generation 2 seem
to be similarly distributed. We can also see that a good number of our
outliers appear to be of the Rock or Steel type.

Moving on to Pokémon battle statistics, we can group by type again and
check out the averages of the big five - Attack, Defense, Special
Attack, Special Defense, and Speed.

``` r
# Numerical summary for average weight of pokemon by type_1 and type_2
all_pokemon %>% group_by(type_1) %>% 
  summarise(AvgAttack = mean(attack), AvgDefense = mean(defense),
            AvgSpecAttack = mean(spec_attack), 
            AvgSpecDefense = mean(spec_defense), 
            AvgSpeed = mean(speed))
```

    ## # A tibble: 17 x 6
    ##    type_1   AvgAttack AvgDefense AvgSpecAttack AvgSpecDefense AvgSpeed
    ##    <chr>        <dbl>      <dbl>         <dbl>          <dbl>    <dbl>
    ##  1 bug           66.1       70.7          47.3           68.6     54.5
    ##  2 dark          79         57.4          74             75.4     86.2
    ##  3 dragon        94         68.3          73.3           73.3     66.7
    ##  4 electric      61.1       59.3          86.3           69.7     87  
    ##  5 fairy         57.1       60.6          60             68.6     35  
    ##  6 fighting      94.4       61.9          42.8           73.3     63.1
    ##  7 fire          80.8       65.1          84.7           76.2     78.8
    ##  8 ghost         52.5       48.8         108.            62.5     92.5
    ##  9 grass         64.8       67.6          76.7           69.3     55.2
    ## 10 ground        84.5       88.6          40.9           51.8     58.2
    ## 11 ice           61.7       52.5          75             70       70  
    ## 12 normal        66.5       53.7          54.5           62.4     66.6
    ## 13 poison        76.1       67.9          58             62.9     63.5
    ## 14 psychic       64.4       64.7          94.7           87       87.3
    ## 15 rock          86.3      103.           60             60.4     54.5
    ## 16 steel         82.5      170            47.5           67.5     50  
    ## 17 water         69.4       76.0          66.2           70.1     63.5

Dragons and Fighting types seem to dominate with Attack, while Rock and
Steel types on average have very high Defense.

For our Special statistics, Ghost and Psychic types are strong in
Special Attack, while Pyschics are the only standouts for Special
Defense.

Meanwhile Electric, Ghost and Psychic types are the fastest Pokémon on
average.

## Berry Analysis

The PokeAPI has it all, from Pokémon to berries, so let’s analyze some
berries!

We can start by pulling in a data frame of all berries using
`berry(all = TRUE)`.

``` r
all_berry <- berry(all = TRUE)
```

Let’s start by examining the firmness of the berries.

``` r
berry_hist <- ggplot(data = all_berry, aes(x = firmness))
berry_hist + geom_histogram(fill = "purple", stat = "count") + labs(title = "Firmness of All Berries by Category") +
  xlab("Firmness") + ylab("Count")
```

![](README_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

Of the five firmness levels, hard is the most common with six berries
falling into that category, while super-hard is by far the least common
with only one.

Perhaps there is a connection between how long it takes a berry to grow,
and the size that it grows to, let’s find out with a scatter plot.

``` r
berry_scatter <- ggplot(all_berry, aes(x = size, y = growth_time))
berry_scatter + geom_point(aes(color = firmness)) + geom_smooth(method = lm) +
  labs(title = "Association Between Size and Growth Time of Berries", 
       x = 'Size', y = 'Growth Time', color = 'Firmness')
```


![](README_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

``` r
levels(all_berry$firmness)
```

    ## [1] "very-soft"  "soft"       "hard"       "very-hard"  "super-hard"

Surprisingly nearly every berry appears to take between two to five days
to grow, regardless of size. Though it is worth noting that one of the
smallest berries actually appears to have by far the longest growth
time.

Finally, let’s examine the average flavor profiles of the berries.

``` r
# Berry taste profile summary data
all_berry %>% select(spicy,dry,sweet,bitter,sour) %>% 
  summarise(AvgSpice = mean(spicy), AvgDryness = mean(dry), 
            AvgSweet = mean(sweet), AvgBitter = mean(bitter), 
            AvgSour = mean(sour))
```

    ##   AvgSpice AvgDryness AvgSweet AvgBitter AvgSour
    ## 1     4.25       4.25     4.25      4.25    4.25

Another surprise - the twenty berries must have been designed to average
out to the same value (4.25) across the five flavors!

## Conclusion

And that completes our quick tour of the `PokeAPI`. Throughout this
vignette I covered a few endpoints that stood out to me -
`pokemon`,`habitat`,`color`,`egg-group`, and `berry`, while making
functions to assist users in bringing in data and data frames.

There are numerous other endpoints from the API, it is quite large and
the [documentation](https://pokeapi.co/docs/v2) is very thorough if you
are interested in learning more or examining other endpoints.
