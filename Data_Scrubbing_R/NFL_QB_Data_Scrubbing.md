---
title: "NFL QB Performance - Summary"
author: "Oren Shapira"
date: "July 7, 2019"
always_allow_html: yes
output: 
  html_document:
    keep_md: yes
    toc: true
    theme: united
  github_document: default
---




# **Summary of Project**

## Overview
With the increasing ubiquity of sports gambling and Fantasy leagues of various formats, NFL Sundays can become a stessful time for high-stakes betting. There are many publicly available tools that can guide people's betting decisions (e.g. Vegas odds, Fantasy League Point Projections), but these are often black-box algorithms that may provide contradictory information. For example, why might Yahoo Fantasy project more points for a given player than in an ESPN Fantasy league with the same scoring settings? The first step to answering this is to identify all elements that may impact a player's performance, and determine which ones have the most significant impact. My goal is to take a bite-size approach to this task. 

For this analysis, I used various machine learning algorithms to determine how much weather and and opposing team defense impact a Quarterback's performance for a given game. I defined "performance" as how many more/less passing yards a QB threw in a game compared to their season average. The specific attributes I focused on for weather were temperature, humidity, and wind (mph). For opponent defense, I focused on the relative strength of a team's passing defense.


## The Data:

This analysis required scrubbing and merging various datasets.

1. Weather_2013_1231.csv: (Weather dataset)
  Summary: This dataset contains weather information for every NFL game between 1960-2013. It has 11 attributes, and 11,192 tuples.
  Source: http://www.nflsavant.com/about.php
  Data snapshot:


id             home_team            home_score  away_team              away_score   temperature   wind_chill  humidity    wind_mph  weather                                          date      
-------------  ------------------  -----------  --------------------  -----------  ------------  -----------  ---------  ---------  -----------------------------------------------  ----------
196009230ram   Los Angeles Rams             21  St. Louis Cardinals            43            66           NA  78%                8  66 degrees- relative humidity 78%- wind 8 mph    9/23/1960 
196009240dal   Dallas Cowboys               28  Pittsburgh Steelers            35            72           NA  80%               16  72 degrees- relative humidity 80%- wind 16 mph   9/24/1960 
196009250gnb   Green Bay Packers            14  Chicago Bears                  17            60           NA  76%               13  60 degrees- relative humidity 76%- wind 13 mph   9/25/1960 


2. Game_Logs_Quarterback.csv (QB Stats for every game)
  Summary: This dataset contains passing statistics for every Quarterback, for NFL game between 2000 - 2016. This includes total passing yards, pass attempts, Touchdowns, Interceptions, etc. This will be joined with the weather dataset above on overlapping years. This dataset has 29 attributes and 40,247 rows. 
  Source: https://www.kaggle.com/kendallgillies/nflstatistics
  Data snapshot:
  

Player.Id                Name               Position    Year  Season       Week  Game.Date   Home.or.Away   Opponent   Outcome   Score       Games.Played  Games.Started   Passes.Completed   Passes.Attempted   Completion.Percentage   Passing.Yards   Passing.Yards.Per.Attempt   TD.Passes   Ints   Sacks   Sacked.Yards.Lost    Passer.Rating  Rushing.Attempts   Rushing.Yards   Yards.Per.Carry   Rushing.TDs   Fumbles   Fumbles.Lost 
-----------------------  -----------------  ---------  -----  ----------  -----  ----------  -------------  ---------  --------  ---------  -------------  --------------  -----------------  -----------------  ----------------------  --------------  --------------------------  ----------  -----  ------  ------------------  --------------  -----------------  --------------  ----------------  ------------  --------  -------------
jaredzabransky/2495791   Zabransky, Jared               2007  Preseason       1  11-Aug      Home           CHI        L         19 to 20               0  0               --                 --                 --                      --              --                          --          --     --      --                             0.0  --                 --              --                --            --        --           
jaredzabransky/2495791   Zabransky, Jared               2007  Preseason       2  18-Aug      Away           ARI        W         33 to 20               1  0               1                  4                  25                      19              4.8                         0           0      0       0                             46.9  --                 --              --                --            --        --           
jaredzabransky/2495791   Zabransky, Jared               2007  Preseason       3  25-Aug      Home           DAL        W         28 to 16               0  0               --                 --                 --                      --              --                          --          --     --      --                             0.0  --                 --              --                --            --        --           


3. Career_Stats_Passing.csv (QB stats for season averages)
  Summary: This dataset contains every NFL player's overall passing stats for the entire season, and spans between 1924-2016. It will be used primarily to normalize across Quarterbacks that have different averages of passing yards per game. There are 23 attributes and 8,525 rows.
  Source: https://www.kaggle.com/kendallgillies/nflstatistics
  Data snapshot:
  

Player.Id           Name          Position    Year  Team               Games.Played  Passes.Attempted   Passes.Completed   Completion.Percentage    Pass.Attempts.Per.Game  Passing.Yards   Passing.Yards.Per.Attempt   Passing.Yards.Per.Game   TD.Passes   Percentage.of.TDs.per.Attempts   Ints   Int.Rate   Longest.Pass   Passes.Longer.than.20.Yards   Passes.Longer.than.40.Yards   Sacks   Sacked.Yards.Lost    Passer.Rating
------------------  ------------  ---------  -----  ----------------  -------------  -----------------  -----------------  ----------------------  -----------------------  --------------  --------------------------  -----------------------  ----------  -------------------------------  -----  ---------  -------------  ----------------------------  ----------------------------  ------  ------------------  --------------
tomfarris/2513861   Farris, Tom               1948  Chicago Rockets               0  --                 --                 --                                          0.0  --              --                          --                       --          --                               --     --         --             --                            --                            --      --                             0.0
tomfarris/2513861   Farris, Tom               1947  Chicago Bears                 9  2                  0                  0                                           0.2  0               0                           0                        0           0                                0      0          --             0                             0                             0       0                             39.6
tomfarris/2513861   Farris, Tom               1946  Chicago Bears                11  21                 8                  38.1                                        1.9  108             5.1                         9.8                      1           4.8                              2      9.5        --             0                             0                             0       0                             31.5
  

4. Sportsref_download_ [2000-2013].xls (Team Defense Stats)
  Summary: This is a series of 13 datasets each NFL season between 2000 - 2013. It contains defensive stats and rankings for each team. To merge these datasets, I looped through all files within the defensive datasets subfolder, and then added a column to identify which season the data corresponded too. Subsequent calculations were done to determine the relative pass defense rating of a team on a scale of 1-5. After merging, this dataset has 23 attributes and 480 tuples.
b.	Source: https://www.pro-football-reference.com/years/2013/opp.htm



```
##  [1] "defense_2000.xlsx" "defense_2001.xlsx" "defense_2002.xlsx"
##  [4] "defense_2003.xlsx" "defense_2004.xlsx" "defense_2005.xlsx"
##  [7] "defense_2006.xlsx" "defense_2007.xlsx" "defense_2008.xlsx"
## [10] "defense_2009.xlsx" "defense_2010.xlsx" "defense_2011.xlsx"
## [13] "defense_2012.xlsx" "defense_2013.xlsx"
```



 Rk  Tm                      G   Cmp   Att   Cmp%    Yds   TD   TD%   Int   Int%   Y/A   AY/A    Y/C     Y/G   Rate   Sk   Yds__1   NY/A   ANY/A    Sk%      EXP  Year 
---  --------------------  ---  ----  ----  -----  -----  ---  ----  ----  -----  ----  -----  -----  ------  -----  ---  -------  -----  ------  -----  -------  -----
  1  Tennessee Titans       16   242   466   51.9   2423   10   2.1    17    3.6   5.9    4.7   11.4   151.4   62.0   55      338    4.7     3.6   10.6   117.40  2000 
  2  Washington Redskins    16   254   462   55.0   2621   12   2.6    17    3.7   6.3    5.1   11.4   163.8   67.4   45      283    5.2     4.1    8.9    62.41  2000 
  3  Dallas Cowboys         16   277   458   60.5   2693   20   4.4    16    3.5   6.3    5.6   10.4   168.3   78.7   25      189    5.6     4.9    5.2    -4.35  2000 


# **Data Scrubbing**

There was a significant amount of preprocessing that had to be done to prepare these datasets for data mining analysis. At a high level, this included filtering the datasets to rows/columns of interest, joining the datasets together, and discretizing numeric variables into categorical values. Preprocessing details for each dataset are described below:





## Weather dataset:
1.	Convert humidity variable from string to numeric
2.	Convert game date from string into date format
3.	Impute a value of 1 mph for all rows in the dataset that had a Wind mph value of 0. I felt there was a high likelihood the source data was inaccurate for these rows
4.	Remove teams from dataset that play in a dome. These teams/game locations are not subject to weather conditions, which was the primary data mining focus
5.	Convert team names to their respective acronyms (i.e. Buffalo Bills -> "BUF") so that this dataset could be joined to others that used team acronyms











## Game Logs dataset:
1.	Filter on rows where QBs started. I wanted to exclude game log entries for QBs that likely didn't play many minutes and had very few pass attempts
2.	Convert game date from to proper date format
3.	Convert game week from integer to character
4.	Convert game completion percentage to a numeric variable
5.	Drop rows with NA values for game completion percentage 
6.	Filtered only on regular season games. Preseason games were excluded because QBs do not typically play their hardest in these games. Post-season games were excluded beacuse I thought data would be skewed to high-performing quarterbacks that may be less susceptible to weather. Post-season games also had game dates that spilled over into subsequent years (such as early January), which would have complicated my ability to join datasets based on season- year.
7.	Drop rows with NA values for passing yards



```
## [1] "factor"
```

```
## [1] "Date"
```

```
## [1] "integer"
```


## Career Stats dataset:

1. Flag players that normally play in domes for home games
2. Convert team names to their respective acronyms (i.e. Buffalo Bills -> "BUF") so that this dataset could be joined to others that used team acronyms
3. Drop rows with NA values for season completion percentage













Defense dataset:
1.	Calculate mean, standard deviation, and z-score of each team's defensive strength (based on total passing yards allowed in the season). Metrics were calculated relative to all other teams' passing yards allowed within the same season. Since the NFL has gradually become more pass-heavy over time, I felt it was important to calculate z-scores of a team's defensive strength (passing yards allowed) as compared to other teams within the same season rather than across all seasons.
2. Use min-max normalization to convert a team's z-score value to a rating score between 1-5. A higher score indicates a stronger pass defense. Converting from z-score values that span between -3 to 3
3.	Drop columns that were irrelevant to my analysis (e.g. rushing yards allowed, pass completion %, etc.)
3.	Convert team names to their respective acronyms for purposes of joining with other datasets





## Joining datasets:
1.	Inner join Career Stats dataset with the Game Logs dataset based on matching season Year and QB name. Rename this "CareerGameLogs"
a.	Create additional column for game location using an if/else statement. If a game log was marked as "away" for a QB, then game location = Opponent. If it was a "home" game, then game location = the QB's team. 
i.	This added field was used to join with the weather dataset. Without this field, the weather dataset would have only joined with the merged GameLogs/CareerStats dataset on games where the QB was playing a home game. 	
b.	Create a "Yards deviation" column that showed how a QB's passing yards in a game deviate from their season average (game log passing yards - season average passing yards). This was the primary "performance" metric which was later discretized to be the class variable we tried to predict.
2.	Join the merged "CareerGameLogs" with the weather dataset, based on matching game date and game location. Rename this "weather_gamelogs_inner".
3.	Join the merged "weather_gamelogs_inner" with the Defense dataset by joining on matching season year and team acronym.








